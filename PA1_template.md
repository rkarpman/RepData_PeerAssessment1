---
title: "Reproducible Research: Peer Assessment 1"
author: "Rachel Karpman"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

We load the activity data, and convert the `date` column to class `POSIXct` so that we can later use the `weekdays` function on the dates.

It appears that each interval was originally encoded by giving its start time as the number of hours since the time recorded as `0`, followed by the number of minutes, with no spaces or punctuation.  For example, the number `135` would represent ''1 hour, 35 minutes since time `0`.''  For easier plotting, we edit the intervals column so that the intervals are simply numbered in order, starting at `0`.  So for example, interval `5` takes place from 15-20 minutes after time `0`.


```r
data <- read.csv("activity.csv", header = TRUE, na.strings = "NA", 
                 colClasses = c("integer", "POSIXct", "integer"))
convert <- function(time){
    return ((time %/% 100 * 60 + time %% 100)/5)
}
data$interval <- convert(data$interval)
```

## What is the mean total number of steps taken per day?

First, we will attempt to answer this question ignoring all missing values.  We use `tapply` to obtain a vector `by_day` which gives total number of steps for each day.


```r
by_day <- tapply(data$steps, data$date, sum, na.rm = TRUE)
mean(by_day)
```

```
## [1] 9354.23
```

```r
median(by_day)
```

```
## [1] 10395
```

The average number of steps per day is approximately 9354.23 and the median is 10395.


```r
library(ggplot2)
df <- data.frame(by_day)
ggplot(df, aes(df$by_day)) +
           geom_histogram(binwidth = 500, color = "red4", fill = "red3") +
           labs(x = "Steps", y = "Number of Days") +
           ggtitle("Histogram of Total Steps per Day")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

## What is the average daily activity pattern?


```r
library(dplyr)
df <- data %>% group_by(interval) %>% summarize(steps = mean(steps, na.rm = TRUE))
ggplot(df, aes(interval, steps)) +
    geom_line(color = "red4") +
    labs(x = "Five-minute Interval", y = "Average Number of Steps") +
    ggtitle("Average Number of Steps Per Interval")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />

We calculate the interval with the highest average number of steps

```r
    max_interval <- df$interval[which.max(df$steps)]
```

To make this answer more meaningful, we convert it to hours and minutes:

```r
    ## compute number of hours
    (max_interval * 5) %/% 60
```

```
## [1] 8
```

```r
    ## compute number of minutes
    (max_interval * 5) %% 60
```

```
## [1] 35
```
The highest average number of steps occurs in the 5-minute interval that starts 8 hours and 35 minutes after time `0`.

## Imputing Missing Values


```r
missing <- is.na(data$steps)
## compute the number of missing values
sum(missing)
```

```
## [1] 2304
```
There are 2304 missing values in our data set.  We replace each missing value by the average number of steps for the corresponding interval, which we can obtain from the dataframe `df` constructed above.

```r
for(idx in 1:dim(data)[1]){
    if(is.na(data$steps[idx])){
        my_interval <- which(df$interval == data$interval[idx])
        data$steps[idx] <- df$steps[my_interval]
        }
    }
```

We then repeat the computations to find the average and median number of steps per day.


```r
by_day <- tapply(data$steps, data$date, sum)

## compute the mean:
mean(by_day)
```

```
## [1] 10766.19
```

```r
## compute the median:
median(by_day)
```

```
## [1] 10766.19
```

Imputing the missing values cuases both the mean and the median to increase; but the mean increases more.  Without the missing values, the mean was significantly smaller than the median.


```r
df <- data.frame(by_day)
ggplot(df, aes(df$by_day)) +
    geom_histogram(binwidth = 500, color = "red4", fill = "red3") +
    labs(x = "Steps", y = "Number of Days") +
    ggtitle("Total Steps per Day (Missing Data Imputed)")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-11-1.png" style="display: block; margin: auto;" />
From the graph, it appears that the large number of days with zero steps in our original dataset was due to missing data; perhaps corresponding to days when the subject did not carry their device.

## Are there differences in activity patterns between weekdays and weekends?

We create a factor variable which records whether each date is a weekday or weekend, and plot the number of steps in each interval averaged over all weekdays, and over all weekends.


```r
weekend <- function(day){
    weekdays(day) %in% c("Saturday", "Sunday")
}

data$weekend <- factor(weekend(data$date), labels = c("Weekday", "Weekend"))
library(dplyr)
my_data <- data %>% group_by(weekend, interval) %>% summarize(meansteps = mean(steps))
ggplot(data = my_data, aes(x = interval, y = meansteps)) + geom_line(color = "red4") +
    facet_grid(weekend ~ .) +
    labs(x = "Five-Minute Interval", y = "Average Number of Steps") + 
    ggtitle("Average Steps Per Five-Minute Interval")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-12-1.png" style="display: block; margin: auto;" />
Assuming the start time for daily measurements occurs at midnight, we see that on the weekends the subject is less active early in the morning, but more active in the afternoon and evening.  Assuming a typical work or school schedule, this is a very plausible finding.
