# CUDA Batch Solver

Continuous integration: 
- [![Build Status](https://travis-ci.org/EncovGroup/CudaBatchedSolver.svg?branch=develop)](https://travis-ci.org/EncovGroup/CudaBatchedSolver) develop branch

## Building instructions

Required tools:
* CMake >= 3.7 
* Git
* C/C++ compiler (gcc or visual studio or clang) with C++11 support.

Dependencies

* CUDA >= 7.0

### Compile the project

```bash
git clone https://github.com/EncovGroup/CudaBatchedSolver
mkdir build && cd build
cmake  -DCMAKE_INSTALL_PREFIX=$PWD/install ..
make -j 8
```

### CMake options

* `CudaBatchedSolver_BUILD_SAMPLES` (default `ON`) Build the samples 
* `CudaBatchedSolver_EXPORT_PACKAGE` (default `OFF`) Export the library as cmake package

