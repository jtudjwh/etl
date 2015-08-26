[![Travis-CI Build Status](https://travis-ci.org/beanumber/etl.svg?branch=master)](https://travis-ci.org/beanumber/etl)

# etl
R package to facilitate [ETL](https://en.wikipedia.org/wiki/Extract,_transform,_load) operations

```{r, message=FALSE}
require(etl)
require(RPostgreSQL)
require(dplyr)
db <- src_postgres(dbname = "mtcars", user = "postgres", host = "localhost")
etl_cars <- etl_connect("mtcars", db)
str(etl_cars)
```

## Step-by-step

Initialize the database

```{r}
etl_cars <- etl_cars %>%
  etl_init()
str(etl_cars)
```

Download the raw data

```{r}
etl_cars <- etl_cars %>%
  etl_scrape()
list.files(etl_cars$dir)
```

Do any data processing

```{r}
etl_cars <- etl_process(etl_cars)
```

Push the data to the database

```{r}
etl_cars <- etl_cars %>%
  etl_push()
str(etl_cars)
```

Do any data cleanup

```{r}
etl_cars <- etl_cleanup(etl_cars)
```

## Streamlined

OR, do the whole thing in one step!

```{r}
etl_cars <- etl_connect("mtcars", db) %>%
  etl_create()
```

You can also update an existing database without re-initializing, but watch out for primary key collisions.

```{r, eval=FALSE}
etl_cars <- etl_connect("mtcars", db) %>%
  etl_update()
```

## Do Your Analysis

Now that your database is populated, you can work with it as a `src` data table just like any other `dplyr` table. 
```{r}
db %>%
  tbl(from = "mtcars") %>%
  group_by(cyl) %>%
  summarise(N = n(), meanMPG = mean(mpg))
```

## Create your own ETL packages

Suppose you want to create your own ETL package called `pkgname`. All you have to do is write a package that requires `etl`, and then you have to write **two S3 methods**:

```{r, eval=FALSE}
etl_scrape.etl_pkgname()
etl_push.etl_pkgname()
```

You may also wish to write

```{r, eval=FALSE}
etl_init.etl_pkgname()
etl_process.etl_pkgname()
etl_cleanup.etl_pkgname()
```

All of these functions must take and return an object of class `etl_pkgname` that inherits from `etl`. Please see the [`airlines`](https://github.com/beanumber/airlines) package for an example. 