---
title: "Data manipulation across columns"
teaching: 45
exercises: 6
---

:::::::::::::::::::::::::::::::::::::: questions 

- How can I calculate the mean of several columns for every row of data?
- How can I apply the same function across several related columns?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives


::::::::::::::::::::::::::::::::::::::::::::::::



## Motivation

We have covered many topics so far, and changing (or mutating) variables has been a key concept. 
The need to create new columns of data, often based on information in other columns of data, is a type of operation that we need very often.
But some times, you also need to calculate something _per row_ for several solumns. 
For instance, you want the sum of all columns in a certain collection, or the mean of them, how can we do that?

One way, is to write it our entirely.
Let just pretend there is a good reason to get the sum of bill length and bill depth. 
Let us also make a subsetted sample with just the bill measurements so we cab easily see what we are doing.
We can do that in the following way.


```r
penguins_s <- penguins |>
    select(species, starts_with("bill"))

penguins_s |> 
  mutate(
    bill_sum = bill_depth_mm + bill_length_mm
    )
```

```{.output}
# A tibble: 344 × 4
   species bill_length_mm bill_depth_mm bill_sum
   <fct>            <dbl>         <dbl>    <dbl>
 1 Adelie            39.1          18.7     57.8
 2 Adelie            39.5          17.4     56.9
 3 Adelie            40.3          18       58.3
 4 Adelie            NA            NA       NA  
 5 Adelie            36.7          19.3     56  
 6 Adelie            39.3          20.6     59.9
 7 Adelie            38.9          17.8     56.7
 8 Adelie            39.2          19.6     58.8
 9 Adelie            34.1          18.1     52.2
10 Adelie            42            20.2     62.2
# … with 334 more rows
# ℹ Use `print(n = ...)` to see more rows
```

We've seen similar types of operations before.
But what if you want to sum 20 columns, you would need to type our all 20 column names!
Again, tedious. 
We have a special type of operations we can do to get that easily. 
We will use the function `sum` to calculate the sum of several variables when using this pipeline.


```r
penguins_s |>
    mutate(bill_sum = sum(c_across(starts_with("bill"))))
```

```{.output}
# A tibble: 344 × 4
   species bill_length_mm bill_depth_mm bill_sum
   <fct>            <dbl>         <dbl>    <dbl>
 1 Adelie            39.1          18.7       NA
 2 Adelie            39.5          17.4       NA
 3 Adelie            40.3          18         NA
 4 Adelie            NA            NA         NA
 5 Adelie            36.7          19.3       NA
 6 Adelie            39.3          20.6       NA
 7 Adelie            38.9          17.8       NA
 8 Adelie            39.2          19.6       NA
 9 Adelie            34.1          18.1       NA
10 Adelie            42            20.2       NA
# … with 334 more rows
# ℹ Use `print(n = ...)` to see more rows
```

hm, that is not what we expected.
I know why, but the reason is not always easy to understand. 
By default, `c_across` will summarise all the rows for all the bill columns, and give a _single_ value for the entire data set.
There are some `NA`s the entire data set, so it returns `NA`. 
So how can we force it to work in a row-wise fashion?
We can apply a function called `rowwise()` which is a special type of `group_by` that groups your data by each row, so each row is its own group.
Then, `c_across()` will calculate the mean of the columns just for that group (i.e. row in this case).


```r
penguins_s |>
    rowwise() |>
    mutate(bill_sum = sum(c_across(starts_with("bill"))))
```

```{.output}
# A tibble: 344 × 4
# Rowwise: 
   species bill_length_mm bill_depth_mm bill_sum
   <fct>            <dbl>         <dbl>    <dbl>
 1 Adelie            39.1          18.7     57.8
 2 Adelie            39.5          17.4     56.9
 3 Adelie            40.3          18       58.3
 4 Adelie            NA            NA       NA  
 5 Adelie            36.7          19.3     56  
 6 Adelie            39.3          20.6     59.9
 7 Adelie            38.9          17.8     56.7
 8 Adelie            39.2          19.6     58.8
 9 Adelie            34.1          18.1     52.2
10 Adelie            42            20.2     62.2
# … with 334 more rows
# ℹ Use `print(n = ...)` to see more rows
```

Now we can see that we get the row sum of all the bill columns for each row, and the tibble tells us it is "Rowwise".
To stop the data set being rowwise, we can use the `ungroup()` function we learned before.


```r
penguins_s |>
    rowwise() |>
    mutate(bill_sum = sum(c_across(starts_with("bill")))) |>
    ungroup()
```

```{.output}
# A tibble: 344 × 4
   species bill_length_mm bill_depth_mm bill_sum
   <fct>            <dbl>         <dbl>    <dbl>
 1 Adelie            39.1          18.7     57.8
 2 Adelie            39.5          17.4     56.9
 3 Adelie            40.3          18       58.3
 4 Adelie            NA            NA       NA  
 5 Adelie            36.7          19.3     56  
 6 Adelie            39.3          20.6     59.9
 7 Adelie            38.9          17.8     56.7
 8 Adelie            39.2          19.6     58.8
 9 Adelie            34.1          18.1     52.2
10 Adelie            42            20.2     62.2
# … with 334 more rows
# ℹ Use `print(n = ...)` to see more rows
```

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 1
Calculate the mean of all the columns with millimeter measurements, an call it `mm_mean`, for each row of data.

:::::::::::::::::::::::::::::::::::::::: solution 
## Solution 


```r
penguins |> 
  rowwise() |>
  mutate(
    mm_mean = mean(c_across(ends_with("mm")))
  )
```

```{.output}
# A tibble: 344 × 9
# Rowwise: 
   species island    bill_length_mm bill_d…¹ flipp…² body_…³ sex    year mm_mean
   <fct>   <fct>              <dbl>    <dbl>   <int>   <int> <fct> <int>   <dbl>
 1 Adelie  Torgersen           39.1     18.7     181    3750 male   2007    79.6
 2 Adelie  Torgersen           39.5     17.4     186    3800 fema…  2007    81.0
 3 Adelie  Torgersen           40.3     18       195    3250 fema…  2007    84.4
 4 Adelie  Torgersen           NA       NA        NA      NA <NA>   2007    NA  
 5 Adelie  Torgersen           36.7     19.3     193    3450 fema…  2007    83  
 6 Adelie  Torgersen           39.3     20.6     190    3650 male   2007    83.3
 7 Adelie  Torgersen           38.9     17.8     181    3625 fema…  2007    79.2
 8 Adelie  Torgersen           39.2     19.6     195    4675 male   2007    84.6
 9 Adelie  Torgersen           34.1     18.1     193    3475 <NA>   2007    81.7
10 Adelie  Torgersen           42       20.2     190    4250 <NA>   2007    84.1
# … with 334 more rows, and abbreviated variable names ¹​bill_depth_mm,
#   ²​flipper_length_mm, ³​body_mass_g
# ℹ Use `print(n = ...)` to see more rows
```

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 2
Calculate the mean of all the columns with millimeter measurements, an call it `mm_mean`, for each row of data.
Then, group the data by species, and calculate the mean of the `mm_mean` within each species and add it as a column named `mm_mean_species`.
Ignore `NA`s in the last calculation

:::::::::::::::::::::::::::::::::::::::: solution 
## Solution 


```r
penguins |> 
  rowwise() |>
  mutate(
    mm_mean = mean(c_across(ends_with("mm"))),
  ) |>
  group_by(species) |>
  mutate(mm_mean_species = mean(mm_mean, na.rm = TRUE))
```

```{.output}
# A tibble: 344 × 10
# Groups:   species [3]
   species island    bill_…¹ bill_…² flipp…³ body_…⁴ sex    year mm_mean mm_me…⁵
   <fct>   <fct>       <dbl>   <dbl>   <int>   <int> <fct> <int>   <dbl>   <dbl>
 1 Adelie  Torgersen    39.1    18.7     181    3750 male   2007    79.6    82.4
 2 Adelie  Torgersen    39.5    17.4     186    3800 fema…  2007    81.0    82.4
 3 Adelie  Torgersen    40.3    18       195    3250 fema…  2007    84.4    82.4
 4 Adelie  Torgersen    NA      NA        NA      NA <NA>   2007    NA      82.4
 5 Adelie  Torgersen    36.7    19.3     193    3450 fema…  2007    83      82.4
 6 Adelie  Torgersen    39.3    20.6     190    3650 male   2007    83.3    82.4
 7 Adelie  Torgersen    38.9    17.8     181    3625 fema…  2007    79.2    82.4
 8 Adelie  Torgersen    39.2    19.6     195    4675 male   2007    84.6    82.4
 9 Adelie  Torgersen    34.1    18.1     193    3475 <NA>   2007    81.7    82.4
10 Adelie  Torgersen    42      20.2     190    4250 <NA>   2007    84.1    82.4
# … with 334 more rows, and abbreviated variable names ¹​bill_length_mm,
#   ²​bill_depth_mm, ³​flipper_length_mm, ⁴​body_mass_g, ⁵​mm_mean_species
# ℹ Use `print(n = ...)` to see more rows
```

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 


## Mutating several columns in one go

So far, we've been looking at adding or summarising variables one by one, or in a pivoted fashion.
This is of course something we do all the time, but some times we need to do the same change to multiple columns at once. 
Imagine you have a data set with 20 column and you want to scale them all to the same scale.
Writing the same command with different columns 20 times is very tedious. 

In our case, let us say we want to scale the three columns with millimeter measurements so that they have a mean of 0 and standard deviation of 1. 
We've already used the `scale()` function once before, so we will do it again.

In this simple example we might have done so:


```r
penguins |> 
  mutate(
    bill_depth_sc = scale(bill_depth_mm),
    bill_length_sc = scale(bill_length_mm),
    flipper_length_sc = scale(flipper_length_mm)
)
```

```{.output}
# A tibble: 344 × 11
   species island    bill_…¹ bill_…² flipp…³ body_…⁴ sex    year bill_…⁵ bill_…⁶
   <fct>   <fct>       <dbl>   <dbl>   <int>   <int> <fct> <int>   <dbl>   <dbl>
 1 Adelie  Torgersen    39.1    18.7     181    3750 male   2007   0.784  -0.883
 2 Adelie  Torgersen    39.5    17.4     186    3800 fema…  2007   0.126  -0.810
 3 Adelie  Torgersen    40.3    18       195    3250 fema…  2007   0.430  -0.663
 4 Adelie  Torgersen    NA      NA        NA      NA <NA>   2007  NA      NA    
 5 Adelie  Torgersen    36.7    19.3     193    3450 fema…  2007   1.09   -1.32 
 6 Adelie  Torgersen    39.3    20.6     190    3650 male   2007   1.75   -0.847
 7 Adelie  Torgersen    38.9    17.8     181    3625 fema…  2007   0.329  -0.920
 8 Adelie  Torgersen    39.2    19.6     195    4675 male   2007   1.24   -0.865
 9 Adelie  Torgersen    34.1    18.1     193    3475 <NA>   2007   0.480  -1.80 
10 Adelie  Torgersen    42      20.2     190    4250 <NA>   2007   1.54   -0.352
# … with 334 more rows, 1 more variable: flipper_length_sc <dbl[,1]>, and
#   abbreviated variable names ¹​bill_length_mm, ²​bill_depth_mm,
#   ³​flipper_length_mm, ⁴​body_mass_g, ⁵​bill_depth_sc[,1], ⁶​bill_length_sc[,1]
# ℹ Use `print(n = ...)` to see more rows, and `colnames()` to see all variable names
```

Its just three columns, we can do that. 
But let us imagine we have 20 of these, typing all that out is tedious and error prone. 
You might forget to alter the name or keep the same type of naming convention. 
We are only human, we easily make mistakes.
With {dplyr}'s `across()` we can combine our knowledge of tidy-selectors and mutate to create the entire transformation for these columns at once.


```r
penguins |> 
  mutate(across(.cols = ends_with("mm"), 
                .fns = scale))
```

```{.output}
# A tibble: 344 × 8
   species island    bill_length_mm[,1] bill_depth…¹ flipp…² body_…³ sex    year
   <fct>   <fct>                  <dbl>        <dbl>   <dbl>   <int> <fct> <int>
 1 Adelie  Torgersen             -0.883        0.784  -1.42     3750 male   2007
 2 Adelie  Torgersen             -0.810        0.126  -1.06     3800 fema…  2007
 3 Adelie  Torgersen             -0.663        0.430  -0.421    3250 fema…  2007
 4 Adelie  Torgersen             NA           NA      NA          NA <NA>   2007
 5 Adelie  Torgersen             -1.32         1.09   -0.563    3450 fema…  2007
 6 Adelie  Torgersen             -0.847        1.75   -0.776    3650 male   2007
 7 Adelie  Torgersen             -0.920        0.329  -1.42     3625 fema…  2007
 8 Adelie  Torgersen             -0.865        1.24   -0.421    4675 male   2007
 9 Adelie  Torgersen             -1.80         0.480  -0.563    3475 <NA>   2007
10 Adelie  Torgersen             -0.352        1.54   -0.776    4250 <NA>   2007
# … with 334 more rows, and abbreviated variable names ¹​bill_depth_mm[,1],
#   ²​flipper_length_mm[,1], ³​body_mass_g
# ℹ Use `print(n = ...)` to see more rows
```

Whoa! So fast!
Now the three columns are scaled. 
`.col` argument takes a tidy-selection of columns, and `.fns` it where you let it know which function to apply.

But oh no! The columns have been overwritten. Rather than creating new ones, we replaced the old ones.
This might be your intention in some instances, or maybe you will just create a new data set with the scaled variables. 


```r
penguins_mm_sc <- penguins |> 
  mutate(across(.cols = ends_with("mm"),
                .fns = scale))
```

but often, we'd like to keep the original but add the new variants. We can do that to within the across!


```r
penguins |> 
  mutate(across(.cols = ends_with("mm"),
                .fns = scale, 
                .names = "{.col}_sc")) |> 
  select(contains("mm"))
```

```{.output}
# A tibble: 344 × 6
   bill_length_mm bill_depth_mm flipper_length_mm bill_length_…¹ bill_…² flipp…³
            <dbl>         <dbl>             <int>          <dbl>   <dbl>   <dbl>
 1           39.1          18.7               181         -0.883   0.784  -1.42 
 2           39.5          17.4               186         -0.810   0.126  -1.06 
 3           40.3          18                 195         -0.663   0.430  -0.421
 4           NA            NA                  NA         NA      NA      NA    
 5           36.7          19.3               193         -1.32    1.09   -0.563
 6           39.3          20.6               190         -0.847   1.75   -0.776
 7           38.9          17.8               181         -0.920   0.329  -1.42 
 8           39.2          19.6               195         -0.865   1.24   -0.421
 9           34.1          18.1               193         -1.80    0.480  -0.563
10           42            20.2               190         -0.352   1.54   -0.776
# … with 334 more rows, and abbreviated variable names ¹​bill_length_mm_sc[,1],
#   ²​bill_depth_mm_sc[,1], ³​flipper_length_mm_sc[,1]
# ℹ Use `print(n = ...)` to see more rows
```

Now they are all there! neat! But that `.names` argument is a little weird. What does it really mean?

Internally, `across()` stores the column names in a vector it calls `.col`.
We can use this knowledge to tell the across function what to name our new columns. 
In this case, we want to append the column name with `_sc`. 

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 3
Transform all the colmns with an underscore in their name so they are scaled, and add the _prefix_ `sc_` to the columns names.

:::::::::::::::::::::::::::::::::::::::: solution
## Solution 


```r
penguins |> 
  mutate(across(.cols = contains("_"),
                .fns = scale, 
                .names = "sc_{.col}"))
```

```{.output}
# A tibble: 344 × 12
   species island    bill_…¹ bill_…² flipp…³ body_…⁴ sex    year sc_bi…⁵ sc_bi…⁶
   <fct>   <fct>       <dbl>   <dbl>   <int>   <int> <fct> <int>   <dbl>   <dbl>
 1 Adelie  Torgersen    39.1    18.7     181    3750 male   2007  -0.883   0.784
 2 Adelie  Torgersen    39.5    17.4     186    3800 fema…  2007  -0.810   0.126
 3 Adelie  Torgersen    40.3    18       195    3250 fema…  2007  -0.663   0.430
 4 Adelie  Torgersen    NA      NA        NA      NA <NA>   2007  NA      NA    
 5 Adelie  Torgersen    36.7    19.3     193    3450 fema…  2007  -1.32    1.09 
 6 Adelie  Torgersen    39.3    20.6     190    3650 male   2007  -0.847   1.75 
 7 Adelie  Torgersen    38.9    17.8     181    3625 fema…  2007  -0.920   0.329
 8 Adelie  Torgersen    39.2    19.6     195    4675 male   2007  -0.865   1.24 
 9 Adelie  Torgersen    34.1    18.1     193    3475 <NA>   2007  -1.80    0.480
10 Adelie  Torgersen    42      20.2     190    4250 <NA>   2007  -0.352   1.54 
# … with 334 more rows, 2 more variables: sc_flipper_length_mm <dbl[,1]>,
#   sc_body_mass_g <dbl[,1]>, and abbreviated variable names ¹​bill_length_mm,
#   ²​bill_depth_mm, ³​flipper_length_mm, ⁴​body_mass_g, ⁵​sc_bill_length_mm[,1],
#   ⁶​sc_bill_depth_mm[,1]
# ℹ Use `print(n = ...)` to see more rows, and `colnames()` to see all variable names
```

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 4
Transform all the colmns with an underscore in their name so they are scaled, and add the _prefix_ `sc_` to the columns names. 
Add another _standard_ change of the body mass column to kilograms

:::::::::::::::::::::::::::::::::::::::: hint
You can add a standard mutate within the same mutate as across
:::::::::::::::::::::::::::::::::::::::: 

:::::::::::::::::::::::::::::::::::::::: solution 
## Solution 


```r
penguins |> 
  mutate(
    across(.cols = contains("_"),
           .fns = scale, 
           .names = "sc_{.col}"),
    body_mass_kg = body_mass_g / 1000
  )
```

```{.output}
# A tibble: 344 × 13
   species island    bill_…¹ bill_…² flipp…³ body_…⁴ sex    year sc_bi…⁵ sc_bi…⁶
   <fct>   <fct>       <dbl>   <dbl>   <int>   <int> <fct> <int>   <dbl>   <dbl>
 1 Adelie  Torgersen    39.1    18.7     181    3750 male   2007  -0.883   0.784
 2 Adelie  Torgersen    39.5    17.4     186    3800 fema…  2007  -0.810   0.126
 3 Adelie  Torgersen    40.3    18       195    3250 fema…  2007  -0.663   0.430
 4 Adelie  Torgersen    NA      NA        NA      NA <NA>   2007  NA      NA    
 5 Adelie  Torgersen    36.7    19.3     193    3450 fema…  2007  -1.32    1.09 
 6 Adelie  Torgersen    39.3    20.6     190    3650 male   2007  -0.847   1.75 
 7 Adelie  Torgersen    38.9    17.8     181    3625 fema…  2007  -0.920   0.329
 8 Adelie  Torgersen    39.2    19.6     195    4675 male   2007  -0.865   1.24 
 9 Adelie  Torgersen    34.1    18.1     193    3475 <NA>   2007  -1.80    0.480
10 Adelie  Torgersen    42      20.2     190    4250 <NA>   2007  -0.352   1.54 
# … with 334 more rows, 3 more variables: sc_flipper_length_mm <dbl[,1]>,
#   sc_body_mass_g <dbl[,1]>, body_mass_kg <dbl>, and abbreviated variable
#   names ¹​bill_length_mm, ²​bill_depth_mm, ³​flipper_length_mm, ⁴​body_mass_g,
#   ⁵​sc_bill_length_mm[,1], ⁶​sc_bill_depth_mm[,1]
# ℹ Use `print(n = ...)` to see more rows, and `colnames()` to see all variable names
```

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 


## Wrap-up

We hope these sessions have given your a leg-up in starting you R and tidyverse journey. 
Remember that learning to code is like learning a new language, the best way to learn is to keep trying. 
We promise, your efforts will not be in vain as you uncover the power of R and the tidyverse.

### Learning more
As and end to this workshop, we would like to provide you with some learning materials that might aid you in further pursuits of learning R. 

The [tidyverse webpage](https://www.tidyverse.org/) offers lots of resources on learning the tidyverse way of working, and information
about what great things you can do with this collection of packages. 
There is an [R for Datascience](https://www.rfordatasci.com/) learning community that is an excellent and 
welcoming community of other learners navigating the tidyverse. We wholeheartedly recommend joining this community!
The [Rstudio community](https://community.rstudio.com/) is also a great place to ask questions or 
look for solutions for questions you may have, and so is [stackoverflow](https://stackoverflow.com/).

