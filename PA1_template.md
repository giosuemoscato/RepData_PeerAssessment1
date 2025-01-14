---
title: 'Reproducible Research: Peer Assessment 1'
output:
  html_document:
    keep_md: yes
  pdf_document: default
---

```r
library(ggplot2)
```


## Loading and preprocessing the data

```r
temp <- tempfile()
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",temp)
data <- read.csv(unz(temp, "activity.csv"))
unlink(temp)
```

## What is mean total number of steps taken per day?

#### 1. Calculate the total number of steps taken per day

```r
data_per_day <- as.data.frame(with(data, tapply(steps, date, sum)))
names(data_per_day) <- "steps per day"
```

#### 2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
hist(data_per_day$`steps per day`, main = "Total number of steps taken each day", xlab = "Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

#### 3. Calculate and report the mean and median of the total number of steps taken per day

```r
my_mean <- as.integer(mean(data_per_day$`steps per day`, na.rm = TRUE))
my_median <- median(data_per_day$`steps per day`, na.rm = TRUE)
```
The mean of the total number of steps taken per day is : **10766**

The median of the total number of steps taken per day is: **10765**

## What is the average daily activity pattern?

#### 1.Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
interval <- as.data.frame(with(data, tapply(steps, interval, mean, na.rm=TRUE)))
plot(x=rownames(interval), y=interval$`with(data, tapply(steps, interval, mean, na.rm = TRUE))`, type="l", xlab = "Average number of steps", ylab = "Interval No.", main = "Average number of steps per interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

#### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
index <- which.max(interval$`with(data, tapply(steps, interval, mean, na.rm = TRUE))`)
my_max <- interval[index,1]
my_5min <- as.integer(rownames(interval)[index])
```

The 5-minute interval the contains the maximum number of steps is the interval **835**, with an average number of steps of **206.1698113**

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (codes as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

#### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
NAs <- sum(is.na(data))
```

The total number of missing values in the dataset is: **2304**

#### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

My strategy will be the one to use the mean for the 5-minute interval for filling in all of the missing values in the dataset that do not have the number of steps taken in that interval

#### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
NewData <- data
mean_by_interval <- as.data.frame(x=with(data, tapply(steps, interval, mean, na.rm=TRUE)))
mean_by_interval <- data.frame(mean=mean_by_interval[,1], interval=as.numeric(rownames(mean_by_interval)))

row_index <- as.numeric(rownames(NewData[is.na(NewData$steps),]))
for (i in 1:length(row_index)){
       my_interval <- NewData[row_index[i],3]
        value <- mean_by_interval[mean_by_interval$interval== my_interval,]$mean
        NewData[row_index[i],1] <- value
}
```


#### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
data_per_day2 <- as.data.frame(with(NewData, tapply(steps, date, sum)))
names(data_per_day2) <- "steps per day"
hist(data_per_day2$`steps per day`, main = "Total number of steps taken each day", xlab = "Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
my_mean2 <- as.integer(mean(data_per_day2$`steps per day`))
my_median2 <- as.integer(median(data_per_day2$`steps per day`))
```
After the imputation of missing data, the mean of the total number of steps taken per day is : **10766**

After the imputation of missing data, the median of the total number of steps taken per day is: **10766**

Previous values of mean and median were respectively **10766** and **10765**, so the impact of imputing missing data on the estimates of the total daily number of steps is minimal

By comparing the two histograms we can't notice relevant differences in the total daily number of steps

```r
par(mfrow=c(1,2), mar=c(5,4,7,2))
hist(data_per_day$`steps per day`, main = "Original Data", xlab = "Steps")
hist(data_per_day2$`steps per day`, main = "After imputing missing values", xlab = "Steps")
mtext("Total number of steps taken each day", side = 3, line = -2, outer = TRUE, cex = 1.5, font = 2)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->


## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

#### 1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
NewData[,2] <- as.Date(NewData[,2])
NewData$daytype <- "weekday"
for (i in 1:length(NewData$date)){
        if ((weekdays(NewData[i,2])) == "Saturday") {
                NewData[i,4] <- "weekend"
        } 
}
NewData$daytype <- as.factor(NewData$daytype)
```

#### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data. 


```r
interval_daytype <- as.data.frame(with(NewData, tapply(steps, list(interval, daytype), mean)))
par(mfrow=c(2,1), mar=c(0,2,0,2), oma=c(1,2,2,2))
plot(x=rownames(interval_daytype), y=interval_daytype$weekday, type="l",xaxt='n', ylab = '')
legend("topleft","weekday",bty = "n", cex = 1, text.font = 2)
par(mar=c(2,2,0,2))
plot(x=rownames(interval_daytype), y=interval_daytype$weekend, type="l", xlab = "", ylab = "")
mtext("Number of steps", side = 2, line = 1, outer = TRUE, cex = 1, font = 2 )
mtext("Interval", side = 1, line = 0, outer = TRUE, cex = 1, font = 2 )
legend("topleft","weekend",bty = "n", cex = 1, text.font = 2)
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

