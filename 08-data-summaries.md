---
title: "Data summaries with dplyr"
teaching: 60
exercises: 8
---



:::::::::::::::::::::::::::::::::::::: questions 

- How can I create summary tables of my data?
- How can I create different types of summaries based on groups in my data?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Use `summarise()` to create data summaries
- Use `group_by()` to create summaries of groups
- Use `tally()`/`count()` to create a quick frequency table

::::::::::::::::::::::::::::::::::::::::::::::::

# Motivation

Next to visualizing data, creating summaries of the data in tables is a quick way to get an idea of what type of data you have at hand. 
It might help you spot incorrect data or extreme values, or whether specific analysis approaches are needed.
To summarize data with the {tidyverse} efficiently, we need to utilize the tools we have learned the previous days, 
like adding new variables, tidy-selections, pivots and grouping data. All these tools combine amazingly when we start making summaries. 

Let us start from the beginning with summaries, and work our way up to the more complex variations as we go.

First, we must again prepare our workspace with our packages and data.


```r
library(tidyverse)
penguins <- palmerpenguins::penguins
```

We should start to feel quite familiar with our penguins by now. Let us start by finding the mean of the bill length


```r
penguins |> 
  summarise(bill_length_mean = mean(bill_length_mm))
```

```{.output}
# A tibble: 1 × 1
  bill_length_mean
             <dbl>
1               NA
```

`NA`. as we remember, there are some `NA` values in our data. 
R is very clear about trying to do calculations when there is an `NA`. 
If there is an `NA`, i.e. a value we do not know, it cannot create a correct calulcation, so it will return `NA` again.
This is a nice way of quickly seeing that you have missing values in your data.
Right now, we will ignore those.
We can omit these by adding the `na.rm = TRUE` argument, which will remove all `NA`'s before calculating the mean.


```r
penguins |> 
  summarise(bill_length_mean = mean(bill_length_mm, na.rm = TRUE))
```

```{.output}
# A tibble: 1 × 1
  bill_length_mean
             <dbl>
1             43.9
```

An alternative way to remove missing values from a column is to pass the column to {tidyr}'s `drop_na()` function. 


```r
penguins |> 
  drop_na(bill_length_mm) |> 
  summarise(bill_length_mean = mean(bill_length_mm))
```

```{.output}
# A tibble: 1 × 1
  bill_length_mean
             <dbl>
1             43.9
```



```r
penguins |> 
  drop_na(bill_length_mm) |> 
  summarise(bill_length_mean = mean(bill_length_mm),
            bill_length_min = min(bill_length_mm),
            bill_length_max = max(bill_length_mm))
```

```{.output}
# A tibble: 1 × 3
  bill_length_mean bill_length_min bill_length_max
             <dbl>           <dbl>           <dbl>
1             43.9            32.1            59.6
```

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 1
 First start by trying to summarise a single column, `body_mass_g`, by calculating its mean in *kilograms*.
 
 :::::::::::::::::::::::::::::::::::::::: solution 
## Solution


```r
penguins |> 
  drop_na(body_mass_g) |> 
  summarise(body_mass_kg_mean = mean(body_mass_g / 1000))
```

```{.output}
# A tibble: 1 × 1
  body_mass_kg_mean
              <dbl>
1              4.20
```

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 2
 Add a column with the standard deviation of `body_mass_g` on *kilogram* scale.
 
:::::::::::::::::::::::::::::::::::::::: solution 
## Solution

```r
penguins |> 
    drop_na(body_mass_g) |> 
    summarise(
        body_mass_kg_mean = mean(body_mass_g / 1000),
        body_mass_kg_sd = sd(body_mass_g / 1000)
    )
```

```{.output}
# A tibble: 1 × 2
  body_mass_kg_mean body_mass_kg_sd
              <dbl>           <dbl>
1              4.20           0.802
```

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 3
 Now add the same two metrics for `flipper_length_mm` on *centimeter* scale and 
 give the columns clear names. Why could the `drop_na()` step give us wrong results? 
 
:::::::::::::::::::::::::::::::::::::::: solution 
## Solution


```r
penguins |> 
    drop_na(body_mass_g, flipper_length_mm) |> 
    summarise(
        body_mass_kg_mean      = mean(body_mass_g / 1000),
        body_mass_kg_sd        = sd(body_mass_g / 1000),
        flipper_length_cm_mean = mean(flipper_length_mm / 10),
        flipper_length_cm_sd   = sd(flipper_length_mm / 10)
    )
```

```{.output}
# A tibble: 1 × 4
  body_mass_kg_mean body_mass_kg_sd flipper_length_cm_mean flipper_length_cm_sd
              <dbl>           <dbl>                  <dbl>                <dbl>
1              4.20           0.802                   20.1                 1.41
```
When we use drop_na on multiple columns, it will drop the _entire row_ of data where there is `NA` in 
any of the columns we specify. This means that we might be dropping valid data from body mass because
flipper length is missing, and vice versa.

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 


## Summarising grouped data

All the examples we have gone through so far with summarizing data, we have summarized the entire data set. 
But most times, we want to have a look at groups in our data, and summarize based on these groups. 
How can we manage to summarize while preserving grouping information?

We've already worked a little with the `group_by()` function, and we will use it again! 
Because, once we know how to summarize data, summarizing data by groups is as simple as adding one more line to our code.

Let us start with our first example of getting the mean of a single column.


```r
penguins |> 
  drop_na(body_mass_g) |> 
  summarise(body_mass_g_mean = mean(body_mass_g))
```

```{.output}
# A tibble: 1 × 1
  body_mass_g_mean
             <dbl>
1            4202.
```

Here, we are getting a single mean for the entire data set. 
In order to get, for instance the means of each of the species, we can group the data set by species before we summarize.


```r
penguins |> 
  drop_na(body_mass_g) |> 
  group_by(species) |> 
  summarise(body_mass_kg_mean = mean(body_mass_g / 1000))
```

```{.output}
# A tibble: 3 × 2
  species   body_mass_kg_mean
  <fct>                 <dbl>
1 Adelie                 3.70
2 Chinstrap              3.73
3 Gentoo                 5.08
```

And now we suddenly have three means! And they are tidily collected in each their row.
To this we can keep adding as we did before.


```r
penguins |> 
    drop_na(body_mass_g) |> 
    group_by(species) |>
    summarise(
        body_mass_kg_mean = mean(body_mass_g / 1000),
        body_mass_kg_min = min(body_mass_g / 1000),
        body_mass_kg_max = max(body_mass_g / 1000)
    )
```

```{.output}
# A tibble: 3 × 4
  species   body_mass_kg_mean body_mass_kg_min body_mass_kg_max
  <fct>                 <dbl>            <dbl>            <dbl>
1 Adelie                 3.70             2.85             4.78
2 Chinstrap              3.73             2.7              4.8 
3 Gentoo                 5.08             3.95             6.3 
```

Now we are suddenly able to easily compare groups within our data, since they are so neatly summarized here. 

## Simple frequency tables

So far, we have created custom summary tables with means and standard deviations etc.
But what if you want a really quick count of all the records in different groups, a frequency table.

One way, would be to use the summarise function together with the `n()` function, which counts the number of rows in each group.


```r
penguins |> 
  group_by(species) |> 
  summarise(n = n())
```

```{.output}
# A tibble: 3 × 2
  species       n
  <fct>     <int>
1 Adelie      152
2 Chinstrap    68
3 Gentoo      124
```

This is super nice, and `n()` is a nice function to remember when you are making your own custom tables.
But if all you want is the frequency table, we would suggest using the functions `count()` or `tally()`.
They are synonymous in what they do, so you can choose the one that feels more appropriate.


```r
penguins |> 
  group_by(species) |> 
  tally()
```

```{.output}
# A tibble: 3 × 2
  species       n
  <fct>     <int>
1 Adelie      152
2 Chinstrap    68
3 Gentoo      124
```

```r
penguins |> 
  group_by(species) |> 
  count()
```

```{.output}
# A tibble: 3 × 2
# Groups:   species [3]
  species       n
  <fct>     <int>
1 Adelie      152
2 Chinstrap    68
3 Gentoo      124
```

These are two really nice convenience functions for getting a quick frequency table of your data.

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 4
 Create a table that gives the mean and standard deviation of bill length, grouped by island
 
:::::::::::::::::::::::::::::::::::::::: solution 
## Solution


```r
penguins |> 
    drop_na(bill_length_mm) |> 
    group_by(island) |>
    summarise(
        bill_length_mm_mean = mean(bill_length_mm),
        bill_length_mm_sd   = sd(bill_length_mm )
    )
```

```{.output}
# A tibble: 3 × 3
  island    bill_length_mm_mean bill_length_mm_sd
  <fct>                   <dbl>             <dbl>
1 Biscoe                   45.3              4.77
2 Dream                    44.2              5.95
3 Torgersen                39.0              3.03
```

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 5
 Create a table that gives the mean and standard deviation of bill length, grouped by island and sex.
 
:::::::::::::::::::::::::::::::::::::::: solution 

## Solution


```r
penguins |> 
    drop_na(bill_length_mm) |> 
    group_by(island, sex) |>
    summarise(
        bill_length_mm_mean = mean(bill_length_mm),
        bill_length_mm_sd   = sd(bill_length_mm )
    )
```

```{.output}
`summarise()` has grouped output by 'island'. You can override using the
`.groups` argument.
```

```{.output}
# A tibble: 9 × 4
# Groups:   island [3]
  island    sex    bill_length_mm_mean bill_length_mm_sd
  <fct>     <fct>                <dbl>             <dbl>
1 Biscoe    female                43.3              4.18
2 Biscoe    male                  47.1              4.69
3 Biscoe    <NA>                  45.6              1.37
4 Dream     female                42.3              5.53
5 Dream     male                  46.1              5.77
6 Dream     <NA>                  37.5             NA   
7 Torgersen female                37.6              2.21
8 Torgersen male                  40.6              3.03
9 Torgersen <NA>                  37.9              3.23
```

::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::: 


## Ungrouping for future control

We've been grouping a lot and not ungrouping. 
Which might seem fine now, because we have not really done anything more after the summarize. 
But in many cases we might continue our merry data handling way and do lots more, and then the 
preserving of the grouping can give us some unexpected results. Let us explore that a little.


```r
penguins |> 
  group_by(species) |> 
  count()
```

```{.output}
# A tibble: 3 × 2
# Groups:   species [3]
  species       n
  <fct>     <int>
1 Adelie      152
2 Chinstrap    68
3 Gentoo      124
```

When we group by a single column and summarize, the output data is no longer grouped. 
In a way, the `summarize()` uses up one group while summarizing, as based on species, the data can not be condensed any further than this.
When we group by two columns, it actually has the same behavior. 


```r
penguins |> 
  group_by(species, island) |> 
  count()
```

```{.output}
# A tibble: 5 × 3
# Groups:   species, island [5]
  species   island        n
  <fct>     <fct>     <int>
1 Adelie    Biscoe       44
2 Adelie    Dream        56
3 Adelie    Torgersen    52
4 Chinstrap Dream        68
5 Gentoo    Biscoe      124
```

But because we used to have two groups, we now are left with one. 
In this case "species" is still a  grouping variable. 
Lets say we want a column now, that counts the total number of penguins observations. 
That would be the sum of the "n" column.


```r
penguins |> 
  group_by(species, island) |> 
  count() |> 
  mutate(total = sum(n))
```

```{.output}
# A tibble: 5 × 4
# Groups:   species, island [5]
  species   island        n total
  <fct>     <fct>     <int> <int>
1 Adelie    Biscoe       44    44
2 Adelie    Dream        56    56
3 Adelie    Torgersen    52    52
4 Chinstrap Dream        68    68
5 Gentoo    Biscoe      124   124
```

But that is not what we are expecting! why? Because the data is still grouped by species, it is now taking the sum within each species, rather than the whole. To get the whole we need first to `ungroup()`, and then try again.


```r
penguins |> 
  group_by(species, island) |> 
  count() |> 
  ungroup() |> 
  mutate(total = sum(n))
```

```{.output}
# A tibble: 5 × 4
  species   island        n total
  <fct>     <fct>     <int> <int>
1 Adelie    Biscoe       44   344
2 Adelie    Dream        56   344
3 Adelie    Torgersen    52   344
4 Chinstrap Dream        68   344
5 Gentoo    Biscoe      124   344
```

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 6
Create a table that gives the mean and standard deviation of bill length, grouped by island and sex,
then add another column that has the mean for all the data 

:::::::::::::::::::::::::::::::::::::::: solution 
## Solution


```r
penguins |> 
    drop_na(bill_length_mm) |> 
    group_by(island, sex) |>
    summarise(
        bill_length_mm_mean = mean(bill_length_mm),
        bill_length_mm_sd   = sd(bill_length_mm )
    ) |>
    ungroup() |>
    mutate(mean = mean(bill_length_mm_mean))
```

```{.output}
`summarise()` has grouped output by 'island'. You can override using the
`.groups` argument.
```

```{.output}
# A tibble: 9 × 5
  island    sex    bill_length_mm_mean bill_length_mm_sd  mean
  <fct>     <fct>                <dbl>             <dbl> <dbl>
1 Biscoe    female                43.3              4.18  42.0
2 Biscoe    male                  47.1              4.69  42.0
3 Biscoe    <NA>                  45.6              1.37  42.0
4 Dream     female                42.3              5.53  42.0
5 Dream     male                  46.1              5.77  42.0
6 Dream     <NA>                  37.5             NA     42.0
7 Torgersen female                37.6              2.21  42.0
8 Torgersen male                  40.6              3.03  42.0
9 Torgersen <NA>                  37.9              3.23  42.0
```

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 


## Grouped data manipulation

You might have noticed that we managed to do some data manipulation (i.e. `mutate`) while the data were still grouped, 
which in our example before produced unwanted results.
But, often, grouping before data manipulation can unlock great new possibilities for working with our data.

Let us use the data we made where we summarised the body mass of penguins in kilograms, and let us group by species and sex.


```r
penguins |> 
    drop_na(body_mass_g) |> 
    group_by(species, sex) |>
    summarise(
        body_mass_kg_mean = mean(body_mass_g / 1000),
        body_mass_kg_min = min(body_mass_g / 1000),
        body_mass_kg_max = max(body_mass_g / 1000)
    )
```

```{.output}
`summarise()` has grouped output by 'species'. You can override using the
`.groups` argument.
```

```{.output}
# A tibble: 8 × 5
# Groups:   species [3]
  species   sex    body_mass_kg_mean body_mass_kg_min body_mass_kg_max
  <fct>     <fct>              <dbl>            <dbl>            <dbl>
1 Adelie    female              3.37             2.85             3.9 
2 Adelie    male                4.04             3.32             4.78
3 Adelie    <NA>                3.54             2.98             4.25
4 Chinstrap female              3.53             2.7              4.15
5 Chinstrap male                3.94             3.25             4.8 
6 Gentoo    female              4.68             3.95             5.2 
7 Gentoo    male                5.48             4.75             6.3 
8 Gentoo    <NA>                4.59             4.1              4.88
```

The data we get out after that, is still grouped by species.
Let us say that we want to know, the relative size of the penguin sexes body mass to the species mean.
We would need the species mean, in addition to the species sex means.
We can add this, as the data is already grouped by sex, with a mutate.


```r
penguins |> 
    drop_na(body_mass_g) |> 
    group_by(species, sex) |>
    summarise(
        body_mass_kg_mean = mean(body_mass_g / 1000),
        body_mass_kg_min = min(body_mass_g / 1000),
        body_mass_kg_max = max(body_mass_g / 1000)
    ) |> 
    mutate(
        species_mean = mean(body_mass_kg_mean)
    )
```

```{.output}
`summarise()` has grouped output by 'species'. You can override using the
`.groups` argument.
```

```{.output}
# A tibble: 8 × 6
# Groups:   species [3]
  species   sex    body_mass_kg_mean body_mass_kg_min body_mass_kg_max species…¹
  <fct>     <fct>              <dbl>            <dbl>            <dbl>     <dbl>
1 Adelie    female              3.37             2.85             3.9       3.65
2 Adelie    male                4.04             3.32             4.78      3.65
3 Adelie    <NA>                3.54             2.98             4.25      3.65
4 Chinstrap female              3.53             2.7              4.15      3.73
5 Chinstrap male                3.94             3.25             4.8       3.73
6 Gentoo    female              4.68             3.95             5.2       4.92
7 Gentoo    male                5.48             4.75             6.3       4.92
8 Gentoo    <NA>                4.59             4.1              4.88      4.92
# … with abbreviated variable name ¹​species_mean
```

Notice that now, the same value is in the species_mean column for all the rows of each species.
This means our calculation worked!
So, in the same data set, we have everything we need to calculate the relative difference between the species mean of body mass and each of the sexes.



```r
penguins |> 
    drop_na(body_mass_g) |> 
    group_by(species, sex) |>
    summarise(
        body_mass_kg_mean = mean(body_mass_g / 1000),
        body_mass_kg_min = min(body_mass_g / 1000),
        body_mass_kg_max = max(body_mass_g / 1000)
    ) |> 
    mutate(
        species_mean = mean(body_mass_kg_mean),
        rel_species = species_mean - body_mass_kg_mean
    )
```

```{.output}
`summarise()` has grouped output by 'species'. You can override using the
`.groups` argument.
```

```{.output}
# A tibble: 8 × 7
# Groups:   species [3]
  species   sex    body_mass_kg_mean body_mass_kg_min body_mas…¹ speci…² rel_s…³
  <fct>     <fct>              <dbl>            <dbl>      <dbl>   <dbl>   <dbl>
1 Adelie    female              3.37             2.85       3.9     3.65   0.282
2 Adelie    male                4.04             3.32       4.78    3.65  -0.393
3 Adelie    <NA>                3.54             2.98       4.25    3.65   0.111
4 Chinstrap female              3.53             2.7        4.15    3.73   0.206
5 Chinstrap male                3.94             3.25       4.8     3.73  -0.206
6 Gentoo    female              4.68             3.95       5.2     4.92   0.238
7 Gentoo    male                5.48             4.75       6.3     4.92  -0.567
8 Gentoo    <NA>                4.59             4.1        4.88    4.92   0.330
# … with abbreviated variable names ¹​body_mass_kg_max, ²​species_mean,
#   ³​rel_species
```

Now we can see, with how much the male penguins usually weight compared to the female ones.

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 7
Calculate the difference in flipper length between the different species of penguin

:::::::::::::::::::::::::::::::::::::::: solution 
## Solution


```r
penguins |> 
    drop_na(flipper_length_mm) |> 
    group_by(species) |> 
    summarise(
        flipper_mean = mean(flipper_length_mm),
    ) |> 
    mutate(
        species_mean = mean(flipper_mean),
        flipper_species_diff = species_mean - flipper_mean
    )
```

```{.output}
# A tibble: 3 × 4
  species   flipper_mean species_mean flipper_species_diff
  <fct>            <dbl>        <dbl>                <dbl>
1 Adelie            190.         201.                11.0 
2 Chinstrap         196.         201.                 5.16
3 Gentoo            217.         201.               -16.2 
```

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 

::::::::::::::::::::::::::::::::::::: challenge 
## Challenge 8
Calculate the difference in flipper length between the different species of penguin, split by the penguins sex.

:::::::::::::::::::::::::::::::::::::::: solution 
## Solution


```r
penguins |> 
    drop_na(flipper_length_mm) |> 
    group_by(species, sex) |> 
    summarise(
        flipper_mean = mean(flipper_length_mm),
    ) |> 
    mutate(
        species_mean = mean(flipper_mean),
        flipper_species_diff = species_mean - flipper_mean
    )
```

```{.output}
`summarise()` has grouped output by 'species'. You can override using the
`.groups` argument.
```

```{.output}
# A tibble: 8 × 5
# Groups:   species [3]
  species   sex    flipper_mean species_mean flipper_species_diff
  <fct>     <fct>         <dbl>        <dbl>                <dbl>
1 Adelie    female         188.         189.                0.807
2 Adelie    male           192.         189.               -3.81 
3 Adelie    <NA>           186.         189.                3.00 
4 Chinstrap female         192.         196.                4.09 
5 Chinstrap male           200.         196.               -4.09 
6 Gentoo    female         213.         217.                3.96 
7 Gentoo    male           222.         217.               -4.88 
8 Gentoo    <NA>           216.         217.                0.916
```

:::::::::::::::::::::::::::::::::::::::: 
::::::::::::::::::::::::::::::::::::: 


