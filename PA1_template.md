# Reproducible Research: Peer Assessment 1

## Loading packages

```r
library(plyr)
```
## Loading and preprocessing the data


```r
activity <- read.csv("activity.csv")
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
summary(activity)
```

```
##      steps               date          interval   
##  Min.   :  0.0   2012-10-01:  288   Min.   :   0  
##  1st Qu.:  0.0   2012-10-02:  288   1st Qu.: 589  
##  Median :  0.0   2012-10-03:  288   Median :1178  
##  Mean   : 37.4   2012-10-04:  288   Mean   :1178  
##  3rd Qu.: 12.0   2012-10-05:  288   3rd Qu.:1766  
##  Max.   :806.0   2012-10-06:  288   Max.   :2355  
##  NA's   :2304    (Other)   :15840
```

Create new data frame for daily activity with two variables, dates and spd (steps per day):


```r
daily <- ddply(activity, .(date), summarise, steps = sum(steps,na.rm=TRUE))
activity$interval <- as.factor(activity$interval)
per_min <- ddply(activity, .(interval), summarise, steps = sum(steps,na.rm=TRUE))
str(per_min)
```

```
## 'data.frame':	288 obs. of  2 variables:
##  $ interval: Factor w/ 288 levels "0","5","10","15",..: 1 2 3 4 5 6 7 8 9 10 ...
##  $ steps   : int  91 18 7 8 4 111 28 46 0 78 ...
```

## What is mean total number of steps taken per day?


```r
 hist(daily$steps,nrow(daily))
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

The mean total number of steps taken per day is 9354.2295 steps. The median is 10395 steps.

## What is the average daily activity pattern?


```r
plot(per_min$interval,per_min$steps)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

The 5-minute interval when most steps are taken (on average) is the 835th interval.

## Imputing missing values


The total number of missing values is 2304.

## Are there differences in activity patterns between weekdays and weekends?
