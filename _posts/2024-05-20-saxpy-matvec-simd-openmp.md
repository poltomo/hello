---
layout: post
title:  "saxpy and matvec, SIMD Intrinsics, and OpenMP multithreading"
date:   2024-05-20 03:13:00 -0700
---

Extending python with C libraries is very useful. Say there's something I want to do with torch tensors that Pytorch does not support; it can be much easier and faster to do the job on the raw float arrays. I can also hand optimize operations this way. This is especially useful for writing your own CUDA kernels. Here, I am using ARM NEON intrinsics, but you can use whatever you like.

## saxpy


```python
import torch

n = 2 * 4 * 100000
s = torch.empty(n)
a = torch.rand(1)
x = torch.rand(n)
y = torch.rand(n)

s = a * x + y
%timeit a * x + y
s
```

    392 µs ± 920 ns per loop (mean ± std. dev. of 7 runs, 1,000 loops each)





    tensor([0.9778, 0.2121, 0.6137,  ..., 1.3553, 1.3155, 1.4675])



## saxpy made 4x faster with SIMD intrinsics  
You can get addresses the first element of torch tensors with the data_ptr() member. These are 64 bit unsigned integers on 64 bit systems. With these addresses and the tensors' lengths, you can do whatever you want with the tensor. Be careful not to leak memory here. Do things in place or place the results of your computation into a passed tensor. Below, I place the results of a * x + y into the passed s tensor. Also, make sure not to get the data type wrong. In this case, I know that I am working with the FP32 type.


```python
%%writefile ./my_lib1.c
#include<arm_neon.h>
// assumes n is a multiple of 8
void saxpy4(float* s, float a, const float* x, const float* y, int n) {
    float32x4_t vs, va, vx, vy;
    va = vdupq_n_f32(a);
    for (int i = 0;i < n; i += 4) {
        vx = vld1q_f32(x + i);
        vy = vld1q_f32(y + i);
        vs = vfmaq_f32(vy, va, vx);
        vst1q_f32(s + i, vs);
    }
}
```

    Writing ./my_lib1.c



```python
!gcc-13 -O3 -shared -o my_lib1.so -fPIC my_lib1.c
```


```python
import ctypes
lib = ctypes.CDLL('./my_lib1.so')

lib.saxpy4.argtypes = (ctypes.c_uint64, ctypes.c_float, ctypes.c_uint64, ctypes.c_uint64, ctypes.c_int)
lib.saxpy4.restype = None

%timeit lib.saxpy4(s.data_ptr(), a.item(), x.data_ptr(), y.data_ptr(), n)
s
```

    106 µs ± 129 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)





    tensor([0.9778, 0.2121, 0.6137,  ..., 1.3553, 1.3155, 1.4675])



Its roughly 4 times as fast because the SIMD vectors are 128 bits wide which is 4 fp32s.

## SIMD intrinsics and OpenMP multithreading

multithreading is only worth it for large n. Here, I'm using just two threads. Be careful about bad memory access patterns and false sharing. That's not a concern in this case.


```python
%%writefile ./my_lib2.c
#include <omp.h>
#include<arm_neon.h>
// assumes n is a multiple of 8
void saxpy2x4(float* s, float a, const float* x, const float* y, int n) {
    #pragma omp parallel num_threads(2)
    {
        n /= 2;
        int tid = omp_get_thread_num();
        s += tid * n;
        x += tid * n;
        y += tid * n;

        float32x4_t vs, va, vx, vy;
        va = vdupq_n_f32(a);
        for (int i = 0;i < n; i += 4) {
            vx = vld1q_f32(x + i);
            vy = vld1q_f32(y + i);
            vs = vfmaq_f32(vy, va, vx);
            vst1q_f32(s + i, vs);
        }
    }
}
```

    Writing ./my_lib2.c



```python
!gcc-13 -O3 -shared -o my_lib2.so -fPIC -fopenmp my_lib2.c
```


```python
lib2 = ctypes.CDLL('./my_lib2.so')

lib2.saxpy2x4.argtypes = (ctypes.c_uint64, ctypes.c_float, ctypes.c_uint64, ctypes.c_uint64, ctypes.c_int)
lib2.saxpy2x4.restype = None

%timeit lib2.saxpy2x4(s.data_ptr(), a.item(), x.data_ptr(), y.data_ptr(), n)
s
```

    67.2 µs ± 267 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)





    tensor([0.9778, 0.2121, 0.6137,  ..., 1.3553, 1.3155, 1.4675])



Its even faster!

## matrix vector product



```python
W = torch.rand(4,4)
h = torch.empty(4)
x = torch.rand(4)

h = torch.mv(W, x) # same as W @ x
%timeit torch.mv(W, x)
h
```

    980 ns ± 14.4 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)





    tensor([0.9658, 0.7360, 1.3095, 1.7704])



## matvec made faster with SIMD intrinsics

if h = Wx, the ith entry of h is the dot product of the ith row of W and x. This can be extended to matrix multiplications too: C = AB, where the ith column of C is the matrix vector product of A and the ith column of B. There's more optimizations that can be done for matmuls.


```python
%%writefile ./my_lib3.c
#include<arm_neon.h>
// assumes W is 4x4 and x is 4x1
void matvec4x4(float* h, const float* W, const float* x) {
    float32x4_t vw, vx;
    vx = vld1q_f32(x);
    vw = vld1q_f32(W);
    h[0] = vaddvq_f32(vmulq_f32(vx, vw));
    vw = vld1q_f32(W + 4);
    h[1] = vaddvq_f32(vmulq_f32(vx, vw));
    vw = vld1q_f32(W + 8);
    h[2] = vaddvq_f32(vmulq_f32(vx, vw));
    vw = vld1q_f32(W + 12);
    h[3] = vaddvq_f32(vmulq_f32(vx, vw));
}
```

    Writing ./my_lib3.c



```python
!gcc-13 -O3 -shared -o my_lib3.so -fPIC -fopenmp my_lib3.c
```


```python
lib3 = ctypes.CDLL('./my_lib3.so')

lib3.matvec4x4.argtypes = (ctypes.c_uint64, ctypes.c_uint64, ctypes.c_uint64)
lib3.matvec4x4.restype = None

%timeit lib3.matvec4x4(h.data_ptr(), W.data_ptr(), x.data_ptr())
h
```

    675 ns ± 10.7 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)





    tensor([0.9658, 0.7360, 1.3095, 1.7704])



SIMD extensions are great if your linear operators are very small
