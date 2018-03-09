# Vc: portable, zero-overhead C++ types for explicitly data-parallel programming

*NOTE*: This is the development version, implementing https://wg21.link/p0214.
For production use consider the latest release or the 1.3 branch.
This implementation requires GCC 7.1 or newer.
Support for Clang, ICC, and MSVC is available with the 1.3 branch.

## Introduction

Recent generations of CPUs, and GPUs in particular, require data-parallel codes
for full efficiency. Data parallelism requires that the same sequence of
operations is applied to different input data. CPUs and GPUs can thus reduce
the necessary hardware for instruction decoding and scheduling in favor of more
arithmetic and logic units, which execute the same instructions synchronously.
On CPU architectures this is implemented via SIMD registers and instructions.
A single SIMD register can store N values and a single SIMD instruction can
execute N operations on those values. On GPU architectures N threads run in
perfect sync, fed by a single instruction decoder/scheduler. Each thread has
local memory and a given index to calculate the offsets in memory for loads and
stores.

Current C++ compilers can do automatic transformation of scalar codes to SIMD
instructions (auto-vectorization). However, the compiler must reconstruct an
intrinsic property of the algorithm that was lost when the developer wrote a
purely scalar implementation in C++. Consequently, C++ compilers cannot
vectorize any given code to its most efficient data-parallel variant.
Especially larger data-parallel loops, spanning over multiple functions or even
translation units, will often not be transformed into efficient SIMD code.

The Vc library provides the missing link. Its types enable explicitly stating
data-parallel operations on multiple values. The parallelism is therefore added
via the type system. Competing approaches state the parallelism via new control
structures and consequently new semantics inside the body of these control
structures.

Vc is a free software library to ease explicit vectorization of C++ code. It
has an intuitive API and provides portability between different compilers and
compiler versions as well as portability between different vector instruction
sets. Thus an application written with Vc can be compiled for:

* AVX and AVX2
* SSE2 up to SSE4.2 or SSE4a
* Scalar
* MIC (only before Vc 2.0)
* AVX-512 (since Vc 2.0)
* NEON (in development)
* NVIDIA GPUs / CUDA (research)

## Examples

### Scalar Product

Let's start from the code for calculating a 3D scalar product using builtin floats:
```cpp
using Vec3D = std::array<float, 3>;
float scalar_product(Vec3D a, Vec3D b) {
  return a[0] * b[0] + a[1] * b[1] + a[2] * b[2];
}
```
Using Vc, we can easily vectorize the code using the `native_simd<float>` type:
```cpp
using Vc::native_simd;
using Vec3D = std::array<native_simd<float>, 3>;
float_v scalar_product(Vec3D a, Vec3D b) {
  return a[0] * b[0] + a[1] * b[1] + a[2] * b[2];
}
```
The above will scale to 1, 4, 8, 16, etc. scalar products calculated in parallel, depending
on the target hardware's capabilities.

For comparison, the same vectorization using Intel SSE intrinsics is more verbose and uses
prefix notation (i.e. function calls):
```cpp
using Vec3D = std::array<__m128, 3>;
__m128 scalar_product(Vec3D a, Vec3D b) {
  return _mm_add_ps(_mm_add_ps(_mm_mul_ps(a[0], b[0]), _mm_mul_ps(a[1], b[1])),
                    _mm_mul_ps(a[2], b[2]));
}
```
The above will neither scale to AVX, MIC, etc. nor is it portable to other SIMD ISAs.

## Build Requirements

cmake >= 3.0

C++17 Compiler:

* GCC >= 7.1


## Building and Installing Vc

* Create a build directory:

```sh
$ mkdir build
$ cd build
```

* Call `cmake`; the following options may be interesting:
  - `-DCMAKE_INSTALL_PREFIX=<path>`:
    Select a different install prefix. Note that installing is not required (use
     `-I<path to Vc src>`) and currently not supported.
  - `-DENABLE_UBSAN=ON`:
    Build tests with the “undefined behavior sanitizer” enabled.
  - `-DTARGET_ARCHITECTURE=<target>`:
    Select a target architecture, different from the one you are building on.
  - `-DUSE_CCACHE=ON`:
    Use `ccache` (when found) to speed up recurring builds.

```sh
$ cmake <srcdir>
```

* Build and run tests:

```sh
$ make -j8
$ ctest -j8
```

## Documentation

The documentation of the master branch is currently out of date. Please refer to
https://wg21.link/p0214 for the specification.

Documentation for older releases is available at:

* [1.3.0 release](https://web-docs.gsi.de/~mkretz/Vc-1.3.0/)
* [1.2.0 release](https://web-docs.gsi.de/~mkretz/Vc-1.2.0/)
* [1.1.0 release](https://web-docs.gsi.de/~mkretz/Vc-1.1.0/)
* [0.7 branch](https://web-docs.gsi.de/~mkretz/Vc-0.7/)

## Publications

* [M. Kretz, "Extending C++ for Explicit Data-Parallel Programming via SIMD
  Vector Types", Goethe University Frankfurt, Dissertation,
  2015.](http://publikationen.ub.uni-frankfurt.de/frontdoor/index/index/docId/38415)
* [M. Kretz and V. Lindenstruth, "Vc: A C++ library for explicit
  vectorization", Software: Practice and Experience,
  2011.](http://dx.doi.org/10.1002/spe.1149)
* [M. Kretz, "Efficient Use of Multi- and Many-Core Systems with Vectorization
  and Multithreading", University of Heidelberg,
  2009.](http://code.compeng.uni-frankfurt.de/attachments/13/Diplomarbeit.pdf)

[Work on integrating the functionality of Vc in the C++ standard library.](
https://github.com/VcDevel/Vc/wiki/ISO-Standardization-of-the-Vector-classes)

## Communication

A channel on the freenode IRC network is reserved for discussions on Vc:
[##vc on freenode](irc://chat.freenode.net:6667/##vc)
([via SSL](ircs://chat.freenode.net:6697/##vc))

Feel free to use the GitHub issue tracker for questions.
Alternatively, there's a [mailinglist for users of
Vc](https://compeng.uni-frankfurt.de/mailman/listinfo/vc)

## License

Vc is released under the terms of the [3-clause BSD license](http://opensource.org/licenses/BSD-3-Clause).
