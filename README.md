## Randomized Extended Kaczmarz (C) <img src="https://travis-ci.org/zouzias/REK-C.svg?branch=master"/>

The Randomized Extended Kaczmarz algorithm is a randomized algorithm for solving least-squares/linear regression problems. The current project contains a C implementation of the algorithm.

[Randomized Extended Kaczmarz for Solving Least-Squares.](http://dx.doi.org/10.1137/120889897)
SIAM. J. Matrix Anal. & Appl., 34(2), 773–793. (21 pages) 
Authors: [Anastasios Zouzias](https://github.com/zouzias) and Nikolaos Freris 
### Installation

Clone the project. Type type

```
make && make test
./main
```

The above code runs a simple instance of least-squares for a gaussian random matrix A and gaussian vector b. See 'main.c' for more details.

### Usage

```
#include <stdio.h>
#include <string.h>
#include <time.h>
#include <math.h>
#include <stddef.h>
#include "algorithms/REK/REKBLAS.h"
#include "matrix/sparseMatrix.h"
#include "matrix/matrix.h"
#include "utils/utils.h"

int main() {
    
    /* initialize random seed */
    srand(time(NULL));

    // Dimensions of matrix A (m x n).
    int m = 5000, n = 500;
    
    // Maximum number of iterations
    double TOL = 10e-8;
    
    // Matrix sparsity (% of nnz)
    double sparsity = 0.25;
    
    // Allocating space for unknown vector x, 
    // right size vector b, and solution vector xls, for Ax = b
    double *xls = gaussianVector(n);
    double* b = (double*) malloc( m * sizeof(double));
    double* x = (double*) malloc( n * sizeof(double));
    memset (x, 0, n * sizeof (double));
    
    // Sparse input matrix
    SMAT *As = fillSparseMat(m, n, sparsity);
    
    // Set b = As * x
    myDGEMVSparse(As,xls, b);
    
    printf("REK for sparse (%d x %d)-matrix with sparsity %f.\n", m, n, sparsity);
    sparseREK (As, x, b, TOL);
    double error = lsErrorSparse(As, x, b);
    
    printf("Sparse REK: LS error is : %e\n", error);

    freeSMAT(As); free(xls); free(x); free(b);
    return 0;
}
```

#### Bugs

Please report bugs by opening a new [issue](https://github.com/zouzias/REK-CPP/issues/new).

#### Implementation Details

REK-BLAS is an implementation of REK with two additional technical features. First, REK-BLAS utilizes level-1 BLAS routines for 
all operations of REK and second REK-BLAS additionally stores explicitly the transpose of A for more efficiently 
memory access of both the rows and columns of A using BLAS (see the above paper for more details). 

The sampling operations of REK are implemented using the so-called ``alias method`` for generating samples 
from any given discrete distribution [Vos91]. In particular, the alias method, assuming access 
to a uniform random variable on [0,1] in constant time and linear time preprocessing, generates one sample
of a given distribution in constant time. We use an implementation of W. D. Smith.

### Credits and acknowledgments

* Credits go to Warren D. Smith for implementing the aliasing method [Vos91] in C.
* [Vos91] M. D. Vose. A Linear Algorithm for Generating Random Numbers with a given Distribution. IEEE Trans. Softw. Eng., 17(9):972–975, September 1991.
