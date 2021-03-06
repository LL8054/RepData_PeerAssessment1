---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

This is Peer Assessment 1 from the Reproducible Research class in the Data Scientist Specialization 
course.  The data used in this assignment are contained in this repository.  The code used to access
and manipulate this data to answer the assignment questions are documented in this markdown file.  
Away we go.



###<font color=red>Loading and preprocessing the data </font>

<font color=red>Loading data</font>

```r
data <- read.csv("activity.csv", stringsAsFactors=FALSE)
```

<font color=red>Preprocessing data</font>

```r
#Load Libraries
library(plyr)
library(dplyr)
library(ggplot2)
```


```r
#Turn dates from character string into POSIX class for sortability reasons.
data$date <- as.POSIXct(data$date, format = "%Y-%m-%d")
#Turn data into tbl_df
data <- tbl_df(data)
#Sums total steps per day
steps <- ddply(data, .(date), summarize, steps = sum(steps, na.rm=TRUE))
```



### <font color=blue> What is mean total number of steps taken per day? </font>

<font color=blue> Histogram of the total number of steps taken each day </font>

```r
ggplot(steps, aes(date, steps)) + geom_histogram(stat="identity", fill = "blue") + theme_bw() + labs(title="Total Steps per Day", x = "Date", y = "Total Steps") + theme(title = element_text(face="bold", color="blue"), axis.title.x = element_text(face="bold", color="blue"), axis.title.y = element_text(face="bold", color="blue"))
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

<font color=blue> Calculate and report the *mean* and *median* total number of steps taken per day.</font>

```r
mean <- mean(steps$steps)
median <- median(steps$steps)
mean <- as.integer(mean)
median <- as.integer(median)
```

**<font color=blue>
The mean number of steps taken each day is 9354.  
The median number of steps taken each day is 10395. 
</font>**



### <font color = green>What is the average daily activity pattern?</font>

<font color = green> Time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis). </font>

```r
intervals <- ddply(data, .(interval), summarize, steps = mean(steps, na.rm=TRUE))
#Add linear column to smooth out non-linear 5-minute interval column
intervals <- mutate(intervals, x=(1:nrow(intervals)))
ggplot(intervals, aes(x, steps)) + geom_line(stat = "identity", color ="green4") + theme_bw() + labs(title = "Average # Steps Across Each Day Per 5 Minutes", x="5 Minute Intervals", y="Avg # Steps") + theme(title = element_text(face="bold", color="green4"), axis.title.x = element_text(face="bold", color="green4"), axis.title.y = element_text(face="bold", color="green4")) + scale_x_discrete(breaks=c(intervals[1,3], intervals[73,3], intervals[145,3], intervals[217,3], intervals[288,3]), labels=c(intervals[1,1], intervals[73,1], intervals[145,1], intervals[217,1], intervals[288,1]))
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

<font color = green> Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps? </font>

```r
intervals <- arrange(intervals, desc(steps))
```

**<font color=green>835 is the 5-minute interval which contains the maximum number of steps. </font>**

### <font color=purple>Imputing missing values

Finding the total number of rows with NAs

```r
NArows <- apply(data, 1, function(x){any(is.na(x))})
```

**The total number of rows with NAs is 2304.**

Filling in the missing variables with the mean across all days for that 5-minute interval in order to create a new dataset.

```r
#Create separate datasets, one with all rows with missing variables and one with all rows of complete variables
NAset <- data[NArows,]
completeSet <- data[!NArows,]

#Merge dataset full of NAs with dataset containing average steps for each interval
new <- merge(NAset, intervals, by="interval")

#Clean up the new dataset then merge with original data to create imputed dataframe
new <- select(new, steps.y, date, interval)
colnames(new)[1] <- "steps"
newData <- rbind(new, completeSet)
newData <- arrange(newData, date, interval)
```

Make a histogram of the total number of steps taken each day.  Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
#Sums total steps per day
newSteps <- ddply(newData, .(date), summarize, steps = sum(steps, na.rm=TRUE))

#Calculates mean and median
newMean <- mean(newSteps$steps)
newMedian <- median(newSteps$steps)
newMean <- as.integer(newMean)
newMedian <- as.integer(newMedian)

#Histogram
ggplot(newData, aes(date, steps)) + geom_histogram(stat="identity", fill = "purple") + theme_bw() + labs(title="Total Steps per Day with Imputed Data", x = "Date", y = "Total Steps") + theme(title = element_text(face="bold", color="purple"), axis.title.x = element_text(face="bold", color="purple"), axis.title.y = element_text(face="bold", color="purple"))
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 

###**Means**

* Original Data: 9354  
* Imputed Data:  10766  

###**Medians**

* Original Data: 10395  
* Imputed Data:  10766

**The impact of imputing the average number of steps per interval taken over all days in the data set adds to the total number of steps taken.  It may be an obvious consequence, as you can only add a positive number of steps and you can't take away steps (a negative number).  The Imputed Mean and the Imputed Median are both 10766 because the NAs in the original data set consist of dates for which there is no data at all - that is, each 5-minute interval for those dates is void of data, hence NA.  So if the average 5-minute values are taken across every day that has those values and then imputed into those 5-minute intervals that have NA values, the sum for those imputed days will be equal.  And the average of each day in the data set will be equal.  And there are enough NA days to have them be the median value. **</font>

###<font color=orange>Are there differences in activity patterns between weekdays and weekends?

Add factor variable of "weekday" or "weekend" to imputed dataset

```r
newData <- mutate(newData, day=weekdays(date))
newData <- mutate(newData, type=ifelse(day=="Saturday" | day=="Sunday", "Weekend", "Weekday"))
newData$type <- as.factor(newData$type)
```

Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
splitIntervals <- ddply(newData, .(interval, type), summarize, steps = mean(steps))
#Add linear column to smooth out non-linear 5-minute intervals column
splitIntervals <- mutate(splitIntervals, x=(rep(1:(nrow(splitIntervals)/2), each =2)))
ggplot(splitIntervals, aes(x, steps, type)) + geom_line(stat = "identity", color ="orange") + theme_bw() + labs(title = "Average # Steps Per 5 Minutes", x="5 Minute Intervals", y="Avg # Steps") + theme(title = element_text(face="bold", color="orange"), axis.title.x = element_text(face="bold", color="orange"), axis.title.y = element_text(face="bold", color="orange")) + facet_wrap( ~type, ncol=1) + scale_x_discrete(breaks=c(splitIntervals[1,4], splitIntervals[73,4], splitIntervals[145,4], splitIntervals[217,4], splitIntervals[289,4], splitIntervals[361,4], splitIntervals[433,4], splitIntervals[505,4], splitIntervals[575,4]), labels=c(splitIntervals[1,1], splitIntervals[73,1], splitIntervals[145,1], splitIntervals[217,1], splitIntervals[289,1], splitIntervals[361,1], splitIntervals[433,1], splitIntervals[505,1], splitIntervals[575,1]))
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12.png) 
</font>
    
