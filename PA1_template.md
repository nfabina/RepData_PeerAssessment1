---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
#### Load the data (i.e. read.csv())


```r
activity<-read.csv("activity.csv")
```

#### Process/transform the data into a format suitable for analysis

```r
activity$NewDate<- format(as.Date(activity$date, format = "%Y-%m-%d"), "%m/%d/%Y")
activity$date<- NULL
```


## What is mean total number of steps taken per day?
#### Removing NA values from the dataset.

```r
activityNew<- na.omit(activity)
```

#### Total number of steps taken everyday.

```r
total <- aggregate(activityNew$steps, list(Date = activityNew$NewDate), sum)
```

#### Mean number of steps taken everyday.

```r
mean_total_number_of_steps <- aggregate(activityNew$steps, list(activityNew$NewDate), na.rm = TRUE, mean)
```

#### Plotting a histogram of the total number of steps taken each day.

```r
hist(total$x, main = "Histogram of the total number of steps taken each day", xlab = "Total number of steps taken each day", border = "blue", col = "red")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

#### Total mean and median.

```r
round(mean(total$x))
```

```
## [1] 10766
```

```r
median(total$x)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
#### Average number of steps taken, averaged across all days

```r
average_daily_activity.ts <- aggregate(steps ~ interval, data =  activityNew, mean)

## Time Series plot of the 5-minute interval on the x-axis and the average number of steps taken, averaged across all days on the y-axis.
plot(average_daily_activity.ts, main = "Average number of steps by interval", ylab = " Average number of steps taken", xlab = "Time intervals", type = "l", pch = 20)
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

```r
## The maximum number of steps in the 5-minute interval on average across all the days in the dataset.

average_daily_activity.ts[which.max(average_daily_activity.ts$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```

```r
## The maximum number of steps in the 5-minute interval on average across all the days in the dataset is at interval 835 with a value of 206.1698.
```

## Imputing missing values

```r
## The total number of rows with NA's
sum(is.na(activity$steps))
```

```
## [1] 2304
```

```r
## Creating a new dataset with NA values
activity.NA<-activity
```


```r
## Filling in all of the missing values in the dataset with the mean for that 5-minute interval
for(i in 1:nrow(activity.NA)) { 
  if(is.na(activity.NA$steps[i]))
  {avg_value<- average_daily_activity.ts$steps[which(average_daily_activity.ts$interval == activity.NA$interval[i])]
    
  activity.NA$steps[i]<-avg_value
  }
}
```


```r
## A new dataset that is equal to the original dataset but with the missing data filled in with the mean of the 5-minute interval.
total_steps_per_day.NA<- aggregate(steps ~ NewDate, data =  activity.NA, sum)
```


```r
##  Histogram of the total number of steps taken each day
hist(total_steps_per_day.NA$steps, main = "Histogram of the total number of steps taken each day", xlab = "Total number of steps taken each day", border = "blue", col = "red")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

```r
## Total mean and median.
round(mean(total_steps_per_day.NA$steps))
```

```
## [1] 10766
```

```r
(median(total_steps_per_day.NA$steps))
```

```
## [1] 10766.19
```
#### Mean values stays the same but there is slight difference in meadian value.

## Are there differences in activity patterns between weekdays and weekends?

```r
## Creating a new factor variable in the dataset with two levels such as "weekday" and "weekend" 

week_day <- function(date_val) {
  wd <- weekdays(as.Date(date_val, '%m/%d/%Y'))
  if  (!(wd == 'Saturday' || wd == 'Sunday')) {
    x <- 'Weekday'
  } else {
    x <- 'Weekend'
  }
  x
}
activity.NA$day_type <- as.factor(sapply(activity.NA$NewDate, week_day))

## The average number of steps taken, averaged across all weekday days or weekend days
mean_type_of_day<- aggregate(steps ~ interval + day_type, activity.NA, mean)

## Plot containing the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.4.3
```

```r
qplot(interval, 
      steps, 
      data = mean_type_of_day, 
      type = 'l', 
      geom=c("line"),
      xlab = "Interval", 
      ylab = "Number of steps", 
      main = "No of steps Per Interval by day type") +
      facet_wrap(~ day_type, ncol = 1)
```

```
## Warning: Ignoring unknown parameters: type
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->


