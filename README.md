
<!-- README.md is generated from README.Rmd. Please edit that file -->

# lc – List comprehension in R

[![Travis build
status](https://travis-ci.org/mailund/lc.svg?branch=master)](https://travis-ci.org/mailund/lc)
[![AppVeyor build
status](https://ci.appveyor.com/api/projects/status/wsopc251n1jpj40j/branch/master?svg=true)](https://ci.appveyor.com/project/mailund/lc/branch/master)
[![Coverage
Status](https://img.shields.io/codecov/c/github/mailund/lc/master.svg)](https://codecov.io/github/mailund/lc?branch=master)
[![Coverage
Status](https://coveralls.io/repos/github/mailund/lc/badge.svg?branch=master)](https://coveralls.io/github/mailund/lc?branch=master)
[![lifecycle](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)

The goal of `lc` is to implement Haskell- and Python-like list
comprehension in R. List comprehensions provide a syntax for mapping and
filtering sequences. In R we would use functions such as `Map` or
`Filter`, or the `purrr` alternatives, for this, but in languages such
as Haskell or Python, there is syntactic sugar to make combinations of
mapping and filtering easier to program.

## Installation

You can install lc from GitHub with:

``` r
install.packages("devtools")
devtools::install_github("mailund/lc")
```

or from CRAN using

``` r
install.packages("lc")
```

## Examples

### Quick-sort

The algorithm [quick-sort](https://en.wikipedia.org/wiki/Quicksort) is
based on the idea that to sort a list you pick a random element in it,
called the *pivot*, split the data into those elements smaller than the
pivot, equal to the pivot and larger than the pivot. Then sort those
smaller and larger elements recursively and concatenate the three lists
to get the final sorted list. One way to implement this in R is using
the `Filter` function:

``` r
qsort <- function(lst) {
  n <- length(lst)
  if (n < 2) return(lst)

  pivot <- lst[[sample(n, size = 1)]]
  smaller <- Filter(function(x) x < pivot, lst)
  equal <- Filter(function(x) x == pivot, lst)
  larger <- Filter(function(x) x > pivot, lst)
  c(qsort(smaller), equal, qsort(larger))
}

(lst <- sample(1:10))
#>  [1]  1  4  3  8  5  6  7  9 10  2
unlist(qsort(lst))
#>  [1]  1  2  3  4  5  6  7  8  9 10
```

Using `lc`, you can implement the same algorithm like this:

``` r
qsort <- function(lst) {
  n <- length(lst)
  if (n < 2) return(lst)
  
  pivot <- lst[[sample(n, size = 1)]]
  smaller <- lc(x, x = lst, x < pivot)
  equal <- lc(x, x = lst, x == pivot)
  larger <- lc(x, x = lst, x > pivot)
  
  c(qsort(smaller), equal, qsort(larger))
}

(lst <- sample(1:10))
#>  [1]  6  5  3  7  2  8  1  9 10  4
unlist(qsort(lst))
#>  [1]  1  2  3  4  5  6  7  8  9 10
```

### Eratosthenes sieve

The [Eratosthenes
sieve](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes) algorithm
identifies the primes in the first `n` natural numbers by removing those
elements that are divisible by smaller numbers. We can implement it
using `lc` like this:

``` r
get_primes <- function(n) {
  not_primes <- lc(seq(from = 2*x, to = n, by = x), x = 2:sqrt(n)) %>% unlist %>% unique
  lc(p, p = 1:n, !(p %in% not_primes)) %>% unlist
}
get_primes(100)
#>  [1]  1  2  3  5  7 11 13 17 19 23 29 31 37 41 43 47 53 59 61 67 71 73 79
#> [24] 83 89 97
```

Here, we create a list of all the non-primes first and then identifies
the primes as those that are left. Alternatively, we can remove the
non-primes iteratively while identifying the primes. For the
implementation of this idea, we need to grow a list of primes. We don’t
want to do that using `list`, since this will give us a quadratic time
algorithm, so I use the [`pmatch`
package](https://github.com/mailund/pmatch) to implement a linked list
first:

``` r
library(pmatch)

linked_list := NIL | CONS(car, cdr : linked_list)

ll_length <- function(x, acc = 0) {
  cases(x,
        NIL -> acc,
        CONS(car,cdr) -> ll_length(cdr, acc + 1))
}
ll_to_list <- function(x) {
  y <- vector("list", length = ll_length(x))
  f <- function(x, i) {
    cases(x,
          NIL -> NULL,
          CONS(car, cdr) -> {
            y[[i]] <<- car
            f(cdr, i + 1)
        })
  }
  f(x, 1)
  y
}
```

The `get_primes` function can now be implemented like this:

``` r
get_primes <- function(n) {
  candidates <- 2:n
  primes <- NIL
  while (length(candidates) > 0) {
    p <- candidates[[1]]
    primes <- CONS(p, primes)
    candidates <- lc(x, x = candidates, x %% p != 0)
  }
  primes %>% ll_to_list %>% unlist %>% rev
}
get_primes(100) 
#>  [1]  2  3  5  7 11 13 17 19 23 29 31 37 41 43 47 53 59 61 67 71 73 79 83
#> [24] 89 97
```
