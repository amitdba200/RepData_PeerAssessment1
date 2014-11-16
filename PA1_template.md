---
title: "PA1_template"
author: "amitdba200"
date: "Sunday, November 16, 2014"
output: html_document
---

**Introduction**

It is now possible to collect a large amount of data about personal movement using activity monitoring
devices such as a Fitbit (http://www.fitbit.com), Nike Fuelband (http://www.nike.com/us/en_us/c/nikeplusfuelband),
or Jawbone Up (https://jawbone.com/up). These type of devices are part of the "quantified self"
movement - a group of enthusiasts who take measurements about themselves regularly to improve their
health, to find patterns in their behavior, or because they are tech geeks. But these data remain underutilized
both because the raw data are hard to obtain and there is a lack of statistical methods and
software for processing and interpreting the data.
This assignment makes use of data from a personal activity monitoring device. This device collects data
at 5 minute intervals through out the day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012 and include the number of steps
taken in 5 minute intervals each day.

**Show any code that is needed to**
- Load the data (i.e. read.csv() )
- Process/transform the data (if necessary) into a format suitable for your analysis


```r
library(ggplot2)
## Reading the data file
data<-read.csv("activity.csv")
## Ignore missing values
data<-subset(data,!is.na(data$steps))
```
**What is mean total number of steps taken per day?**

For this part of the assignment, you can ignore the missing values in the dataset.

- *1 . Make a histogram of the total number of steps taken each day*

```r
result<-aggregate(data$steps, by=list(data$date), FUN=sum)
colnames(result)<-c("date","total_steps")
out<- ggplot(result, aes(date, total_steps))+geom_bar(stat = "identity", colour = "blue")
out<-out+xlab("Date")+ylab("Number of steps") + theme(axis.text.x = element_text(angle = 90))
out+ggtitle("Total number of steps taken each day")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

- *2. Calculate and report the mean and median total number of steps taken per day*

```r
assign1_mean<-mean(result$total_steps)
cat("The mean total number of steps taken per day : ",assign1_mean)
```

```
## The mean total number of steps taken per day :  10766.19
```

```r
assign1_median<-median(result$total_steps)
cat("The median total number of steps taken per day : ",assign1_median)
```

```
## The median total number of steps taken per day :  10765
```

**What is the average daily activity pattern?**

- * 1. Make a time series plot (i.e. type = "l" ) of the 5minute interval (xaxis) and the average number of steps taken, averaged across all days (yaxis)*

```r
result<-aggregate(data$steps, by=list(data$interval), FUN=mean)
colnames(result)<-c("interval","mean_steps")
out<- ggplot(result, aes(interval, mean_steps)) + geom_line(color = "blue") 
out<-out+xlab("Interval")+ylab("Average steps") + theme(axis.text.x = element_text(angle = 90))
out+ggtitle("Average number of steps taken each day")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 
- *Which 5minute interval, on average across all the days in the dataset, contains the max number of steps?*


```r
cat("Max number of steps :","")
```

```
## Max number of steps :
```

```r
subset(result,result$mean_steps == max(result$mean_steps))
```

```
##     interval mean_steps
## 104      835   206.1698
```

**Imputing missing values**

-*1. Calculate and report the total number of missing values in the dataset (total number of rows with NAs)*


```r
data<-read.csv("activity.csv")
cat("Total number of missing values in data set :",nrow(subset(data,is.na(data$steps))))
```

```
## Total number of missing values in data set : 2304
```

-*2. Devise a strategy for filling in all of the missing values in the dataset.*

I use the mean for a day's interval to fill each NA value in the steps column.

-*3. Create a new dataset that is equal to the original dataset but with the missing data filled in.*

```r
data<-subset(data,!is.na(data$steps))
result<-aggregate(data$steps, by=list(data$date), FUN=sum)
colnames(result)<-c("date","total_steps")

data<-read.csv("activity.csv")
for (index_count in 1: nrow(data)) 
  {
	if (is.na(data$steps[index_count])) 
		{
		data$steps[index_count] <-subset(result,data$date[index_count]  ==  result$date)
		}
	}
```

- *4.Make a histogram of the total number of steps taken each day*

```r
##result<-aggregate(data$steps, by=list(data$date), FUN=sum)
colnames(result)<-c("date","total_steps")
## datewise chart for total
out<-ggplot(result, aes(date, total_steps))+geom_bar(stat = "identity", colour = "blue")
out<-out+xlab("Date")+ylab("Number of steps") + theme(axis.text.x = element_text(angle = 90))
out+ggtitle("Total number of steps taken each day")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 


-*4. Calculate and report the mean and median total number of steps taken per day*

```r
cat("New - The mean total number of steps taken per day : ",mean(result$total_steps))
```

```
## New - The mean total number of steps taken per day :  10766.19
```

```r
cat("New - The median total number of steps taken per day : ",median(result$total_steps))
```

```
## New - The median total number of steps taken per day :  10765
```

```r
cat("Old - The mean total number of steps taken per day : ",assign1_mean)
```

```
## Old - The mean total number of steps taken per day :  10766.19
```

```r
cat("Old - The median total number of steps taken per day : ",assign1_median)
```

```
## Old - The median total number of steps taken per day :  10765
```
-*4. Do these values differ from the estimates from the first part of the assignment?*

No these values dont differ.

**Are there differences in activity patterns between weekdays and weekends?**
- *1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.*

```r
data<-read.csv("activity.csv")
data<-subset(data,!is.na(data$steps))
data$weekdays <-weekdays(as.Date(as.character(data$date)))
levels(data$weekdays) <- list(weekday = c("Monday", "Tuesday","Wednesday","Thursday", "Friday"),weekend = c("Saturday", "Sunday"))
result <-aggregate(data$steps,list(interval = as.numeric(as.character(data$interval)),weekdays = data$weekdays),FUN = mean)
```

- *2. Make a panel plot containing a time series plot (i.e. type = "l" ) of the 5minute interval (xaxis) and the average number of steps taken, averaged across all weekday days or weekend days (yaxis)*

```r
colnames(result)<-c("interval","weekdays","mean_steps")
library(lattice)
xyplot(result$mean_steps ~ result$interval|result$weekdays,layout = c(1, 2), type = "l")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) ![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-2.png) ![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-3.png) ![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-4.png) 