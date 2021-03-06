[![Build Status](https://travis-ci.org/mannau/h5.svg?branch=master)](https://travis-ci.org/mannau/h5) 
[![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/mannau/h5?branch=master&svg=true)](https://ci.appveyor.com/project/mannau/h5)
[![codecov.io](http://codecov.io/github/mannau/h5/coverage.svg?branch=master)](http://codecov.io/github/mannau/h5?branch=master) 
[![CRAN_Status_Badge](http://www.r-pkg.org/badges/version/h5)](http://cran.r-project.org/package=h5)

**[h5](http://cran.r-project.org/web/packages/h5/index.html)** is an R 
interface to the [HDF5](https://www.hdfgroup.org/HDF5) library under active development. It is available on [Github](https://github.com/mannau/h5) and already released on [CRAN](https://cran.r-project.org/web/packages/h5/index.html) for all major platforms (Windows, OS X, Linux). 
Online documentation for the package is available at http://h5.predictingdaemon.com.

[HDF5](https://www.hdfgroup.org/HDF5/) is an excellent library and data model to 
store huge amounts of data in a binary file format. Supporting most major 
platforms and programming languages it can be used to exchange data files in a 
language independent format. Compared to R's integrated *save()* and *load()* 
functions it also supports access to only parts of the binary data files and can
therefore be used to process data not fitting into memory.

**[h5](http://cran.r-project.org/web/packages/h5/index.html)** utilizes the 
[HDF5 C++ API](https://www.hdfgroup.org/HDF5/doc/cpplus_RM/) through 
**[Rcpp](http://cran.r-project.org/web/packages/Rcpp/index.html)** and S4 classes. 
The package is covered by 200+ test cases with a [coverage](https://codecov.io/github/mannau/h5?branch=master) greater than 80%.

# Install
**h5** has already been released on [CRAN](https://cran.r-project.org/web/packages/h5/index.html), and can therefore be installed using

```python
install.packages("h5")
```

The most recent development version can be installed from [Github](https://github.com/mannau/h5) using [**devtools**](https://cran.r-project.org/web/packages/devtools/index.html):

```python
library(devtools)
install_github("mannau/h5")
```
Please note that this version has been tested with the current hdf5 library 1.10.0patch1 (and 1.8.16 for OS X) - you should therefore install the most current hdf5 library including its C++ API for your platform. The minimum version HDF5 version required is 1.8.12.

## Requirements

### Windows
This package already ships the library for windows operating systems through [h5-libwin](https://github.com/mannau/h5-libwin). No additional requirements need to be installed.


### OS X
Using OS X and [Homebrew](http://brew.sh) you can use the following command to install HDF5 library dependencies and headers:
```shell
brew install homebrew/science/hdf5 --enable-cxx
```

### Linux (e.g. Debian >= 8.0, Ubuntu >= 15.04)
With Debian-based Linux systems you can use the following command to install the dependencies:
```shell
sudo apt-get install libhdf5-dev
```

### Linux (e.g. Debian < 8.0, Ubuntu < 15.04)
For older versions (or to use the latest version) it is required to install the library from source using the following steps (e.g for HDF5 ver 1.10patch1):
```shell
wget https://www.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.0-patch1/src/hdf5-1.10.0-patch1.tar.gz
tar -xzf hdf5-1.10.0-patch1.tar.gz
cd hdf5-1.10.0-patch1
./configure --prefix=/usr/local --enable-cxx --enable-build-mode=production 
sudo make install
```

## Custom Install Parameters
If the hdf5 library is not located in a standard directory recognized by the configure script the parameters CPPFLAGS and LIBS may need to be set manually. 
This can be done using the --configure-vars option for R CMD INSTALL in the command line, e.g
```shell
R CMD INSTALL h5_<version>.tar.gz --configure-vars='LIBS=<LIBS> CPPFLAGS=<CPPFLAGS>'
```

The most recent version with required paramters can also be directly installed from github using **devtools** in R:
```shell
require(devtools)
install_github("mannau/h5", args = "--configure-vars='LIBS=<LIBS> CPPFLAGS=<CPPFLAGS>'")
```

A concrete OS X example setting could look like this:
```shell
R CMD INSTALL h5_0.9.2.tar.gz --configure-vars='LIBS=-L/usr/local/Cellar/hdf5/1.8.13/lib -L/usr/local/opt/szip/lib  -L. -lhdf5_cpp -lhdf5 -lz -lm CPPFLAGS=-I/usr/local/include -I/usr/local/include/freetype2 -I/opt/X11/include'
```

# Quick Start

We start by creating an HDF5 file holding a numeric vector, an integer matrix and a character array.


```r
library(h5)
testvec <- rnorm(10)
testmat <- matrix(1:9, nrow = 3)
row.names(testmat) <- 1:3
colnames(testmat) <- c("A", "BE", "BU")
letters1 <- paste(LETTERS[runif(45, min = 1, max = length(LETTERS))])
letters2 <- paste(LETTERS[runif(45, min = 1, max = length(LETTERS))])
testarray <- array(paste0(letters1, letters2), c(3, 3, 5))

file <- h5file("test.h5")
# Save testvec in group 'test' as DataSet 'testvec'
file["test/testvec"] <- testvec
file["test/testmat"] <- testmat
file["test/testarray"] <- testarray
h5close(file)
```

We can now retrieve the data from the file


```r
file <- h5file("test.h5")
dataset_testmat <- file["test/testmat"]
# We can now retrieve all data from the DataSet object using e.g. the  subsetting operator
dataset_testmat[]
```

```
##      [,1] [,2] [,3]
## [1,]    1    4    7
## [2,]    2    5    8
## [3,]    3    6    9
```

We can also subset the data directly, e.g. row 1 and 3

```r
dataset_testmat[c(1, 3), ]
```

```
##      [,1] [,2] [,3]
## [1,]    1    4    7
## [2,]    3    6    9
```

Note, that we have now lost the row- and column names associated with the *testmat* object
in the retrieved matrix. HDF5 supports metadata with attributes, which we need to
add to (retrieve from) the DataSet manually.


```r
h5attr(dataset_testmat, "rownames") <- row.names(testmat)
h5attr(dataset_testmat, "colnames") <- colnames(testmat)
```

We can now retrieve our matrix including meta-data as follows:



```r
outmat <- dataset_testmat[]
row.names(outmat) <- h5attr(dataset_testmat, "rownames")
colnames(outmat) <- h5attr(dataset_testmat, "colnames")
identical(outmat, testmat)

h5close(dataset_testmat)
```

```
## [1] TRUE
```

Do not forget to close the HDF5 file in the end


```r
h5close(file)
```

# License [![License](https://img.shields.io/badge/license-BSD%202%20clause-blue.svg?style=flat)](http://opensource.org/licenses/BSD-2-Clause)

This package is shipped with a [BSD-2-Clause License](http://opensource.org/licenses/BSD-2-Clause). 

