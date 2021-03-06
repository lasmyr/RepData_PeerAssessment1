---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
Reading the main csv into the variable **activity** 

```r
activity<-read.csv("activity.csv", header = TRUE)
```

Look at the data

```r
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

Initiate all the packages needed

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(lattice)
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

## What is mean total number of steps taken per day?
Add all the steps by date through aggregate. Arrange by date to the variable **totalsteps**


```r
totalsteps<-aggregate(activity$steps,
                    by = list(activity$date),
                    FUN = sum,
                    na.rm=TRUE)
colnames(totalsteps)<-c("date", "totalsteps")
totalsteps<-arrange(totalsteps, date)
```

#plot histogram
Plot the histogram to see which steps are most frequent

```r
hist(totalsteps$totalsteps, xlab = "Total Steps", main = "Total Steps per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

#get mean and median
Run the function summary to see the mean and median

```r
summary(totalsteps)
```

```
##          date      totalsteps   
##  2012-10-01: 1   Min.   :    0  
##  2012-10-02: 1   1st Qu.: 6778  
##  2012-10-03: 1   Median :10395  
##  2012-10-04: 1   Mean   : 9354  
##  2012-10-05: 1   3rd Qu.:12811  
##  2012-10-06: 1   Max.   :21194  
##  (Other)   :55
```
Another way to get the mean is to use the mean function below:

```r
meandailysteps<-mean(totalsteps$totalsteps, na.rm=TRUE)
```

Mean is 10766.188, median is 10765

## What is the average daily activity pattern?
Average the total steps per interval


```r
aveinterval<-aggregate(activity$steps,
                       by = list(activity$interval),
                       FUN = mean,
                       na.rm=TRUE)
colnames(aveinterval)<-c("interval", "steps")
```

Plot the average steps per interval using the base plot system of r


```r
plot(aveinterval$interval, aveinterval$steps, type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

Get the time interval with the most activity by subsetting the interval vector like so:


```r
maxstep<-max(aveinterval$steps)
maxstepint<-aveinterval$interval[which(aveinterval$steps == maxstep)]
```

Thus we can see 8:35 is the most active interval

## Inputing missing values
Filter only the NAs by identifying rows with NAs, then using the filter function

```r
activity$na<-ifelse(is.na(activity$steps)==TRUE, "TRUE", "FALSE") 
activityNA<-filter(activity, activity$na == TRUE)
```

My strategy for filling out missing values is to use the average steps taken per interval across all days, and use that value to fill up missing values with the matching interval.

This was dones through merging the average interval dataframe with the original activity dataframe:


```r
activityNA<-merge(activityNA, aveinterval, by = "interval")
activityNA<-select(activityNA, c("steps.y", "interval", "date"))
colnames(activityNA)<-c("NAsteps", "interval", "date")

#merge back to original activity dataframe, and replace NAs with values

activityx<-left_join(activity, activityNA, by = c("date", "interval"))
activityx$realstep<-ifelse(activityx$na=="TRUE", activityx$NAsteps, activityx$steps)
activityx<-select(activityx, c("realstep", "date", "interval"))
colnames(activityx)<-c("steps", "date", "interval")
```

Now again, sum by date to see the total steps taken per day, this time with inputted values.


```r
dailystep<-aggregate(activityx$steps,
                     by = list(activityx$date),
                     FUN = sum)
colnames(dailystep)<-c("date", "total.daily.step")
```

Now we can plot a histogram as did before

```r
hist(dailystep$total.daily.step, xlab = "Total Daily Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png)<!-- -->
and we can get the mean again using the summary function

```r
summary(dailystep)
```

```
##          date    total.daily.step
##  2012-10-01: 1   Min.   :   41   
##  2012-10-02: 1   1st Qu.: 9819   
##  2012-10-03: 1   Median :10766   
##  2012-10-04: 1   Mean   :10766   
##  2012-10-05: 1   3rd Qu.:12811   
##  2012-10-06: 1   Max.   :21194   
##  (Other)   :55
```

Mean and median are both 10766

## Are there differences in activity patterns between weekdays and weekends?

To see this first we have to coerce dates into the date format, as right now it is still as factor. We can use the as.Date function


```r
activityx$date<-as.Date(activity$date, format="%Y-%m-%d")
```

From there we can use lubridate to create a type of day column, which we can use to surmise if it is a weekend or weekday. For ease of use, we can add another column that states whether the date is weekend or weekday


```r
activityx$day<-weekdays(activityx$date)
activityx$typeday<-ifelse(activityx$day %in% c("Sunday", "Saturday"), "weekend", "weekday")
```

Now, we can average steps per interval per type of day


```r
activitywk<-aggregate(activityx$steps,
                      by=list(activityx$interval, activityx$typeday),
                      FUN = mean)
                      colnames(activitywk)<-c("interval", "daytype", "steps")
```
                      
Now create a lattice plot so we can see the difference of the activity on weekends and weekdays


```r
xyplot(steps~interval|daytype, activitywk, type ="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-19-1.png)<!-- -->
