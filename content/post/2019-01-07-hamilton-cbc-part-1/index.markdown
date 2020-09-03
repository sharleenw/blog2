---
title: 'Hamilton Christmas Bird Count: Part 1'
author: Sharleen
date: '2019-01-07'
summary: Importing and cleaning of 100 years of the Hamilton Christmas Bird Count data.
tags: ["data cleaning", "birding"]
draft: false
coverImage: https://www.publicdomainpictures.net/pictures/250000/velka/black-capped-chickadee-on-branch-3.jpg
thumbnailImage: https://www.publicdomainpictures.net/pictures/250000/velka/black-capped-chickadee-on-branch-3.jpg
thumbnailImagePosition: left
---



{{< alert info >}}
Note: This is the first part of four for this dataset.
{{< /alert >}}

- [Part 2a](https://sharleenw.rbind.io/post/hamilton_cbc_part_2a/hamilton-christmas-bird-count-part-2a/) contains data downloading and cleaning

- [Part 2b](https://sharleenw.rbind.io/post/hamilton_cbc_part_2b/hamilton-christmas-bird-count-part-2b/) contains visualizations

- [Part 3](https://sharleenw.rbind.io/post/hamilton_cbc_part_3/hamilton-christmas-bird-count-part-3/) contains my Shiny app!

- [Part 4](https://sharleenw.rbind.io/post/hamilton_cbc_part_4/hamilton-christmas-bird-count-part-4/) contains a `gganimate`d plot

# Introduction

About two years ago, I was taking my dog for a walk through a park and I began to notice the birds and how fascinating they were! 🐦 I began regularly going out birding (aka "bird-watching") and reading up on these cool little flying dinosaurs.

It turns out there's a lot of data in the birding world as well. Birding attracts the sort of detail-oriented person who likes to count and record stuff.

So there are opportunities to get involved in citizen science projects, including a long-running project called the Christmas Bird Count (CBC). It started in 1900, when Frank Chapman, an ornithologist, came up with the idea of counting birds as an alternative to hunting them at Christmas (hunting them being the previous tradition).[^1]

Birders have been going out every year around Christmas, to spend the day walking, biking, or driving through a census area to count all the birds they see or hear.

For the past two years, I have gone out with Hamilton's Christmas Bird Count. I learn a lot while I'm out there and it feels like we are contributing to a larger purpose because of the data we are collecting.

So I thought I would look at the data and see what it could tell me!

Specifically, I've noticed birders will say things like, "the House Sparrows are getting worse every year" or, "the number of Bald Eagles has increased", and I was wondering if the Christmas Bird Count data would agree or disagree with those statements.

To access the data, I went on the [Bird Studies Canada](https://www.birdscanada.org/index.jsp) website, clicked on Citizen Science, then Christmas Bird Count, then CBC Audubon Database, and then Historical Results by Count. I downloaded all years of data that existed for the Hamilton count.

If you would like to directly access the csv file that I used from my Github page, [here](https://raw.githubusercontent.com/sharleenw/my_blog/master/content/post/hamilton_cbc_part_1/hamilton-cbc-all-years-csv.csv) it is!

# Data import

I started by loading all of the packages I will be using:


```r
library(dplyr)
library(janitor)
library(readr)
library(naniar)
library(lubridate)
library(stringr)
library(tidyr)
library(here)
```

I read in the data using the [`readr`](https://readr.tidyverse.org/) and [`here`](https://malco.io/2018/11/05/why-should-i-use-the-here-package/) packages.


```r
hamilton_cbc <- read_csv(here::here(
  "content",
  "post",
  "hamilton_cbc_part_1",
  "hamilton-cbc-all-years-csv.csv"))
```

# Data cleaning

As shown below, it turns out that the first row just gives information about the count name and latitude/longitude, so I extracted those two pieces of information as `current_circle_name` and `lat_long` and then [`slice`](https://dplyr.tidyverse.org/reference/slice.html)d the file so that the first two lines were excluded from the dataset. I then used `clean_names` from the [`janitor`](https://www.rdocumentation.org/packages/janitor/versions/1.1.1/topics/clean_names) package.


```r
hamilton_cbc %>%
  head()
```

```
# A tibble: 6 x 9
  CircleName Abbrev    LatLong        X4      X5      X6    X7       X8    X9   
  <chr>      <chr>     <chr>          <chr>   <chr>   <chr> <chr>    <chr> <chr>
1 Hamilton   ONHA      43.2678790000~ <NA>    <NA>    <NA>   <NA>    <NA>  <NA> 
2 <NA>       <NA>      <NA>           <NA>    <NA>    <NA>   <NA>    <NA>  <NA> 
3 CountYear3 LowTemp   HighTemp       AMCloud PMClou~ AMRa~ "PMRain" AMSn~ PMSn~
4 118        -18.0 Ce~ -13.0 Celsius  Clear   Clear   None  "None"   None  None 
5 117        -2.0 Cel~ 11.0 Celsius   Cloudy  Cloudy  Light "Heavy\~ None  None 
6 116        -2.0 Cel~ 5.0 Celsius    Partly~ Partly~ None  "Light"  None  None 
```

```r
current_circle_name <- hamilton_cbc[1, 1]
lat_long <- hamilton_cbc[1, 3]

hamilton_cbc <- hamilton_cbc %>%
  slice(3 : n())

hamilton_cbc <- hamilton_cbc %>%
  clean_names()
```

Since I played around with the data before writing this, I know that there are actually six tables in this dataset.

The first three tables contain count day weather data. A lot of the weather data is missing and inconsistent. I will remove these three tables from `hamilton_cbc`.

Here is the end of the first table and the start of the second table. Notice the line of `NA`s between the two tables:


```r
hamilton_cbc %>%
  slice(47:54)
```

```
# A tibble: 8 x 9
  circle_name abbrev    lat_long  x4       x5       x6      x7     x8     x9    
  <chr>       <chr>     <chr>     <chr>    <chr>    <chr>   <chr>  <chr>  <chr> 
1 66          10        25        Clear    Clear    Unknown Unkno~ Unkno~ Unkno~
2 65          18        31        Cloudy   Cloudy   Unknown Unkno~ Unkno~ Unkno~
3 64          26        34        Cloudy   Cloudy   Unknown Unkno~ Unkno~ Unkno~
4 63          21        36        Cloudy   Cloudy   Unknown Unkno~ Unkno~ Unkno~
5 <NA>        <NA>      <NA>      <NA>     <NA>     <NA>    <NA>   <NA>   <NA>  
6 CountYear5  LowTemp3  HighTemp2 AMCloud2 PMCloud~ <NA>    <NA>   <NA>   <NA>  
7 118         12/26/20~ 85        198.75   100      <NA>    <NA>   <NA>   <NA>  
8 117         12/26/20~ 95        216.65   97       <NA>    <NA>   <NA>   <NA>  
```

Here is the end of the second table and the start of the third table. Notice the line of `NA`s between the two tables:


```r
hamilton_cbc %>% 
  slice(143:150)
```

```
# A tibble: 8 x 9
  circle_name abbrev                lat_long x4    x5    x6    x7    x8    x9   
  <chr>       <chr>                 <chr>    <chr> <chr> <chr> <chr> <chr> <chr>
1 26          12/26/1925            10       <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
2 25          12/27/1924            8        <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
3 23          12/26/1922            9        <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
4 22          12/23/1921            2        8     <NA>  <NA>  <NA>  <NA>  <NA> 
5 <NA>        <NA>                  <NA>     <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
6 CountYear4  LowTemp2              <NA>     <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
7 118         Hamilton Naturalists~ <NA>     <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
8 117         Hamilton Naturalists~ <NA>     <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
```

Here is the end of the third table and the start of the fourth table. Notice that there is a line of `NA`s between the two tables. The fourth table is where the bird count data actually starts!


```r
hamilton_cbc %>%
  slice(239:246)
```

```
# A tibble: 8 x 9
  circle_name     abbrev           lat_long x4     x5    x6    x7    x8    x9   
  <chr>           <chr>            <chr>    <chr>  <chr> <chr> <chr> <chr> <chr>
1 "26"             <NA>            <NA>     <NA>   <NA>  <NA>  <NA>  <NA>  <NA> 
2 "25"             <NA>            <NA>     <NA>   <NA>  <NA>  <NA>  <NA>  <NA> 
3 "23"             <NA>            <NA>     <NA>   <NA>  <NA>  <NA>  <NA>  <NA> 
4 "22"             <NA>            <NA>     <NA>   <NA>  <NA>  <NA>  <NA>  <NA> 
5  <NA>            <NA>            <NA>     <NA>   <NA>  <NA>  <NA>  <NA>  <NA> 
6 "COM_NAME"      "CountYear"      how_man~ Numbe~ Flags <NA>  <NA>  <NA>  <NA> 
7 "Snow Goose\r\~ "1921 [22]\r\nC~ <NA>     <NA>   <NA>  <NA>  <NA>  <NA>  <NA> 
8 "Snow Goose\r\~ "1922 [23]\r\nC~ <NA>     <NA>   <NA>  <NA>  <NA>  <NA>  <NA> 
```

The last two tables of the six tables contain the names of the people who went out counting each year. I will also remove these two tables.

Since the tables are separated by having a line of `NA`'s in between each table, I will first figure out which rows are a line of NAs. Then I will only keep the rows of the fourth table.


```r
blank_lines <- hamilton_cbc %>%
  mutate(row_num = row_number()) %>%
  filter(is.na(circle_name))

blank_lines
```

```
# A tibble: 5 x 10
  circle_name abbrev lat_long x4    x5    x6    x7    x8    x9    row_num
  <chr>       <chr>  <chr>    <chr> <chr> <chr> <chr> <chr> <chr>   <int>
1 <NA>        <NA>   <NA>     <NA>  <NA>  <NA>  <NA>  <NA>  <NA>       51
2 <NA>        <NA>   <NA>     <NA>  <NA>  <NA>  <NA>  <NA>  <NA>      147
3 <NA>        <NA>   <NA>     <NA>  <NA>  <NA>  <NA>  <NA>  <NA>      243
4 <NA>        <NA>   <NA>     <NA>  <NA>  <NA>  <NA>  <NA>  <NA>    23463
5 <NA>        <NA>   <NA>     <NA>  <NA>  <NA>  <NA>  <NA>  <NA>    23473
```

```r
starting_line <- blank_lines %>%
  filter(row_number() == 3) %>%
  pull(row_num)

ending_line <- blank_lines %>%
  filter(row_number() == 4) %>%
  pull(row_num)
```

So, with those values of `starting_line` and `ending_line`, we can `slice` our dataset to only have the rows between those two values. Here's what it looks like:


```r
hamilton_cbc <- hamilton_cbc %>%
  # Only keep the rows within the fourth table
  slice((starting_line + 1):(ending_line - 1))

hamilton_cbc %>%
  head(n = 3)
```

```
# A tibble: 3 x 9
  circle_name     abbrev           lat_long x4     x5    x6    x7    x8    x9   
  <chr>           <chr>            <chr>    <chr>  <chr> <chr> <chr> <chr> <chr>
1 "COM_NAME"      "CountYear"      how_man~ Numbe~ Flags <NA>  <NA>  <NA>  <NA> 
2 "Snow Goose\r\~ "1921 [22]\r\nC~ <NA>     <NA>   <NA>  <NA>  <NA>  <NA>  <NA> 
3 "Snow Goose\r\~ "1922 [23]\r\nC~ <NA>     <NA>   <NA>  <NA>  <NA>  <NA>  <NA> 
```

```r
hamilton_cbc %>%
  tail(n = 3)
```

```
# A tibble: 3 x 9
  circle_name      abbrev           lat_long x4    x5    x6    x7    x8    x9   
  <chr>            <chr>            <chr>    <chr> <chr> <chr> <chr> <chr> <chr>
1 "House Sparrow\~ "2015 [116]\r\n~ 2326     10.5~ <NA>  <NA>  <NA>  <NA>  <NA> 
2 "House Sparrow\~ "2016 [117]\r\n~ 2565     11.8~ <NA>  <NA>  <NA>  <NA>  <NA> 
3 "House Sparrow\~ "2017 [118]\r\n~ 2731     13.7~ <NA>  <NA>  <NA>  <NA>  <NA> 
```

You can see that the table starts with Snow Goose data from 1921 and goes until House Sparrow data in 2017.

Now we can clean this dataset up a bit more using the `janitor` package ❤️! This package will remove any empty columns, convert the top row to the column names of the dataset and it will clean the names.


```r
# Janitor package to the rescue!
hamilton_cbc <- hamilton_cbc %>%
  janitor::remove_empty(which = "cols") %>%  
  janitor::row_to_names(row_number = 1) %>%
  janitor::clean_names() %>%
  rename(species = com_name)

hamilton_cbc %>%
  head()
```

```
# A tibble: 6 x 5
  species        count_year                  how_many_cw number_by_party_~ flags
  <chr>          <chr>                       <chr>       <chr>             <chr>
1 "Snow Goose\r~ "1921 [22]\r\nCount Date: ~ <NA>        <NA>              <NA> 
2 "Snow Goose\r~ "1922 [23]\r\nCount Date: ~ <NA>        <NA>              <NA> 
3 "Snow Goose\r~ "1924 [25]\r\nCount Date: ~ <NA>        <NA>              <NA> 
4 "Snow Goose\r~ "1925 [26]\r\nCount Date: ~ <NA>        <NA>              <NA> 
5 "Snow Goose\r~ "1926 [27]\r\nCount Date: ~ <NA>        <NA>              <NA> 
6 "Snow Goose\r~ "1928 [29]\r\nCount Date: ~ <NA>        <NA>              <NA> 
```

- `species` gives the species name in English and the scientific name, in parentheses
- `count_year` data has a lot of information that we will parse out in a moment
- `how_many_cw` provides the actual bird count
- `number_by_party_hours` is how many birds were counted divided by the number of person-hours that year
- `flags` contains values like `US` for "unusual" bird (as per the Christmas Bird Count [documentation](https://www.audubon.org/sites/default/files/documents/compilers_manual_0.pdf))

Now we do some regex!

First, I want to split up the `species` variable into the common `species` name and the scientific `species_latin` name.

For the first mutate:
I will use `@kohske`'s regex I found on [StackOverflow](https://stackoverflow.com/questions/8613237/extract-info-inside-all-parenthesis-in-r), which, as Nettle writes:

> I like @kohske's regex, which looks behind for an open parenthesis ?<=\\(, looks ahead for a closing parenthesis ?=\\), and grabs everything in the middle (lazily) .+?, in other words (?<=\\().+?(?=\\))
s

For the second mutate:
As you can see in the code below, there is a line break (`\n`) between every English name and every scientific name in `species`. We will use that to parse out the scientific name:


```r
hamilton_cbc %>% 
  filter(row_number() == 1) %>% 
  pull(species)
```

```
[1] "Snow Goose\r\n[Chen caerulescens]"
```

Here are the two `mutate`s together:


```r
# Putting it together: Mutating the two variables
hamilton_cbc <- hamilton_cbc %>%
  mutate(species_latin = str_extract(species, "(?<=\\[).+?(?=\\])"),
         species = word(species, start = 1, sep = fixed('\n[')))
```

Now we will look at the `count_year` variable. Let's get a sense of what the variable looks like, using the White-Breasted Nuthatch count in 2016:


```r
hamilton_cbc %>% 
  filter(row_number() == 15133) %>% 
  pull(count_year)
```

```
[1] "2016 [117]\r\nCount Date: 12/26/2016\r\n# Participants: 1\r\n# Species Reported: 97\r\nTotal Hrs.: 216.65"
```

The `count_year` variable is actually several variables in one:

- calendar year
- [CBC count number]
- calendar count date
- number of participants
- number of species reported
- total hours spent that year on the count

This is all metadata and we can take most of it out of this dataset. The only variable we will keep in the `hamilton_cbc` dataset is the calendar year.

And where are we at with the `hamilton_cbc` dataset?


```r
hamilton_cbc %>%
  tail()
```

```
# A tibble: 6 x 6
  species   count_year          how_many_cw number_by_party~ flags species_latin
  <chr>     <chr>               <chr>       <chr>            <chr> <chr>        
1 "House S~ "2012 [113]\r\nCou~ 1473        7.5713           <NA>  Passer domes~
2 "House S~ "2013 [114]\r\nCou~ 1802        9.8902           <NA>  Passer domes~
3 "House S~ "2014 [115]\r\nCou~ 1318        7.3529           <NA>  Passer domes~
4 "House S~ "2015 [116]\r\nCou~ 2326        10.5249          <NA>  Passer domes~
5 "House S~ "2016 [117]\r\nCou~ 2565        11.8394          <NA>  Passer domes~
6 "House S~ "2017 [118]\r\nCou~ 2731        13.7409          <NA>  Passer domes~
```

Let's clean up the variables a bit more:


```r
hamilton_cbc <- hamilton_cbc %>%
  rename(participant_info = count_year,
         how_many_counted = how_many_cw) %>%
  mutate(year = as.integer(word(participant_info)),  # We will keep year and total_hours
         total_hours = as.double(
           str_extract(
             participant_info, "(?<=Hrs\\.:\\s).*$")))
```

We almost have a clean dataset! 🔜 ✨✨✨🎉

I am going to remove the `flags` variable. I am also going to remove `number_by_party_hours` and derive it myself instead.


```r
hamilton_cbc <- hamilton_cbc %>%
  select(year, species, species_latin, how_many_counted, total_hours)
```

It turns out that `how_many_counted` also has a `cw` value, which means the bird was not seen on count day itself, but was seen on a day close to the count. I am going to set these bird counts to be `NA`, as they don't have a specified value.


```r
hamilton_cbc <- hamilton_cbc %>%
  mutate(how_many_counted = ifelse(how_many_counted == "cw", NA, how_many_counted),
         how_many_counted = as.integer(how_many_counted))
```

In the `species` variable, there are some rows that are identified only to the genus level (and not to the species level). I will exclude these records, as I believe [eBird](https://ebird.org/home) excludes them too.


```r
hamilton_cbc %>%
filter(str_detect(species, "sp\\.")) %>%
distinct(species)
```

```
# A tibble: 25 x 1
   species            
   <chr>              
 1 "scoter sp.\r"     
 2 "duck sp.\r"       
 3 "loon sp.\r"       
 4 "Accipiter sp.\r"  
 5 "hawk sp.\r"       
 6 "eagle sp.\r"      
 7 "jaeger sp.\r"     
 8 "gull sp.\r"       
 9 "screech-owl sp.\r"
10 "owl sp.\r"        
# ... with 15 more rows
```

```r
hamilton_cbc <- hamilton_cbc %>%
  filter(!(str_detect(species, "sp\\.")))
```

Two final mutates:

- Using `tidyr`'s [`replace_na`](https://rdrr.io/cran/tidyr/man/replace_na.html) function, let's make all of the `NA`s equal to 0 for `how_many_counted`. That means we are assuming that all birds in the area were successfully counted on count day.
- Let's also calculate the number of birds counted (within each species) divided by the total number of count hours that happened that year.


```r
hamilton_cbc <- hamilton_cbc %>%
  mutate(how_many_counted = replace_na(how_many_counted, 0),
         how_many_counted_by_hour = as.double(how_many_counted) / total_hours)
```

And that's it! 😄 🎉 We have cleaned the dataset and are ready to do some visualizing 👀 in Part 2!

# Final dataset

Here is a glimpse of our final dataset:


```r
hamilton_cbc %>%
  tail()
```

```
# A tibble: 6 x 6
   year species    species_latin  how_many_counted total_hours how_many_counted~
  <int> <chr>      <chr>                     <dbl>       <dbl>             <dbl>
1  2012 "House Sp~ Passer domest~             1473        195.              7.57
2  2013 "House Sp~ Passer domest~             1802        182.              9.89
3  2014 "House Sp~ Passer domest~             1318        179.              7.35
4  2015 "House Sp~ Passer domest~             2326        221              10.5 
5  2016 "House Sp~ Passer domest~             2565        217.             11.8 
6  2017 "House Sp~ Passer domest~             2731        199.             13.7 
```




And thank you to the CBC! The CBC Data was provided by [National Audubon Society](www.christmasbirdcount.org) and through the generous efforts of [Bird Studies Canada](www.bsc-eoc.org) and countless volunteers across the western hemisphere.

[^1]: https://news.nationalgeographic.com/news/2014/12/141227-christmas-bird-count-anniversary-audubon-animals-science/


