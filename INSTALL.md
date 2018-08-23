# CUDA Batch Solver

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

## Using CUDA Batch Solver as third party

When you install CudaBatchedSolver (say in `<prefix>`) a file `CudaBatchedSolverConfig.cmake` is installed in `<prefix>/lib/cmake/CudaBatchedSolver/` that allows you to import the library in your CMake project.
In your `CMakeLists.txt` file you can add the dependency in this way:

```cmake
# Find the package from the CudaBatchedSolverConfig.cmake 
# in <prefix>/lib/cmake/CudaBatchedSolver/. Under the namespace CudaBatchedSolver::
# it exposes the target CudaBatchedSolver that allows you to compile
# and link with the library
find_package(CudaBatchedSolver CONFIG REQUIRED)
...
# suppose you want to try it out in a executable
add_executable(test yourfile.cpp)
# add link to the library
target_link_libraries(test PUBLIC CudaBatchedSolver::CudaBatchedSolver)
```

Then, in order to build just pass the location of `CudaBatchedSolverConfig.cmake` from the cmake command line:

```bash
cmake .. -DCudaBatchedSolver_DIR=<prefix>/lib/cmake/CudaBatchedSolver/
```

Note that `target_include_directories()` is not necessary.