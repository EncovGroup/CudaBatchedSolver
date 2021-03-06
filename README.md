# CUDA Batch Solver

This code provides an efficient solver and matrix inversion for small matrices,
using partial pivoting. 
This repository is a mirror of the code that can be found at https://developer.nvidia.com/computeworks-developer-exclusive-downloads 

On sm_13 GPUs (with 16 KB of shared memory), the maximum dimensions are:

```
DP           batched solver, non-batched matrix inverse:  2... 44
DP complex   batched solver, non-batched matrix inverse:  2... 31

SP           batched matrix inverse:                      2... 62
SP complex   batched matrix inverse:                      2... 44
DP           batched matrix inverse:                      2... 44
DP complex   batched matrix inverse:                      2... 31
```

On sm_2x and sm_3x GPUs (with 48 KB shared memory), the maximum dimensions are:
```
DP           batched solver, non-batched matrix inverse:  2... 76
DP complex   batched solver, non-batched matrix inverse:  2... 53

SP           batched matrix inverse:                      2...109
SP complex   batched matrix inverse:                      2... 77
DP           batched matrix inverse:                      2... 77
DP complex   batched matrix inverse:                      2... 55
```

The main CUDA code is in two files, `inverse.cu` (which implements the batched
matrix inversion) and `solve.cu` (which implements the solver code also used by 
the non-batched matrix inversion). These modules export relevant functions via 
a C-style interface that can easily be bound to Fortran (see files `inverse.h`
and solve.h).

The example code demonstrates how to call the batched and non-batched matrix 
inversion and the batched solver. The output should be similar to the below:.
```bash
$ ./example_batch_solver 
```

Non-batched matrix inversion
```
        3.000000   1.000000   1.000000              0.400000  -0.100000  -0.100000
A =     1.000000   3.000000   1.000000   Ainv =    -0.100000   0.400000  -0.100000
        1.000000   1.000000   3.000000             -0.100000  -0.100000   0.400000

          1  +0i     1  +2i     2 +10i               10  +1i    -2  +6i    -3  -2i
A =       1  +1i     0  +3i    -5 +14i   Ainv =       9  -3i     0  +8i    -3  -2i
          1  +1i     0  +5i    -8 +20i               -2  +2i    -1  -2i     1  +0i

```

Batched matrix inversion
```
        1.000000   0.500000   0.333333              1.333333  -0.666667  -0.000000
A(0) =  0.500000   1.000000   0.666667   Ainv(0) = -0.666667   2.133333  -1.200000
        0.333333   0.666667   1.000000              0.000000  -1.200000   1.800000

        3.000000   1.000000   1.000000              0.400000  -0.100000  -0.100000
A(1) =  1.000000   3.000000   1.000000   Ainv(1) = -0.100000   0.400000  -0.100000
        1.000000   1.000000   3.000000             -0.100000  -0.100000   0.400000

       -3.000000   1.000000   0.000000             -0.500000  -0.500000  -0.500000
A(2) =  1.000000  -2.000000   1.000000   Ainv(2) = -0.500000  -1.500000  -1.500000
        0.000000   1.000000  -1.000000             -0.500000  -1.500000  -2.500000

         -1  +0i     1  -2i     0  +0i              0.250+0.000i   0.250-0.500i   0.500+0.250i
A(0) =    1  +2i     0  +0i     0  -1i   Ainv(0) =  0.250+0.500i   0.250-0.000i  -0.000+0.250i
          0  +0i     0  +1i     1  +0i              0.500-0.250i  -0.000-0.250i   1.250+0.000i

          1  +0i     1  +1i     0  +2i              0.220-0.000i  -0.000+0.146i   0.073+0.317i
A(1) =    1  -1i     5  +0i    -3  +0i   Ainv(1) =  0.000-0.146i   0.098-0.000i  -0.122-0.049i
          0  -2i    -3  +0i     0  +0i              0.073-0.317i  -0.122+0.049i  -0.073-0.000i

          1  +0i     1  +2i     2 +10i             10.000+1.000i  -2.000+6.000i  -3.000-2.000i
A(2) =    1  +1i     0  +3i    -5 +14i   Ainv(2) =  9.000-3.000i   0.000+8.000i  -3.000-2.000i
          1  +1i     0  +5i    -8 +20i             -2.000+2.000i  -1.000-2.000i   1.000+0.000i

```

Batched solver (single right-hand side)
```
        0.000000   4.000000   1.000000              9.000000              1.000000
A(0) =  1.000000   1.000000   3.000000      b(0) =  6.000000      x(0) =  2.000000
        2.000000  -2.000000   1.000000             -1.000000              1.000000

        1.000000   0.500000   0.200000              4.000000              6.339286
A(1) =  0.500000   1.000000   0.500000      b(1) = -1.000000      x(1) = -6.714286
        0.200000   0.500000   1.000000              3.000000              5.089286

        2.000000   0.000000   0.000000              6.000000              3.000000
A(2) =  1.000000   5.000000   0.000000      b(2) =  2.000000      x(2) = -0.200000
        7.000000   9.000000   8.000000              5.000000             -1.775000

         -2  +3i     1  +0i     0  +2i               -2  +5i                0  +1i
A(0) =    0  -5i     3  +0i     1  -1i      b(0) =   11  +0i      x(0) =    1  +1i
          3  -2i    -2  +0i     2  +0i                6  +1i                3  +0i

          1  -5i    -3  +0i     1  -1i               15 -14i                3  +2i
A(1) =    2  +0i     0  +1i     5  +0i      b(1) =    6  -2i      x(1) =   -1  +0i
          1  +0i    -3  +2i    -1  +0i                6  +1i               -0  -1i

          5  -1i     0  +3i     1  +0i                8  +5i                1  +0i
A(2) =    0  +1i     2  +1i    -4  +0i      b(2) =   -8  +3i      x(2) =    2  +0i
          2  -3i     1  +0i     3  +0i               13  -3i                3  -0i

```


## Building

See [INSTALL](INSTALL.md) text file.

Continuous integration:
- [![Build Status](https://travis-ci.org/EncovGroup/CudaBatchedSolver.svg?branch=develop)](https://travis-ci.org/EncovGroup/CudaBatchedSolver) develop branch
 
 
## License
[BSD 3-Clause License](LICENSE.md), 2011-2013, NVIDIA Corporation