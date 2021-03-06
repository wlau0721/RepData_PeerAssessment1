---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

Reproducible Research: Peer Assessment 1
=============

Prepared by Wilson Lau
-------------


## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## Data

The data for this assignment can be downloaded from the course web site:

- Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

- date: The date on which the measurement was taken in YYYY-MM-DD format

- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Loading and preprocessing the data


```r
setwd('C:\\Data2\\Coursera\\ReproducibleResearch\\RepData_PeerAssessment1')
library(ggplot2)
library(knitr)

#read data using read.csv
if (sum(dir(path = ".") == "activity.csv") > 0) {
  data <- read.csv(".\\activity.csv",header=TRUE,sep=",")
}
```


## What is mean total number of steps taken per day?

### 1. Calculate the total number of steps taken per day


```r
#sum steps by each date
totalStepsByDate <- aggregate(steps ~ date,data=cdata, FUN = sum)
```

### 2. Make a histogram of the total number of steps taken each day


```r
#plot total steps by date
hist(totalStepsByDate$steps, main="Histogram of the total number of steps taken each day",
     xlab = "Total number of skeps taken each day", ylab = "Number of days")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

### 3. Calculate and report the mean and median of the total number of steps taken per day

#### The mean of the total number of steps taken per day is: 10766.19


```r
#calcuate mean for totalStepsByDate
mean(totalStepsByDate$steps)
```

```
## [1] 10766.19
```

#### The median of the total number of steps taken per day is 10765


```r
#calcuate median for totalStepsByDate
median(totalStepsByDate$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
#convert date
totalStepsByDate$date <- as.Date(as.Date(totalStepsByDate$date,"%Y-%m-%d"))

#average steps by each interval across all days
meanStepsByInterval <- aggregate(steps ~ interval,data=cdata, FUN = mean)

#plot time series chart 
plot(meanStepsByInterval$interval,meanStepsByInterval$steps,type="l", 
     main="Time Series Chart of average number of steps taken across all days",
     xlab="5 mins Time Interval",ylab="Average number of steps")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

The maximum average steps interval is 835 (8:35am) averaging 206.17 steps across all days.


```r
#which interval contains the max number of steps
meanStepsByInterval[which(meanStepsByInterval$steps == max(meanStepsByInterval$steps)),]
```

```
##     interval    steps
## 104      835 206.1698
```


## Imputing missing values

### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

The total number of missing values is 2304. 


```r
#total number of NA value
sum(is.na(data$steps))
```

```
## [1] 2304
```

### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

I will fill in all of the missing values with the mean for that interval

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
#make copy of the original data
datanew <- data

#replace NA with the mean for that interval across all days
for (x in which(is.na(datanew$steps))) {
    datanew[x,1] <- meanStepsByInterval[which(meanStepsByInterval$interval == datanew[x,3]),2]
}
```

### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
#calculate new total steps by Date with new dataset
totalStepsByDateNew <- aggregate(steps ~ date,data=datanew, FUN = sum)

#plot total steps by date with new dataset
hist(totalStepsByDateNew$steps, main="Histogram of the total number of steps taken each day using new dataset",
     xlab = "Total number of skeps taken each day", ylab = "Number of days")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

#### The mean of the total number of steps taken per day is 10766.19


```r
#calcuate mean for totalStepsByDateNew
mean(totalStepsByDateNew$steps)
```

```
## [1] 10766.19
```

#### The median of the total number of steps taken per day is 10766.19


```r
#calcuate median for totalStepsByDateNew
median(totalStepsByDateNew$steps)
```

```
## [1] 10766.19
```

## Are there differences in activity patterns between weekdays and weekends?

### 1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
#convert date for new dataset with NA filled
datanew$date <- as.Date(as.Date(datanew$date,"%Y-%m-%d"))

#create new column to indicate whether the date is weekday or weekend
datanew$dayofweek <- 0  

#find out the date and fill the new dayofweek column
for (x in 1:dim(datanew)[1]) {
    if (sum(weekdays(datanew[x,2]) == c("Monday","Tuesday","Wednesday","Thursday","Friday")) > 0) {
        datanew[x,4] <- "weekday"   #set weekday
    }
    else {
        datanew[x,4] <- "weekend"   #set weekday
    }
}

#set dayofweek column as factor
datanew$dayofweek <- factor(datanew$dayofweek)
```

### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 


```r
#compute average steps for each interval for all days by weekday or weekend 
meanStepsByIntervalnew <- aggregate(steps ~ interval+dayofweek,data=datanew, FUN = mean)

# make panel plot
g <- ggplot(meanStepsByIntervalnew,aes(interval,steps))
g1 <- g + geom_line(size = 1) + xlab("5 mins Time Interval") + ylab("Average number of steps") + 
    ggtitle("Time series plot of Interval and average steps across all weekday or weekend days")
g2 <- g1 + facet_grid(dayofweek ~ .)
print(g2)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14-1.png) 

There are differences between weekday and weekend.  
