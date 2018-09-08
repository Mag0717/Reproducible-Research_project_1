### INTRODUCTION

This document presents the results regarding the Peer asingment fo
Reproductible research week 2.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012 and
include the number of steps taken in 5 minute intervals each day.

##### This document presents the answers for the following questions regarding the Activity Monitoring Data

1.- What is mean total number of steps taken per day? 2.- What is the
average daily activity pattern? 3.-Are there differences in activity
patterns between weekdays and weekends?

Firstly, get the packages you need

    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(ggplot2)

Load the data into your environment using read.csv()

    activity <- read.csv("activity.csv")

Look at your data

    str(activity)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

    summary(activity)

    ##      steps                date          interval     
    ##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
    ##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
    ##  Median :  0.00   2012-10-03:  288   Median :1177.5  
    ##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
    ##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
    ##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
    ##  NA's   :2304     (Other)   :15840

    head(activity)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

Remove the missing values

    act.complete <- na.omit(activity)

Try dplyr library to sort by day and summarize the steps

    act.day <- group_by(act.complete, date)
    act.day <- summarize(act.day, steps=sum(steps))

Look at the data per day

    summary(act.day)

    ##          date        steps      
    ##  2012-10-02: 1   Min.   :   41  
    ##  2012-10-03: 1   1st Qu.: 8841  
    ##  2012-10-04: 1   Median :10765  
    ##  2012-10-05: 1   Mean   :10766  
    ##  2012-10-06: 1   3rd Qu.:13294  
    ##  2012-10-07: 1   Max.   :21194  
    ##  (Other)   :47

##### Plot the histogram of the total steps per day

    qplot(steps, data=act.day)

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-10-1.png)

### Calculate the mean and median

    mean(act.day$steps)

    ## [1] 10766.19

    median(act.day$steps)

    ## [1] 10765

### What is the average daily activity pattern?

Store a data frame in which steps are aggregated into averages within
each 5 minute interval: \#\#\#\#\# First I create a data frame in which
steps are aggregated into averages within each 5 minute interval:

    act.int <- group_by(act.complete, interval)
    act.int <- summarize(act.int, steps=mean(steps))

Plot the average daily steps \#\#\#\#\# Next I plot the average daily
steps against the intervals:

    ggplot(act.int, aes(interval, steps)) + geom_line()

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-14-1.png)

### The 5-minute interval that, on average, contains the maximum number of steps

    act.int[act.int$steps==max(act.int$steps),]

    ## # A tibble: 1 x 2
    ##   interval    steps
    ##      <int>    <dbl>
    ## 1      835 206.1698

### Check the recuency of the missing values

    nrow(activity)-nrow(act.complete)

    ## [1] 2304

Fill in all of the missing values in the dataset. The strategy does not
need to be sophisticated. replace missing values with the mean number of
steps for each interval across all of the days. The act.int data frame
contains these means.

    names(act.int)[2] <- "mean.steps"
    act.impute <- merge(activity, act.int)

#### Create a new dataset that is equal to the original dataset but with the missing data filled in.

    act.impute$steps[is.na(act.impute$steps)] <- act.impute$mean.steps[is.na(act.impute$steps)]

Plot the histogram of the total number of steps taken each day and
Calculate and report the mean and median total number of steps taken per
day. Do these values differ from the estimates from the first part of
the assignment? \#\#\#\#\# I first create a dataset with the total
number of steps per day using the imputed data:

    act.day.imp <- group_by(act.impute, date)
    act.day.imp <- summarize(act.day.imp, steps=sum(steps))

##### I generate the histogram and summary statistics:

    qplot(steps, data=act.day.imp)

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-20-1.png)

### Mean and Median

    mean(act.day.imp$steps)

    ## [1] 10766.19

    median(act.day.imp$steps)

    ## [1] 10766.19

### Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels ???
???weekday??? and ???weekend??? indicating whether a given date is a
weekday or weekend day.

    act.impute$dayofweek <- weekdays(as.Date(act.impute$date))
    act.impute$weekend <-as.factor(act.impute$dayofweek=="Saturday"|act.impute$dayofweek=="Sunday")
    levels(act.impute$weekend) <- c("Weekday", "Weekend")

##### First I create separate data frames for weekends and weekdays:

    act.weekday <- act.impute[act.impute$weekend=="Weekday",]
    act.weekend <- act.impute[act.impute$weekend=="Weekend",]

##### Later for each one, I find the mean number of steps across days for each 5 minute interval:

    act.int.weekday <- group_by(act.weekday, interval)
    act.int.weekday <- summarize(act.int.weekday, steps=mean(steps))
    act.int.weekday$weekend <- "Weekday"
    act.int.weekend <- group_by(act.weekend, interval)
    act.int.weekend <- summarize(act.int.weekend, steps=mean(steps))
    act.int.weekend$weekend <- "Weekend"

##### I append the two data frames together, and I make the two time series plots:

    act.int <- rbind(act.int.weekday, act.int.weekend)
    act.int$weekend <- as.factor(act.int$weekend)
    ggplot(act.int, aes(interval, steps)) + geom_line() + facet_grid(weekend ~ .)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-26-1.png)
