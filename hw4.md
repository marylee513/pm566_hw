hw4
================
Yiping Li
2022-11-17

\#HPC \##Q1: Rewrite the following R functions to make them faster

``` r
# Total row sums
fun1 <- function(mat) {
  n <- nrow(mat)
  ans <- double(n) 
  for (i in 1:n) {
    ans[i] <- sum(mat[i, ])
  }
  ans
}

fun1alt <- function(mat) {
  # YOUR CODE HERE, wk10 lab q3
  rowSums(mat)
}

# Cumulative sum by row
fun2 <- function(mat) {
  n <- nrow(mat)
  k <- ncol(mat)
  ans <- mat
  for (i in 1:n) {
    for (j in 2:k) {
      ans[i,j] <- mat[i, j] + ans[i, j - 1]
    }
  }
  ans
}

fun2alt <- function(mat) {
  # YOUR CODE HERE
  t(apply(mat, 1, cumsum))
}

# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)

# Test for the first
library(microbenchmark)
microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat), check = "equivalent"
)
```

    ## Unit: microseconds
    ##          expr     min       lq      mean  median      uq      max neval
    ##     fun1(dat) 367.302 385.2505 426.71126 395.146 428.799  899.041   100
    ##  fun1alt(dat)  35.292  35.7020  49.03884  36.402  37.781 1014.542   100

``` r
# Test for the second
microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat), check = "equivalent"
)
```

    ## Unit: microseconds
    ##          expr      min        lq     mean   median       uq      max neval
    ##     fun2(dat) 2008.640 2086.7285 2248.773 2145.959 2287.872  3744.25   100
    ##  fun2alt(dat)  571.749  894.6055 1343.487  947.579 1086.318 21191.47   100

``` r
#get rid off unit = "relative" due to error
```

\##Q2: Make things run faster with parallel computing. The following
function allows simulating PI

``` r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

    ## [1] 3.132

In order to get accurate estimates, we can run this function multiple
times, with the following code:

``` r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

    ## [1] 3.14124

    ##    user  system elapsed 
    ##   3.204   0.986   4.291

Rewrite the previous code using parLapply() to make it run faster. Make
sure you set the seed using clusterSetRNGStream():

``` r
library(parallel)
cl <- makePSOCKcluster(4)
clusterSetRNGStream(cl, 1231)
clusterExport(cl,varlist = c("sim_pi"), envir = environment())

# YOUR CODE HERE
system.time({
  # YOUR CODE HERE
  ans <- unlist(parLapply(cl, 1:4000, sim_pi, n = 10000))
  print(mean(ans))
  stopCluster(cl)
})
```

    ## [1] 3.141578

    ##    user  system elapsed 
    ##   0.004   0.001   2.520

\#SQL Setup a temporary database by running the following chunk

``` r
# install.packages(c("RSQLite", "DBI"))
library(RSQLite)
library(DBI)

# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
film <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film.csv")
film_category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film_category.csv")
category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/category.csv")

# Copy data.frames to database
dbWriteTable(con, "film", film)
dbWriteTable(con, "film_category", film_category)
dbWriteTable(con, "category", category)
```

When you write a new chunk, remember to replace the r with sql,
connection=con. Some of these questions will reqruire you to use an
inner join. Read more about them here
<https://www.w3schools.com/sql/sql_join_inner.asp>

\##Q1: How many many movies is there avaliable in each rating catagory.
(#or COUNT(\*) AS N)

``` sql
SELECT rating, COUNT (film_id) AS movie_count
FROM film
GROUP BY rating
ORDER BY movie_count
```

| rating | movie_count |
|:-------|------------:|
| G      |         180 |
| PG     |         194 |
| R      |         195 |
| NC-17  |         210 |
| PG-13  |         223 |

5 records

\##Q2: What is the average replacement cost and rental rate for each
rating category.

``` sql
SELECT rating, 
        AVG(replacement_cost) AS avg_replacementcost,
        AVG(rental_rate) AS avg_rentalrate
FROM film
GROUP BY rating
```

| rating | avg_replacementcost | avg_rentalrate |
|:-------|--------------------:|---------------:|
| G      |            20.12333 |       2.912222 |
| NC-17  |            20.13762 |       2.970952 |
| PG     |            18.95907 |       3.051856 |
| PG-13  |            20.40256 |       3.034843 |
| R      |            20.23103 |       2.938718 |

5 records

\##Q3: Use table film_category together with film to find the how many
films there are witth each category ID

``` sql
SELECT category_id, COUNT(category_id) AS film_count
FROM film_category AS fc INNER JOIN film AS f 
ON fc.film_id = f.film_id
GROUP BY category_id
```

| category_id | film_count |
|:------------|-----------:|
| 1           |         64 |
| 2           |         66 |
| 3           |         60 |
| 4           |         57 |
| 5           |         58 |
| 6           |         68 |
| 7           |         62 |
| 8           |         69 |
| 9           |         73 |
| 10          |         61 |

Displaying records 1 - 10

\##Q4: Incorporate table category into the answer to the previous
question to find the name of the most popular category.

``` sql
SELECT fc.category_id, name, COUNT(fc.category_id) AS film_count
FROM film_category AS fc INNER JOIN film AS f 
ON fc.film_id = f.film_id
LEFT JOIN category AS c
ON fc.category_id = c.category_id
GROUP BY fc.category_id
```

| category_id | name        | film_count |
|:------------|:------------|-----------:|
| 1           | Action      |         64 |
| 2           | Animation   |         66 |
| 3           | Children    |         60 |
| 4           | Classics    |         57 |
| 5           | Comedy      |         58 |
| 6           | Documentary |         68 |
| 7           | Drama       |         62 |
| 8           | Family      |         69 |
| 9           | Foreign     |         73 |
| 10          | Games       |         61 |

Displaying records 1 - 10
