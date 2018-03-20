---
layout: post
title: Group Therapy: Working with groups in R
subtitle: The easy way!
tags: [rstats, r-bloggers, tutorial, professional]
---

Introduction
------------

When working with data, it is usual to perform computations on a per group basis, i.e., to make operations for every group of interest within the data. This type of operation becomes more relevant when dealing with complex datasets with multiple variables.

The strategy for this operation is known as split-apply-combine because:

1.  The dataset needs to be split first
2.  Some computation needs to be performed for every group within the dataset
3.  Everything is combined in a new dataset

Don't worry, you won't have to code hard for each of these steps. As the subtitle says, we are going to do it *The easy way!*. And by that, I mean using a well known package called `dplyr`, which is part of the **Tidyverse**.

Following the split-apply-combine strategy described before, the process using `dplyr` is:

1.  **Use the `group_by()` function with a data.frame**, which organizes the data in groups, so we can make different computations in a group-wise basis. You only give it a data.frame and a variable to group by. It will return another data.frame, with a special structure underneath. If you print it (do it now!), it will look as the original data.frame...the magic is under the hood!
2.  **Use special functions from `dplyr`, like `summarise`, `mutate()` or `filter()`, with the result from step 1 (grouped data.frame)** and every operation will be performed by group. You just perform the operations as if you were dealing with a regular data.frame (no groups), but internally the operations are performed by group, and
3.  **the result will be a new dataset**, with all the by-group results aggregated.

It may look a bit complicated but, as you will see, it is a very easy process.

I'm assuming you already know the main `dplyr` functions. But if it is not the case, I will give a very brief explanation of the `dplyr` functions used as they appear in this post.

The Pipe `%>%`
--------------

Along this blog post, I will make extensive use of the "pipe" operator (`%>%`), so I will give a very brief introduction to it here.

The "pipe" is a special operator, provided by the [magrittr](https://cran.r-project.org/package=magrittr) package, that allows us to pipe a value forward into an expression or function call. So, instead of writing `f(x)` you can write `x %>% f`.

In other words, it allows us to chain multiple expressions in an easy way, instead of having complex nested expressions. For example, this expression

``` r
table(sapply(airquality, class))
```

    ## 
    ## integer numeric 
    ##       5       1

can be more clearly written as:

``` r
airquality %>%
  sapply(class) %>%
  table
```

    ## .
    ## integer numeric 
    ##       5       1

By default the left-hand side will be piped in as the first argument of the function appearing on the right-hand side, as you can see above (e.g., airquality is piped in as the first argument of sapply, before `class`).

You can learn more of this useful operator in the documentation and vignette of the package.

The Data
--------

The datasets used for this post come from the [billboard](https://cran.r-project.org/package=billboard) package, which contains data of Billboard Hot 100 songs from 1960 to 2016.

Specifically, we are going to use 2 datasets from the package. First, we'll be working with the `wiki_hot_100s` dataset:

``` r
library(billboard)
hot100_tbl <- wiki_hot_100s %>% as_tibble()  # just transforms the dataset in a more suitable format for display.
hot100_tbl$year <- as.integer(hot100_tbl$year)
hot100_tbl
```

    ## # A tibble: 5,701 x 4
    ##       no                     title              artist  year
    ##    <chr>                     <chr>               <chr> <int>
    ##  1     1 Theme from A Summer Place         Percy Faith  1960
    ##  2     2          He'll Have to Go          Jim Reeves  1960
    ##  3     3             Cathy's Clown The Everly Brothers  1960
    ##  4     4              Running Bear      Johnny Preston  1960
    ##  5     5                Teen Angel        Mark Dinning  1960
    ##  6     6                 I'm Sorry          Brenda Lee  1960
    ##  7     7         It's Now or Never       Elvis Presley  1960
    ##  8     8                 Handy Man         Jimmy Jones  1960
    ##  9     9              Stuck on You       Elvis Presley  1960
    ## 10    10                 The Twist      Chubby Checker  1960
    ## # ... with 5,691 more rows

It is a data set containing 57 years of Billboard Hot 100 songs. It gives, for every year (1960-2016), the titles, artists and ranking.

Then, we'll move to a more interesting related dataset, `spotify_track_data`. This dataset contains the features of most of the tracks shown above (hot100\_tbl), obtained using the Spotify API:

``` r
data(spotify_track_data)  # This dataset is already in **tibble** format
spotify_track_data
```

    ## # A tibble: 5,497 x 23
    ##     year                 artist_name              artist_id explicit
    ##    <chr>                       <chr>                  <chr>    <lgl>
    ##  1  1960 Percy Faith & His Orchestra 24DQLSng7bKZD4GXLIaQbv    FALSE
    ##  2  1960                  Jim Reeves 2Ev0e8GUIX4u7d9etNLTXg    FALSE
    ##  3  1960              Johnny Preston 1B8n8vtEeexQ5NKehHTkeo    FALSE
    ##  4  1960                Mark Dinning 55Rf9Kfqd30nmqaeEDMpic    FALSE
    ##  5  1960                  Brenda Lee 4cPHsZM98sKzmV26wlwD2W    FALSE
    ##  6  1960               Elvis Presley 43ZHCT0cAZBISjO8DG9PnE    FALSE
    ##  7  1960                 Jimmy Jones 7ydcRbgt0yM9etADb1Ackp    FALSE
    ##  8  1960               Elvis Presley 43ZHCT0cAZBISjO8DG9PnE    FALSE
    ##  9  1960              Chubby Checker 7qQJQ3YtcGlqaLg5tcypN2    FALSE
    ## 10  1960              Connie Francis 3EY5DxGdy7x4GelivOjS2Q    FALSE
    ## # ... with 5,487 more rows, and 19 more variables: track_name <chr>,
    ## #   track_id <chr>, danceability <dbl>, energy <dbl>, key <int>,
    ## #   loudness <dbl>, mode <int>, speechiness <dbl>, acousticness <dbl>,
    ## #   instrumentalness <dbl>, liveness <dbl>, valence <dbl>, tempo <dbl>,
    ## #   type <chr>, uri <chr>, track_href <chr>, analysis_url <chr>,
    ## #   duration_ms <int>, time_signature <int>

Notice that in this case the range of years has changed a bit: 1960-2015

``` r
spotify_track_data$year <- as.integer(spotify_track_data$year)
range(spotify_track_data$year)
```

    ## [1] 1960 2015

More information on these datasets can be found on the package documentation.

Exploratory Analysis
--------------------

#### ...By Groups!

Now we are ready to start exploring our data. I am going make a series of questions to answer with R code.

##### 1. Who got into the Billboard Hot 100 the most?

One very handy `dplyr` function is `count()`. You just give it a data.frame and one or more variables of interest, and it will count the number of observations for every group within the variables supplied.

Let's use `count()` right now:

``` r
hot100_tbl %>%
  count(artist, sort = TRUE)  # I added 'sort = TRUE' so it is sorted in descending order
```

    ## # A tibble: 2,768 x 2
    ##             artist     n
    ##              <chr> <int>
    ##  1         Madonna    35
    ##  2      Elton John    26
    ##  3     The Beatles    26
    ##  4   Janet Jackson    24
    ##  5    Mariah Carey    24
    ##  6 Michael Jackson    22
    ##  7   Stevie Wonder    22
    ##  8         Rihanna    20
    ##  9    Taylor Swift    20
    ## 10 Whitney Houston    19
    ## # ... with 2,758 more rows

Clearly, Madonna is the artist who got into the Billboards Hot 100 the most.

Of course, if your data is already "grouped" by the variable of interest (using the `group_by` function I mentioned in the introduction) you can avoid giving `count()` that variable of interest:

``` r
hot100_tbl %>%
  group_by(artist) %>%  # As I mentioned in the introduction, this returns the same data.frame but internally organized in groups
  count(sort = TRUE)  # And now we don't give any variable to count(), because it will use the grouping variable
```

    ## # A tibble: 2,768 x 2
    ## # Groups:   artist [2,768]
    ##             artist     n
    ##              <chr> <int>
    ##  1         Madonna    35
    ##  2      Elton John    26
    ##  3     The Beatles    26
    ##  4   Janet Jackson    24
    ##  5    Mariah Carey    24
    ##  6 Michael Jackson    22
    ##  7   Stevie Wonder    22
    ##  8         Rihanna    20
    ##  9    Taylor Swift    20
    ## 10 Whitney Houston    19
    ## # ... with 2,758 more rows

##### 2. Now, How many artists got into the Billboards **n** times? Where **n** can go from 1:35

Building on the code used in the previous question:

``` r
hot100_tbl %>%
  count(artist) %>%
  count(n, sort = TRUE)  # The variable with the counts will be named "nn"
```

    ## # A tibble: 24 x 2
    ##        n    nn
    ##    <int> <int>
    ##  1     1  1818
    ##  2     2   418
    ##  3     3   176
    ##  4     4    89
    ##  5     5    75
    ##  6     6    58
    ##  7     7    31
    ##  8     8    24
    ##  9     9    15
    ## 10    11    15
    ## # ... with 14 more rows

1818 out of 2768 artists got into the Billboards just 1 time (more than 50% of artists). That means that we have 1818 one-hit wonders (technically, for the one-hit wonders it is usually considered the top 40 only). Beware we've not checked artists that have worked together (say, duos).

And of course, only one artist got into the Billboards 35 times, Madonna (check `tail()` of the last table and see).

##### 3. We could also be interested in which songs got into the Billboards Hot 100 (year-end) more than once.

Let's check that out:

``` r
hot100_tbl %>%
  count(title, sort = TRUE) %>%
  filter(n > 1)  # We keep just those titles that appear more than once
```

    ## # A tibble: 494 x 2
    ##             title     n
    ##             <chr> <int>
    ##  1          Angel     6
    ##  2 Because of You     5
    ##  3        Forever     5
    ##  4         Heaven     5
    ##  5        My Love     5
    ##  6    Without You     5
    ##  7         Always     4
    ##  8          Crazy     4
    ##  9           Hero     4
    ## 10      I Like It     4
    ## # ... with 484 more rows

After `count()` we used `filter()` to filter out those titles that appear more than once. This`dplyr` function just takes a data.frame and a condition and it returns the rows where the condition is true.

Apparently, many titles appeared more than once (494). Let's dig into it using a very useful dplyr function that is the motivation of this post: `group_by` (again, to stick it in your memory, this function organizes the data in groups so we can make different computations in a group-wise basis).

Let's check the range of years in which the songs appeared:

``` r
hot100_tbl %>%
  group_by(title) %>%  # grouping the dataset. The same as: group_by(hot100_tbl, title)
  summarise(n = n(), min_year = min(year), max_year = max(year)) %>%  # the n() function just counts the number of observations for each group (the same we did before with count()
  arrange(desc(n))
```

    ## # A tibble: 5,109 x 4
    ##             title     n min_year max_year
    ##             <chr> <int>    <dbl>    <dbl>
    ##  1          Angel     6     1985     2003
    ##  2 Because of You     5     1998     2007
    ##  3        Forever     5     1960     2010
    ##  4         Heaven     5     1985     2004
    ##  5        My Love     5     1966     2007
    ##  6    Without You     5     1961     2012
    ##  7         Always     4     1987     1995
    ##  8          Crazy     4     1991     2006
    ##  9           Hero     4     1994     2002
    ## 10      I Like It     4     1989     2010
    ## # ... with 5,099 more rows

In the first 10 rows we can see that the titles that appear more than once (see `n`) come from a wide range of years

The `summarise()` function, when used in conjunction with a grouped data.frame, computes the specified summary statistics (e.g., `min(year)`) for every group within the data. The `arrange()` function just arranges the rows by the the variables specified (in this case, by `n` in `desc`ending order).

Let's add a couple of variables to understand why these titles appear in a wide range of years:

``` r
hot100_tbl %>%
  group_by(title) %>%
  summarise(n = n(), min_year = min(year), max_year = max(year), n_artists = n_distinct(artist), min_artist = min(artist), max_artist = max(artist)) %>%
  arrange(desc(n))
```

    ## # A tibble: 5,109 x 7
    ##             title     n min_year max_year n_artists                         min_artist              max_artist
    ##             <chr> <int>    <dbl>    <dbl>     <int>                              <chr>                   <chr>
    ##  1          Angel     6     1985     2003         6                          Aerosmith Shaggy featuring Rayvon
    ##  2 Because of You     5     1998     2007         3                         98 Degrees                   Ne-Yo
    ##  3        Forever     5     1960     2010         4                        Chris Brown      The Little Dippers
    ##  4         Heaven     5     1985     2004         5                        Bryan Adams                 Warrant
    ##  5        My Love     5     1966     2007         4   Justin Timberlake featuring T.I.            Petula Clark
    ##  6    Without You     5     1961     2012         4       David Guetta featuring Usher             Mötley Crüe
    ##  7         Always     4     1987     1995         3                     Atlantic Starr                 Erasure
    ##  8          Crazy     4     1991     2006         4                          Aerosmith                    Seal
    ##  9           Hero     4     1994     2002         3 Chad Kroeger featuring Josey Scott            Mariah Carey
    ## 10      I Like It     4     1989     2010         4                               Dino  The Blackout All-Stars
    ## # ... with 5,099 more rows

It's clear that most of the titles that appeared more than once are titles from different artists in different years, but with the same name. Exploring a bit more, you will find that just a small number of titles from the same artist got into the list more than once (e.g., "Because of You" from 98 Degrees appeared in the list in 1998 and 1999. The same title, "Because of You", but from **Kelly Clarkson** appeared in the list in 2005 and 2006).

So, If you would like to know which **songs** appeared in the Billboards Hot 100 more than once you'll have to work with both the title and artist. And this is the type of cases where `group_by()` comes in handy.

``` r
hot100_tbl %>%
  group_by(title, artist) %>%  # Every unique combination of title-artist will be considered a group
  summarise(n = n(), min_year = min(year), max_year = max(year)) %>%
  filter(n > 1) %>%
  arrange(desc(n))
```

    ## # A tibble: 223 x 5
    ## # Groups:   title [222]
    ##                      title                        artist     n min_year max_year
    ##                      <chr>                         <chr> <int>    <dbl>    <dbl>
    ##  1          100% Pure Love                Crystal Waters     2     1994     1995
    ##  2                       3                Britney Spears     2     2009     2010
    ##  3 4 Seasons of Loneliness                   Boyz II Men     2     1997     1998
    ##  4                     679 Fetty Wap featuring Remy Boyz     2     2015     2016
    ##  5                   Adorn                        Miguel     2     2012     2013
    ##  6                   Again                 Janet Jackson     2     1993     1994
    ##  7     All About That Bass                Meghan Trainor     2     2014     2015
    ##  8           All Cried Out          Allure featuring 112     2     1997     1998
    ##  9             All for You                  Sister Hazel     2     1997     1998
    ## 10          All I Wanna Do                   Sheryl Crow     2     1994     1995
    ## # ... with 213 more rows

And now whe have only 223 songs that appeared more than once in the Billboards Hot 100 (remember that initially we had 494). And, with just 6 exceptions, these songs appeared in consecutive years (try getting those 6 exceptions!).

Let's move on to the more interesting related dataset, `spotify_track_data`, which contains the features of most of the tracks in hot100\_tbl (see [The Data](#the-data)).

##### 4. Did the songs use to last longer than now? What about the loudness?

``` r
year_duration_loud <- spotify_track_data %>%
  group_by(year) %>%
  summarise(duration_avg = mean(duration_ms),  # In milliseconds
            loudness_avg = mean(loudness))  # In dB. The closer to 0, the louder

year_duration_loud
```

    ## # A tibble: 56 x 3
    ##     year duration_avg loudness_avg
    ##    <int>        <dbl>        <dbl>
    ##  1  1960     155654.1   -11.127660
    ##  2  1961     156342.8   -10.835400
    ##  3  1962     160043.7   -10.561870
    ##  4  1963     155572.9   -10.334300
    ##  5  1964     155645.1    -9.612210
    ##  6  1965     160676.3    -9.191265
    ##  7  1966     166129.5    -9.849979
    ##  8  1967     172115.5    -9.941828
    ##  9  1968     192539.9   -10.888580
    ## 10  1969     194415.0   -10.601936
    ## # ... with 46 more rows

Let's graph these results.

``` r
p_year_duration <- ggplot(year_duration_loud, aes(year, duration_avg/1000)) + geom_line() +
  scale_x_continuous(breaks = seq(1960, 2010, by = 10)) + ylab("Avg. Duration (seconds)")
p_year_loud <- ggplot(year_duration_loud, aes(year, loudness_avg)) + geom_line() +
  scale_x_continuous(breaks = seq(1960, 2010, by = 10)) + ylab("Avg. Loudness (dB)")

grid.arrange(p_year_duration, p_year_loud, ncol = 2)
```

![](/img/20180320/unnamed-chunk-10-1.png)

Clearly, new songs tend to last longer and be louder.

I tried using medians instead of means, and the result is almost the same. Of course, you can use any type of graph you want. For example, we could have used a boxplot but, in this case, the evolution of duration and loudness over time is clearer with a line graph than with a boxplot (try it!).

##### 5. Get z-scores for the `duration` variable, for every year.

Sometimes it can be useful to get z-scores for each group in our data frame. With `group_by()`, it is very easy to get z-scores by group. I will do it in 2 steps so It is easier to understand:

**1st. step:** get distances from the mean for the `duration_ms` variable for each group (year).

``` r
spotify_track_data %>%
  group_by(year) %>%
  mutate(duration_dist_mean = duration_ms - mean(duration_ms)) %>%
  select(year, artist_name, track_name, duration_ms, duration_dist_mean)  # line added only to make visible the last operation (mutation)
```

    ## # A tibble: 5,497 x 5
    ## # Groups:   year [56]
    ##     year                 artist_name                                           track_name duration_ms duration_dist_mean
    ##    <int>                       <chr>                                                <chr>       <int>              <dbl>
    ##  1  1960 Percy Faith & His Orchestra "The Theme From \"A Summer Place\" - Single Version"      144893           -10761.1
    ##  2  1960                  Jim Reeves                                     He'll Have to Go      138640           -17014.1
    ##  3  1960              Johnny Preston                                         Running Bear      160027             4372.9
    ##  4  1960                Mark Dinning                                           Teen Angel      157080             1425.9
    ##  5  1960                  Brenda Lee                           I'm Sorry - Single Version      160067             4412.9
    ##  6  1960               Elvis Presley               It's Now Or Never - 2003 Sony Remaster      195733            40078.9
    ##  7  1960                 Jimmy Jones                                            Handy Man      120973           -34681.1
    ##  8  1960               Elvis Presley                                         Stuck on You      139293           -16361.1
    ##  9  1960              Chubby Checker                             The Twist - Re-Recording      158560             2905.9
    ## 10  1960              Connie Francis                          Everybody's Somebody's Fool      160093             4438.9
    ## # ... with 5,487 more rows

And now we have, for every year, the distances from the mean for the `duration_ms` variable.

Here we used another `dplyr` verb (function), `mutate()`.This function allows us to add new variables to our data.frame, often as a function of existing variables. When used in conjunction with a "grouped" data.frame, it makes every computation per group (in this case, for every year).

The `select()` function (also from `dplyr`) lets us select variables (columns) in our data.frame in a very easy way (as you can see in the code).

**2nd. step:** get the z-scores for the `duration_ms` variable for each group.

``` r
spotify_track_data %>%
  group_by(year) %>%
  mutate(duration_dist_mean = duration_ms - mean(duration_ms),  # the same we did in the 1st. step
         duration_z_score = duration_dist_mean / sd(duration_ms)) %>%  # we use the just computed variable, and divide it by the standard deviation
  select(year, artist_name, track_name, duration_ms, duration_dist_mean, duration_z_score)
```

    ## # A tibble: 5,497 x 6
    ## # Groups:   year [56]
    ##     year                 artist_name                                           track_name duration_ms duration_dist_mean duration_z_score
    ##    <int>                       <chr>                                                <chr>       <int>              <dbl>            <dbl>
    ##  1  1960 Percy Faith & His Orchestra "The Theme From \"A Summer Place\" - Single Version"      144893           -10761.1      -0.42008025
    ##  2  1960                  Jim Reeves                                     He'll Have to Go      138640           -17014.1      -0.66417814
    ##  3  1960              Johnny Preston                                         Running Bear      160027             4372.9       0.17070457
    ##  4  1960                Mark Dinning                                           Teen Angel      157080             1425.9       0.05566275
    ##  5  1960                  Brenda Lee                           I'm Sorry - Single Version      160067             4412.9       0.17226604
    ##  6  1960               Elvis Presley               It's Now Or Never - 2003 Sony Remaster      195733            40078.9       1.56455699
    ##  7  1960                 Jimmy Jones                                            Handy Man      120973           -34681.1      -1.35384348
    ##  8  1960               Elvis Presley                                         Stuck on You      139293           -16361.1      -0.63868703
    ##  9  1960              Chubby Checker                             The Twist - Re-Recording      158560             2905.9       0.11343740
    ## 10  1960              Connie Francis                          Everybody's Somebody's Fool      160093             4438.9       0.17328100
    ## # ... with 5,487 more rows

That's it! That's all we need to do to get the z-scores. And, of course, you can do it all in just one step, using the general formula you know for z-scores:

``` r
duration_extreme_tracks <- spotify_track_data %>%
  group_by(year) %>%
  mutate(duration_z_score = (duration_ms - mean(duration_ms)) / sd(duration_ms)) %>%
  select(year, artist_name, track_name, duration_ms, duration_z_score) %>%  # the same as before, in just one step
  filter(duration_z_score > 5)  # filter the extreme tracks (for every year)

duration_extreme_tracks
```

    ## # A tibble: 12 x 5
    ## # Groups:   year [12]
    ##     year       artist_name                                                      track_name duration_ms duration_z_score
    ##    <int>             <chr>                                                           <chr>       <int>            <dbl>
    ##  1  1961    The Four Preps                              More Money For You And Me (Medley)      386467         6.107162
    ##  2  1962       Jimmy Smith                                           Walk On The Wild Side      356867         6.182167
    ##  3  1963     Stevie Wonder Fingertips Pts. 1 & 2 - Live At The Regal Theater, Chicago/1962      413467         7.665375
    ##  4  1964         Stan Getz                                           The Girl From Ipanema      317987         5.227879
    ##  5  1967         The Doors              Light My Fire - New Stereo Mix Advanced Resolution      418387         6.653281
    ##  6  1970           Melanie                                  Lay Down (Candles In The Rain)      459960         5.107751
    ##  7  1973   The Temptations                                                     Masterpiece      817733         6.426979
    ##  8  1974     Mike Oldfield                                           Tubular Bells - Pt. I     1561133         9.307609
    ##  9  1977      The Floaters                                                        Float On      708867         6.410564
    ## 10  1984         Sheila E.                                              The Glamorous Life      543560         5.471275
    ## 11  1987    George Michael                         I Want Your Sex - Pts. 1 & 2 Remastered      557293         6.294869
    ## 12  1993 Tony! Toni! Toné!                                                     Anniversary      563840         5.584988

Now, using the z-scores, we've filtered out the extreme tracks based on duration (those with z-score greater than 5). Notice that there are no recent tracks - the most recent track is from 1993. However, according to a plot I showed above, recent tracks tend to last longer. What is going on? This is because our z-scores were computed for each year so, for recent years the z-scores are never greater than 5, i.e., for recent years (1994 and ahead) there is no track that lasts extremely longer than the rest. And with a boxplot we can see it clearly:

``` r
ggplot(spotify_track_data, aes(year, duration_ms, group = year)) +
  geom_boxplot() +
  geom_text(data = filter(spotify_track_data, row_number(desc(duration_ms)) < 3),
            aes(year, duration_ms, label = paste0(track_name, "\n", "artist: ", artist_name)),
            col = 2, size = 2.8,
            vjust = 0, hjust = 0, check_overlap = TRUE) +
  ylim(0, 1620000) +
  scale_x_continuous(name = NULL, breaks = seq(1960, 2010, by = 10))
```

![](/img/20180320/unnamed-chunk-14-1.png)

##### 6. Do more energetic songs tend to be louder?

Let's start answering this question graphically.

``` r
ggplot(spotify_track_data, aes(energy, loudness)) +
  geom_point() +
  geom_smooth(method = "lm")
```

![](/img/20180320/unnamed-chunk-15-1.png)

Let's split this scatterplot by year:

``` r
ggplot(spotify_track_data, aes(energy, loudness)) +
  geom_point() +
  geom_smooth(method = "lm") +
  facet_wrap(~year, nrow = 8)
```

![](/img/20180320/scatterlpot_mat-1.png)

We can see a positive relationship between `energy` and `loudness`. Something else that can be noticed easily in the last plot is that the minimum loudness tend to increase. In fact, in the last 8 years the minimun loudness is around -10 (compare it with the first years in the plot).

Let's check out another aproach for the same question, this time using 2 variables to group by.

``` r
spotify_track_data$energy_cat <- factor(cut2(spotify_track_data$energy, g = 3),
                                        labels = c("low_energy", "medium_energy", "high_energy"))

year_energy_loud <- spotify_track_data %>%
  group_by(year, energy_cat) %>%  # Notice how easy it is to group our data by 2+ variables
  summarise(energy_avg = mean(energy),
            loudness_avg = mean(loudness),
            acousticness_avg = mean(acousticness))

year_energy_loud
```

    ## # A tibble: 168 x 5
    ## # Groups:   year [?]
    ##     year    energy_cat energy_avg loudness_avg acousticness_avg
    ##    <int>        <fctr>      <dbl>        <dbl>            <dbl>
    ##  1  1960    low_energy  0.3521547   -12.149453        0.7196406
    ##  2  1960 medium_energy  0.6220313    -9.363344        0.5225563
    ##  3  1960   high_energy  0.8242500    -8.893500        0.6342500
    ##  4  1961    low_energy  0.3469313   -12.388156        0.7126016
    ##  5  1961 medium_energy  0.6263750    -8.576833        0.5037200
    ##  6  1961   high_energy  0.8358333    -7.071167        0.5147691
    ##  7  1962    low_energy  0.3462619   -12.168889        0.6961587
    ##  8  1962 medium_energy  0.6306250    -8.621833        0.4750125
    ##  9  1962   high_energy  0.8465385    -6.355615        0.3635390
    ## 10  1963    low_energy  0.3476909   -12.573400        0.6355600
    ## # ... with 158 more rows

Now, let's graph it:

``` r
ggplot(year_energy_loud, aes(year, loudness_avg, fill = energy_cat)) +
  geom_bar(stat = "identity", position = "dodge") +
  geom_line(aes(colour = energy_cat), size = 1)
```

![](/img/20180320/unnamed-chunk-17-1.png)

The answer to the original question is an absolute yes. More energetic songs tend to be louder for every year (what was expected).

##### 7. Build a linear model for every year, relating energy with loudness. Then, get the slopes from all those models.

Finally, a function that lets you **do** anything within each group of data, not only summary statistics: `dplyr::do()`

``` r
yearly_mods <- spotify_track_data %>%
  group_by(year) %>%
  do(model = lm(energy ~ loudness, data = .))

yearly_mods
```

    ## Source: local data frame [56 x 2]
    ## Groups: <by row>
    ## 
    ## # A tibble: 56 x 2
    ##     year    model
    ##  * <int>   <list>
    ##  1  1960 <S3: lm>
    ##  2  1961 <S3: lm>
    ##  3  1962 <S3: lm>
    ##  4  1963 <S3: lm>
    ##  5  1964 <S3: lm>
    ##  6  1965 <S3: lm>
    ##  7  1966 <S3: lm>
    ##  8  1967 <S3: lm>
    ##  9  1968 <S3: lm>
    ## 10  1969 <S3: lm>
    ## # ... with 46 more rows

And now we have a linear model for every year. That easy! You can get a particular one in the same way as you would extract a value from a regular data.frame:

``` r
yearly_mods$model[[1]]  # You can use any valid method for extracting a specific value from a data.frame.
```

    ## 
    ## Call:
    ## lm(formula = energy ~ loudness, data = .)
    ## 
    ## Coefficients:
    ## (Intercept)     loudness  
    ##     0.84672      0.03499

You can get many details for every model, but in this example we only want the slopes. Let's do it:

``` r
yearly_mods %>%
  do(data.frame(year = .$year,
                b0 = coef(.$model)[1],
                b1 = coef(.$model)[2]))
```

    ## Source: local data frame [56 x 3]
    ## Groups: <by row>
    ## 
    ## # A tibble: 56 x 3
    ##     year        b0         b1
    ##  * <int>     <dbl>      <dbl>
    ##  1  1960 0.8467184 0.03498663
    ##  2  1961 0.8379673 0.03371369
    ##  3  1962 0.8588901 0.03591647
    ##  4  1963 1.0331075 0.04946320
    ##  5  1964 0.9168048 0.03692364
    ##  6  1965 0.9206694 0.03655045
    ##  7  1966 0.9433630 0.03776470
    ##  8  1967 0.9013945 0.03656939
    ##  9  1968 0.9551343 0.03872170
    ## 10  1969 0.8974978 0.03673936
    ## # ... with 46 more rows

Here another way to get the same:

``` r
yearly_mods %>%
  summarise(year = year,
            b0 = coef(model)[1],
            b1 = coef(model)[2])
```

    ## # A tibble: 56 x 3
    ##     year        b0         b1
    ##    <int>     <dbl>      <dbl>
    ##  1  1960 0.8467184 0.03498663
    ##  2  1961 0.8379673 0.03371369
    ##  3  1962 0.8588901 0.03591647
    ##  4  1963 1.0331075 0.04946320
    ##  5  1964 0.9168048 0.03692364
    ##  6  1965 0.9206694 0.03655045
    ##  7  1966 0.9433630 0.03776470
    ##  8  1967 0.9013945 0.03656939
    ##  9  1968 0.9551343 0.03872170
    ## 10  1969 0.8974978 0.03673936
    ## # ... with 46 more rows

The column **b1** gives us the slopes. You can get a summary of that, or graph it. The result is that, as we saw in the previous question, the slope is positive for every year (there is a positive relationship between energy and loundess, every year!).

Conclusions
-----------

Working with groups of observations in our datasets can be very easy. You just need to turn your dataset into a grouped object using `group_by()`, and all subsequent computations (using simple `dplyr` functions) will be performed by group.

Most useful `dplyr` functions to use in conjunction with `group_by()` are:

-   `summarise()`
-   `mutate()`
-   `filter()`

I invite you to explore these datasets a bit more in order to practice grouped operations. There is a lot more that can be explored/analyzed. Maybe "grouped operations" can be a concept a bit hard to grasp at first but with time it will become second nature.
