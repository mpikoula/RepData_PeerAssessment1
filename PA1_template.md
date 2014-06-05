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
per_min <- ddply(activity, .(interval), summarise, steps = mean(steps,na.rm=TRUE))
str(per_min)
```

```
## 'data.frame':	288 obs. of  2 variables:
##  $ interval: Factor w/ 288 levels "0","5","10","15",..: 1 2 3 4 5 6 7 8 9 10 ...
##  $ steps   : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
```

## What is mean total number of steps taken per day?


```r
 hist(daily$steps,nrow(daily))
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

The mean total number of steps taken per day is 9354 steps. The median is 10395 steps.

## What is the average daily activity pattern?


```r
plot(per_min$interval,per_min$steps)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

The 5-minute interval when most steps are taken (on average) is the 835th interval.

## Imputing missing values

The total number of missing values is 2304.


```r
activity_compl <- activity
na_index <- which(is.na(activity_compl$steps))
minutes <- activity_compl$interval[na_index]
missing <- vector('integer', length=0)
for (i in minutes) {
  b <- per_min$steps[per_min$interval == i]
  missing <- c(missing,b)
}
#Replace NAs with their calculated couterpars:
activity_compl$steps[na_index] <- missing

#Create a new per day data.frame:

daily_compl <- ddply(activity_compl, .(date), summarise, steps = sum(steps,na.rm=TRUE))

#And now calculate the new histogram

 hist(daily_compl$steps,nrow(daily_compl))
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

The new mean total number of steps taken per day is 1.0766 &times; 10<sup>4</sup> steps. The median is 1.0766 &times; 10<sup>4</sup> steps.

The new values are different to those calculated by simply ignoring NAs. More precisely, the mean has changed by +1411 steps and the median by + 371 steps. Most of the difference seems to stem from the absence of most of 0-steps days, for which data was not available. This ends up making the distribution across the days look more "normal", hence the bigger influence on the mean, rather than the median value.

## Are there differences in activity patterns between weekdays and weekends?
