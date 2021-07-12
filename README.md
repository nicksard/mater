Getting started with mater
================
Nick Sard

## Installing mater

This package is currently only available on Github. Thus,
library(devtools) is needed to install it.

``` r
#install.packages("devtools") 
#delete the first '#' in the above line and run it if you don't have devtools installed
library(devtools)
install_github(repo = "nicksard/mater")
```

The above lines of code need to only be run once and in all subsequent
uses just load the library before using its functions (see below).  
Once the package is installed, load the library.

``` r
library(mater)
```

## Basic application of the library

The typical workflow in mater includes these steps:  
1\. Make a breeding matrix with brd.mat()  
2\. Fill the breeding matrix with some number of offspring per mate pair
(non-zero cells in the matrix) with brd.mat.fitness()  
3\. Randomly subsample the breeding matrix containing all offspring
produced by all parents with mat.sub.sample()  
4\. Convert the output of mat.sub.sample(), a data frame summarizing
offspring per mate pair actually sampled, to a pedigree with
convert2ped()  

``` r
set.seed(123) #creating a reproducible example
```

``` r
#Step 1)
mat1 <- brd.mat(moms = 50,dads = 50,lambda.low = 4,lambda.high = 4)
```

    ## [1] "Lambda deviations have been minimized - differences = -1"

``` r
head(mat1[1:10,1:10])
```

    ##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
    ## [1,]    0    0    0    0    0    0    0    0    0     0
    ## [2,]    0    0    0    1    0    0    0    0    0     0
    ## [3,]    0    0    0    0    0    0    0    0    0     0
    ## [4,]    0    0    0    0    0    0    0    0    0     0
    ## [5,]    0    0    0    0    0    0    0    0    0     0
    ## [6,]    1    0    0    0    1    0    0    0    0     0

Note the print statement that indicates deviations from random draws of
mates per some specific lambda, in this case 4, have been minimized such
that only one parent has one fewer mates than they should. As noted in
the documentation for this function the number of columns represents the
number of females, and the number of rows represents the number of
males, and 0 and 1 represent if specific mate pairs do not mate or mate.

``` r
?brd.mat #run this line for more documentation associated with the function.
#Similarly, the ? can be placed in front of any of mater's functions to access associated documentation
```

Now that the breeding matrix structure has been defined, offspring
produced by each mate pair are created using brd.mat.fitness()

``` r
#Step 2)
mat1_off <- brd.mat.fitness(mat = mat1,min.fert = 2500,max.fert = 6500,type = "uniform")
head(mat1_off[1:10,1:10])
```

    ##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
    ## [1,]    0    0    0    0    0    0    0    0    0     0
    ## [2,]    0    0    0 2013    0    0    0    0    0     0
    ## [3,]    0    0    0    0    0    0    0    0    0     0
    ## [4,]    0    0    0    0    0    0    0    0    0     0
    ## [5,]    0    0    0    0    0    0    0    0    0     0
    ## [6,] 1873    0    0    0 1193    0    0    0    0     0

Users can quickly get some summary statistics associated with a breeding
matrix by using mat.stats()

``` r
mat.stats(mat = mat1_off,id.col = F)
```

    ##   n.par n.mom n.dad sex.ratio   noff female.rs females.mates male.rs male.mates
    ## 1   100    50    50         1 232983   4659.66          3.12 4659.66       3.12
    ##   mp.count  mp.rs
    ## 1      156 93.193

The above table tells the user that 100 parents created hundreds of
thousands of offspring. Most pedigrees genotype hundreds to maybe two or
three thousand offspring. The function mat.sub.sample() can be used to
randomly subsample the ‘full’ breeding matrix (mat1\_off). In this
example, 200 offspring are randomly subsampled to represent something
more typical of what a researcher might observe after genotyping 200
offspring.

``` r
#Step 3)
df <- mat.sub.sample(mat = mat1_off,noff = 200)
head(df)
```

    ##              mp   dads   moms  off       probs off1
    ## 1  dads1_moms30  dads1 moms30 1584 0.006798779    0
    ## 2 dads10_moms10 dads10 moms10  738 0.003167613    2
    ## 3 dads10_moms23 dads10 moms23 1903 0.008167978    1
    ## 4 dads10_moms41 dads10 moms41  831 0.003566784    0
    ## 5 dads11_moms28 dads11 moms28  620 0.002661138    0
    ## 6  dads11_moms3 dads11  moms3  637 0.002734105    1

The above data frame summarizes how many offspring were produced by each
mate pair (off), the probability of each mate pair being sampled
(probs), and how many times each mate pair was actually sampled when 200
offspring were randomly collected (off1). The information in this data
frame can be converted to a typical three column pedigree where each row
represents an offspring and the parents that produced it using
convert2ped().

``` r
#Step 4)
ped <- convert2ped(df = df)
head(ped)
```

    ##                off    mom    dad
    ## 1 o1_moms10_dads10 moms10 dads10
    ## 2 o2_moms10_dads10 moms10 dads10
    ## 3 o1_moms23_dads10 moms23 dads10
    ## 4  o1_moms3_dads11  moms3 dads11
    ## 5 o1_moms32_dads11 moms32 dads11
    ## 6 o1_moms49_dads12 moms49 dads12

This information can be used in subsequent analyses of interest to the
user. The original application was to estimate the number of
successfully breeding adults in a population, but there may be other
applications.

A final comment, the pedigree can be converted back to a breeding matrix
that can used to get additional summary statistics.

``` r
mat1_off_sub <- ped2mat(ped = ped)
mat.stats(mat = mat1_off_sub)
```

    ##   n.par n.mom n.dad sex.ratio noff female.rs females.mates male.rs male.mates
    ## 1    90    47    43      0.91  200      4.26          2.21    4.65       2.42
    ##   mp.count mp.rs
    ## 1      104 0.099
