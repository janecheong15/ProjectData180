# ProjectData180
This is a project for Data 180
Instructions on how to code using R Languages 

Lecture on Iterating over a vector 

## Iterating over a vector

* For loops allow iteration.
* A common scenario for iteration is that our data is in a vector (list), 
and we want to perform the same operation on 
each element.
* Such iteration is so common that special tools have been developed
with the aim of reducing the amount of code (and therefore
errors) required for common iterative tasks.
    - Tools in base R include the `apply()`
    family of functions. 
    - A tidyverse package called `purrr` includes more.
    
## Example data

* To illustrate iteration we simulate data and fit four 
regression models.

\scriptsize

```{r}
set.seed(42)
n <- 100
x1 <- rnorm(n); x2<-rnorm(n)
y1 <- x1 + rnorm(n,sd=.5); y2 <- x1+x2+rnorm(n,sd=.5)
y3 <- x2 + rnorm(n,sd=.5); y4 <- rnorm(n,sd=.5)
rr <- list(fit1 = lm(y1 ~ x1+x2),
  fit2 = lm(y2 ~ x1+x2),
  fit3 = lm(y3 ~ x1+x2),
  fit4 = lm(y4 ~ x1+x2))
coef(rr$fit1)
```

## Exercise

* The elements of the list `rr` from last slide are
`lm` objects. The function `coef()` is generic.
Assign class "lm_vec" to `rr` and write a 
`coef()` method for objects of this class.
Hint: Your function could encapsulate 
a `for()` loop like the following.

```{r}
for(i in seq_along(rr)) { # safer than 1:length(rr) 
  print(coef(rr[[i]]))
}
```


## Extracting the regression coefficient for `x1`

* Using a `for()` loop, we initialize an object to hold the 
**output**, 
loop along a **sequence** of values for an index variable, 
and execute the **body** for each value of the index variable.

\small

```{r}
betahat <- vector("double",length(rr))
for(i in seq_along(rr)) { # safer than 1:length(rr) 
  betahat[i] <- coef(rr[[i]])["x1"]
}
betahat
```

## Looping over elements of a set

* The index set in the `for()` loop can be general. 
    - We might use this generality to loop over named components of a list.
    
\small

```{r}
fits <- paste0("fit",1:4)
for(ff in fits) {
  print(coef(rr[[ff]])["x1"])
}
```

\normalsize

* Looping over a set makes it harder to save the results, though.

## Avoid growing vectors incrementally

\scriptsize

```{r}
means <- seq.int(1000)
set.seed(123)
system.time({
output <- double()
for (i in seq_along(means)) {
  n <- sample(100, 1)
  output <- c(output, rnorm(n, means[[i]]))
}
})
```

##

\scriptsize

```{r}
system.time({ 
out <- vector("list", length(means))
for (i in seq_along(means)) {
  n <- sample(100, 1)
  out[[i]] <- rnorm(n, means[[i]])
}
out <- unlist(out)
})
```

##  `bind_cols()` and `bind_rows()`

\scriptsize

```{r}
out <- vector("list", length(means))
n <- 100
for (i in seq_along(means)) {
  out[[i]] <- rnorm(n, means[[i]])
}
out <- bind_cols(out) 
out <- vector("list", length(means))
for (i in seq_along(means)) {
  out[[i]] <- tibble(y=rnorm(n, means[[i]]),x=rnorm(n))
}
out <- bind_rows(out) 
```


## The body of a loop can be a small part of the code

* In our examples, most of the code is for setting up the 
output and looping, with very little to do with the body.

* To illustrate, consider a small change: instead of the estimated coefficient of `x1` we wanted
the estimated coefficient of `x2`:

\small

```{r}
betahat <- vector("double",length(rr))
for(i in seq_along(rr)) { # safter than 1:length(rr) 
  betahat[i] <- coef(rr[[i]])["x2"]
}
betahat
```

## Exercise

* Write a `for()` loop to find the `mode()` of
each column in nycflights13::flights


## Using `lapply()`

* The intent of `lapply()` is to take care of 
the output and the loop, allowing us to focus 
on the body.

\scriptsize

```{r}
b1fun <- function(fit) { coef(fit)["x1"] } # body
bfun <- function(fit,cc) { coef(fit)[cc] } # body
lapply(rr,b1fun)  # or sapply(rr,b1fun) or unlist(lapply(rr,b1fun))
lapply(rr,bfun,"x1")  
```


## Exercise

* Re-write your `coef()` method for objects of 
class `lm_vec` to use `lapply()`.

## Iterating with the `map()` functions from `purrr`

* The `purrr` package provides a family of functions 
`map()`, `map_dbl()`, etc. that do the same thing
as `lapply()` but work better with other tidyverse functions.
    * `map()` returns a list, like `lapply()`.
    * `map_dbl()` returns a double vector, etc.

\scriptsize

```{r}
library(purrr)
map_dbl(rr,b1fun) # or rr %>% map_dbl(b1fun)
# map_dbl(rr,bfun,"x1")
```

## Exercises

* Use `map_chr()` to return the `mode()` 
of each column of the `nycflights13::flights`
tibble.
* Use `map()` to return the `summary()`
of each column of the `nycflights13::flights`
tibble.

## Pipes and `map()` functions

* Suppose we want to record a model summary returned by 
the `summary()` function.
    * `summary()` applied to
    an `lm()` object it computes regression summaries like
    standard errors and model R$^2$.

\small

```{r}
rr %>%
  map(summary) %>%
  map_dbl(function(ss) { ss$r.squared })
```

\normalsize

## 

* Notice that we can define a function on-the-fly in the
call to a `map()` function.

* `map()` functions have a short-cut for function
definitions.

\small

```{r}
rr %>%
  map(summary) %>%
  map_dbl(~.$r.squared) # or map_dbl("r.squared")
```

\normalsize

* In `~.` read `~` as "define a function" and `.` as "argument to 
the function"

## Exercise

* Write a call to `map_dbl()` that
does the same thing as `map_dbl(rr,b1fun)`,
but define the function on the fly, as in 
the previous slide. You can use multiple 
calls to `map()` functions.

## Detour: The apply family of functions in R

\small

- The "original" apply is `apply()`, which can 
be used to apply a function to rows or columns 
of a matrix.

\scriptsize

```{r}
mat <- matrix(1:6,ncol=2,nrow=3)
mat
apply(mat,1,sum) # row-wise sums; rowSums() is faster
apply(mat,2,sum) # column-wise; colSums() is faster
```

## Detour, cont.

\small

- `sapply()` takes the output of `lapply()` and simplifies
to a vector or matrix.

\scriptsize

```{r}
sapply(rr,coef)
````

    
