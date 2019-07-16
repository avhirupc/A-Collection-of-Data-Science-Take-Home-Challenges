URL Parsing Challenge
================
Siddhartha Jetti
7/14/2019

# Goal

Being able to efficiently parse URLs and extract info from them is a
very important skill for a data scientist.

Firstly, if you join a very early stage startup, it might be that a lot
of data are just stored in the URLs visited by the users. And,
therefore, you will have to build a system that parses the URLs,
extracts fields out of it, and saves them into a table that can be
easily queried (not the most fun of the jobs, but very useful\!).

Secondly, often using external data can help a lot your models. And a
way to get external data is by scraping websites. And this is often done
by being able to play with a given site URL structure (assuming it is
allowed by the external site ToS).

The goal of this project is to parse a sequence of URLs about user
searches and extract some basic info out of it.

# Challenge Description

Company XYZ is a Online Travel Agent site, such as Expedia, Booking.com,
etc.

They haven’t invested in data science yet and all the data they have
about user searches are simply stored in the URLs the users generate
when they search for a hotel. If you are not familiar with URLs, you can
run a search on any OTA site and see how all search parameters are
always present in the URL.

You are asked to:

1)  Create a clean data set where each column is a field in the URL,
    each row is a given search and the cells are the corresponding URL
    values.

For each search query, how many amenities were selected?

2)  Often, to measure the quality of a search algorithm, data scientists
    use some metric based on how often users click on the second page,
    third page, and so on. The idea here is that a great search
    algorithm should return all interesting results on the first page
    and never force users to visit the other pages (how often do you
    click on the second page results when you search on Google? Almost
    never, right?).

Create a metric based on the above idea and find the city with the worst
search algorithm.

# Data

The file is:

url\_list - a list of search URLs generated by the users when searching
for hotels

## Fields:

hotel.checkin : checkin date in the search query. It is mandatory
hotel.customMinimumPriceFilter : This filter allows to only return
hotels whose nightly price is above a certain threshold (in USD). Useful
to filter out the really bad hotels hotel.customMaximumPriceFilter :
This filter allows to only return hotels whose nightly price is below a
certain threshold (in USD). Useful to filter out the hotels you can’t
afford hotel.freeCancellation : It is a check box. If the user selects
it, only hotels with free cancellation are returned. hotel.stars\_5 : It
is a check box. If the user selects it, 5-star hotels are returned.
Multiple choices are allowed. For instance, a user can select 5 and 4
star hotels by checking this box and the one below. If no check box is
selected, all hotels are returned. hotel.stars\_4 : It is a check box.
If the user selects it, 4-star hotels are returned hotel.stars\_3 : It
is a check box. If the user selects it, 3-star hotels are returned
hotel.stars\_2 : It is a check box. If the user selects it, 2-star
hotels are returned hotel.stars\_1 : It is a check box. If the user
selects it, 1-star hotels are returned hotel.max\_score : This filter
allows to only return hotels whose score is below a certain threshold.
Score is 1-5 with high being good (you can think of TripAdvisor score to
get an idea of what it is) hotel.min\_score : This filter allows to only
return hotels whose score is above a certain threshold hotel.couponCode
: If the user uses a coupon in her search, this fields gets populated
with “Yes” hotel.adults : Number of adults in the search query. This sis
the number of adults who would stay in the same room. It is mandatory
hotel.city : Which city is the user searching for. It is mandatory
hotel.children : Is the user traveling with children? This field returns
the number of children in the search query hotel.amenities : There are a
few amenities that the user can select in her search via different check
boxes. The possible amenities are: shuttle: free shuttle transportation
from the airport internet: free internet breakfast : free breakfast
lounge : does the hotel have a lounge yes\_smoking : are there rooms
where smoking is allowed yes\_pet : is it allowed to bring pets
hotel.checkout : Check out date. It is mandatory hotel.search\_page :
Search page visited. 1 means the user in on the first page results, 2
-\> clicked on the second page etc. This will be used to estimate the
goodness of ranking for different cities

# Problem Setup

``` r
# Load required libraries
library(tidyverse)
```

    ## Registered S3 methods overwritten by 'ggplot2':
    ##   method         from 
    ##   [.quosures     rlang
    ##   c.quosures     rlang
    ##   print.quosures rlang

    ## ── Attaching packages ──────────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.1.1     ✔ purrr   0.3.2
    ## ✔ tibble  2.1.1     ✔ dplyr   0.8.1
    ## ✔ tidyr   0.8.3     ✔ stringr 1.4.0
    ## ✔ readr   1.3.1     ✔ forcats 0.4.0

    ## ── Conflicts ─────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(ggplot2)

# Read in the input data into a dataframe
urls <- read.table("url_list.txt", stringsAsFactors = F)
urls1 <- gsub("http://www.mysearchforhotels.com/shop/hotelsearch?", "", urls[,1], fixed = TRUE)
```

# Question 1:

Transform the URL data

``` r
url_list <- strsplit(urls1, "&") 
nurls <- nrow(urls)
```

Process and clean the data

``` r
# all possible URL fields
url_fields <- c("hotel.checkin", "hotel.customMinimumPriceFilter", "hotel.customMaximumPriceFilter", 
                "hotel.freeCancellation", "hotel.stars_5", "hotel.stars_4", "hotel.stars_3", "hotel.stars_2", 
                "hotel.stars_1", "hotel.max_score", "hotel.min_score", "hotel.couponCode", "hotel.adults", 
                "hotel.city", "hotel.children", "hotel.amenities", "hotel.checkout", "hotel.search_page")

# Initialize a data frame to hold the cleaned data in rectangular format
url_data <- as.data.frame(matrix(NA, nrow = nurls, ncol = 19))
names(url_data) <- c("id", url_fields)
url_data$id <- seq(1, nurls, by = 1)
```

Build the rectangular data set.

``` r
# Loop through all the URLs
for(i in 1:nurls) {
  search <- url_list[[i]]
  # Loop through all the fields for every url entry
  for(field in url_fields){
    if(any(grepl(field, search))){
      search_strings <- paste(search[grep(field, search)], collapse = ',')
      url_data[i, field] <- gsub(paste0(field, "="), "", search_strings, fixed = TRUE)
    } else { url_data[i, field] <- NA }
  }
}
# Remove special characters
url_data <- url_data %>%
  mutate(hotel.city = gsub("+", "", hotel.city, fixed = TRUE))

# Take a peek at the final data
summary(url_data)
```

    ##        id        hotel.checkin      hotel.customMinimumPriceFilter
    ##  Min.   :    1   Length:77677       Length:77677                  
    ##  1st Qu.:19420   Class :character   Class :character              
    ##  Median :38839   Mode  :character   Mode  :character              
    ##  Mean   :38839                                                    
    ##  3rd Qu.:58258                                                    
    ##  Max.   :77677                                                    
    ##  hotel.customMaximumPriceFilter hotel.freeCancellation hotel.stars_5     
    ##  Length:77677                   Length:77677           Length:77677      
    ##  Class :character               Class :character       Class :character  
    ##  Mode  :character               Mode  :character       Mode  :character  
    ##                                                                          
    ##                                                                          
    ##                                                                          
    ##  hotel.stars_4      hotel.stars_3      hotel.stars_2     
    ##  Length:77677       Length:77677       Length:77677      
    ##  Class :character   Class :character   Class :character  
    ##  Mode  :character   Mode  :character   Mode  :character  
    ##                                                          
    ##                                                          
    ##                                                          
    ##  hotel.stars_1      hotel.max_score    hotel.min_score   
    ##  Length:77677       Length:77677       Length:77677      
    ##  Class :character   Class :character   Class :character  
    ##  Mode  :character   Mode  :character   Mode  :character  
    ##                                                          
    ##                                                          
    ##                                                          
    ##  hotel.couponCode   hotel.adults        hotel.city       
    ##  Length:77677       Length:77677       Length:77677      
    ##  Class :character   Class :character   Class :character  
    ##  Mode  :character   Mode  :character   Mode  :character  
    ##                                                          
    ##                                                          
    ##                                                          
    ##  hotel.children     hotel.amenities    hotel.checkout    
    ##  Length:77677       Length:77677       Length:77677      
    ##  Class :character   Class :character   Class :character  
    ##  Mode  :character   Mode  :character   Mode  :character  
    ##                                                          
    ##                                                          
    ##                                                          
    ##  hotel.search_page 
    ##  Length:77677      
    ##  Class :character  
    ##  Mode  :character  
    ##                    
    ##                    
    ## 

``` r
head(url_data)
```

    ##   id hotel.checkin hotel.customMinimumPriceFilter
    ## 1  1    2015-09-19                           <NA>
    ## 2  2    2015-09-14                           <NA>
    ## 3  3    2015-09-26                           <NA>
    ## 4  4    2015-09-02                           <NA>
    ## 5  5    2015-09-20                           <NA>
    ## 6  6    2015-09-14                           <NA>
    ##   hotel.customMaximumPriceFilter hotel.freeCancellation hotel.stars_5
    ## 1                           <NA>                   <NA>          <NA>
    ## 2                           <NA>                   <NA>          <NA>
    ## 3                            175                   <NA>          <NA>
    ## 4                           <NA>                   <NA>           yes
    ## 5                            275                   <NA>          <NA>
    ## 6                           <NA>                    yes          <NA>
    ##   hotel.stars_4 hotel.stars_3 hotel.stars_2 hotel.stars_1 hotel.max_score
    ## 1           yes          <NA>          <NA>          <NA>            <NA>
    ## 2          <NA>           yes          <NA>          <NA>            <NA>
    ## 3           yes          <NA>          <NA>          <NA>            <NA>
    ## 4           yes          <NA>          <NA>          <NA>            <NA>
    ## 5          <NA>          <NA>          <NA>          <NA>            <NA>
    ## 6          <NA>          <NA>          <NA>          <NA>            <NA>
    ##   hotel.min_score hotel.couponCode hotel.adults
    ## 1               4             <NA>            3
    ## 2               4             <NA>            3
    ## 3               5             <NA>            2
    ## 4               4             <NA>            1
    ## 5               5             <NA>            3
    ## 6            <NA>             <NA>            2
    ##                             hotel.city hotel.children hotel.amenities
    ## 1              NewYork,NY,UnitedStates           <NA>            <NA>
    ## 2                 London,UnitedKingdom           <NA>            <NA>
    ## 3              NewYork,NY,UnitedStates           <NA>            <NA>
    ## 4                    HongKong,HongKong           <NA>            <NA>
    ## 5                 London,UnitedKingdom           <NA>            <NA>
    ## 6 SanFrancisco,California,UnitedStates           <NA>            <NA>
    ##   hotel.checkout hotel.search_page
    ## 1     2015-09-20                 1
    ## 2     2015-09-15                 1
    ## 3     2015-09-27                 1
    ## 4     2015-09-03                 1
    ## 5     2015-09-29                 1
    ## 6     2015-09-16                 1

``` r
# Check if any rows exist with all fields other than id missing
any(rowSums(is.na(url_data)) == ncol(url_data[,-1]))
```

    ## [1] FALSE

Every search query need not have all the fields specified. So, the
cleaned dataset is expected to be sparse (several missing values in each
row). No rows exist with all fields other than id missing.

``` r
# Check if missing values exist for search page
sum(is.na(url_data$hotel.search_page))
```

    ## [1] 0

``` r
# Check missing values for city
sum(is.na(url_data$hotel.city))
```

    ## [1] 0

No missing values exist in critical fields. The data looks good.

Count number of amenities for every search

``` r
# Frequency by categories
table(url_data$hotel.amenities, useNA = "always")
```

    ## 
    ##           breakfast   breakfast,yes_pet            internet 
    ##                  39                   1                 272 
    ##              lounge             shuttle             yes_pet 
    ##                  22                 111                  85 
    ##         yes_smoking yes_smoking,yes_pet                <NA> 
    ##                 170                   4               76973

``` r
# Count the amenities searched by splitting at "," for searches involving multiple amenity selection
url_data <- url_data %>%
  mutate(amenities_count = ifelse(is.na(hotel.amenities), 0, str_count(hotel.amenities, ",") + 1))

table(url_data$amenities_count, useNA = "always")
```

    ## 
    ##     0     1     2  <NA> 
    ## 76973   699     5     0

Clearly, More than 99% of the searches does not have any amenity
selected. Only less than 1% of searches have one or more amenities
selected and highest number of amenities selected in a search is two.

# Question 2:

A good search algorithm should rank the most relevant search results
highly and hence show interesting results in the first page. I think
users having to look at second page of search results is as bad as
having to look at 10th page because most users don’t bother to go beyond
first page and ones shown in later pages may not reach the user. So, it
makes sense to bin searches into two categories one where searches are
found on first page and second category being any where else other than
first page.

The metric I choose to know the relavancy of search results, in other
words performance of search algorithm, is “the fraction of searches with
users looking beyond first search results page”. The lower the value of
this metric, the better is the relavancy of search results and the
search algorithm. If large percentage of users looked beyond the first
page then this metric will go up and indicate that search algorithm is
not performing well.

``` r
# Frequency by searched city
table(url_data$hotel.city, useNA = "always")
```

    ## 
    ##                    HongKong,HongKong                 London,UnitedKingdom 
    ##                                11786                                28058 
    ##              NewYork,NY,UnitedStates SanFrancisco,California,UnitedStates 
    ##                                29384                                 8449 
    ##                                 <NA> 
    ##                                    0

``` r
# Order cities based on the chosen metric
url_data %>%
  mutate(next_search_page = ifelse(hotel.search_page == "1", 0, 1)) %>%
  group_by(hotel.city) %>%
  summarize(next_search_page_fraction = mean(next_search_page)) %>%
  arrange(desc(next_search_page_fraction))
```

    ## # A tibble: 4 x 2
    ##   hotel.city                           next_search_page_fraction
    ##   <chr>                                                    <dbl>
    ## 1 London,UnitedKingdom                                    0.473 
    ## 2 NewYork,NY,UnitedStates                                 0.442 
    ## 3 HongKong,HongKong                                       0.0892
    ## 4 SanFrancisco,California,UnitedStates                    0.0407

It looks like London had the worst search algorithm with users looking
beyond first search results page in about 47% of searches. Followed by
New York with about 44% of searches going beyond first page.