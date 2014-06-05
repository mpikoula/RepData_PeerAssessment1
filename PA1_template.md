# Reproducible Research: Peer Assessment 1

## Loading useful packages for the analysis

```r
library(plyr)
library(lattice)
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

Create new data frame for daily activity with two variables, dates and total steps per day:


```r
daily <- ddply(activity, .(date), summarise, steps = sum(steps,na.rm=TRUE))
per_min <- ddply(activity, .(interval), summarise, steps = mean(steps,na.rm=TRUE))
str(per_min)
```

```
## 'data.frame':	288 obs. of  2 variables:
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
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
plot(per_min$interval,per_min$steps, type="l")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

The 5-minute interval when most steps are taken (on average) is the 835th interval.

## Imputing missing values

The total number of missing values is 2304. Now we create the new data frame which includes all completed cases. The strategy to do this involves replacing NAs with the average value for the particular interval across all days.


```r
activity_compl <- activity #make a copy of the data frame
na_index <- which(is.na(activity_compl$steps)) #get the indexes of all NAs for number of steps
minutes <- activity_compl$interval[na_index] #add the interval values of NAs to a vector

#And now we create a new vector with the replacement values:

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

The new mean total number of steps taken per day is 10766 steps. The median is `as.integer(r median(daily_compl$steps))` steps.

The new values are different to those calculated by simply ignoring NAs. More precisely, the mean has changed by +1411 steps and the median by + 371 steps. Most of the difference seems to stem from the absence of most of 0-steps days, for which data was not available. This ends up making the distribution across the days look more "normal", hence the bigger influence on the mean, rather than the median value.

## Are there differences in activity patterns between weekdays and weekends?


```r
activity_compl["day"] <- weekdays(as.Date(activity_compl$date)) #first assign days
#then assign weekdays:
activity_compl$day[activity_compl$day != "Saturday" & activity_compl$day != "Sunday"] <- "weekday"
#weekends:
activity_compl$day[activity_compl$day == "Saturday" | activity_compl$day == "Sunday"] <- "weekend"
#and finally convert to factor instead of character:
activity_compl$day <- as.factor(activity_compl$day)
#check to see if it works:
str(activity_compl)
```

```
## 'data.frame':	17568 obs. of  4 variables:
##  $ steps   : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
##  $ day     : Factor w/ 2 levels "weekday","weekend": 1 1 1 1 1 1 1 1 1 1 ...
```
Plotting weekend vs weekdays
 

```r
 #First we create the per-minute interval data frame as before (keeping also the day type variable):
 per_min_compl <- ddply(activity_compl, .(interval,day), summarise, steps = mean(steps))
 str(per_min_compl)
```

```
## 'data.frame':	576 obs. of  3 variables:
##  $ interval: int  0 0 5 5 10 10 15 15 20 20 ...
##  $ day     : Factor w/ 2 levels "weekday","weekend": 1 2 1 2 1 2 1 2 1 2 ...
##  $ steps   : num  2.2512 0.2146 0.4453 0.0425 0.1732 ...
```

And we plot for each type of day (weekday or weekend):


```r
xyplot(steps ~ interval | day,data=per_min_compl,type="l", layout=c(1,2))
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 
