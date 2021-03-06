Reproducible Research: Peer Assessment 1
========================================

# Loading and preprocessing the data

Show any code that is needed to

* Load the data (i.e. read.csv())
* Process/transform the data (if necessary) into a format suitable for your analysis


```r
library("plyr")
library("dplyr")
```

```
## 
## Attaching package: 'dplyr'
## 
## The following objects are masked from 'package:plyr':
## 
##     arrange, count, desc, failwith, id, mutate, rename, summarise,
##     summarize
## 
## The following object is masked from 'package:stats':
## 
##     filter
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
unzip("./activity.zip", "activity.csv")
activity <- read.csv("./activity.csv")
```

# What is mean total number of steps taken per day?

##For this part of the assignment, you can ignore the missing values in the dataset.

* Calculate the total number of steps taken per day

* If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day


```r
act1 <- ddply(activity, ~interval, transform, sum = sum(steps, na.rm=TRUE))
hist(act1$sum, 
     main="Histogram of Sum of Steps Per Day", 
     xlab="Steps Per Day")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

* Calculate and report the mean and median of the total number of steps taken per day


```r
mean1 <- mean(act1$sum); mean1
```

```
## [1] 1981.278
```

```r
median1 <- median(act1$sum); median1
```

```
## [1] 1808
```

# What is the average daily activity pattern?

* Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
act2 <- summarise(group_by(activity, interval), mean = mean(steps, na.rm=TRUE))
plot(act2$interval, 
     act2$mean, 
     type="l", 
     xlab="Interval", 
     ylab="Mean Number of Steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

* Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
act2[which.max(act2$mean),]
```

```
## Source: local data frame [1 x 2]
## 
##   interval     mean
## 1      835 206.1698
```

# Imputing missing values

## Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.
* Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

* Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
## My strategy is to use the mean value of the interval across all days to fill in NA values.
impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm=TRUE))
```

* Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
act3 <- ddply(activity, ~interval, transform, steps = impute.mean(steps))
## using arrange here because ddply sorts the data.  Probably not strictly necessary, but it makes me happy.
act3 <- arrange(act3, date, interval)
```

* Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
act3A <- ddply(act3, "date", summarise, sum=sum(steps))
hist(act3A$sum, 
     main="Histogram of Sum of Steps Per Day", 
     xlab="Steps Per Day")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 

```r
mean(act3A$sum); mean1
```

```
## [1] 10766.19
```

```
## [1] 1981.278
```

```r
median(act3A$sum); median1
```

```
## [1] 10766.19
```

```
## [1] 1808
```

```r
## By removing a good number of the 0 values of the data, the mean and median are much greater.
```

# Are there differences in activity patterns between weekdays and weekends?

## For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

* Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
act4 <- mutate(act3, dow = weekdays(as.POSIXlt(date)))
act4$dow <- mapvalues(act4$dow, 
                      from=c("Sunday", "Saturday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday"), 
                      to=c("weekend", "weekend", "weekday", "weekday", "weekday", "weekday", "weekday"))
act4$dow = as.factor(act4$dow)
```

* Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
plotdata <- ddply(act4, c("dow", "interval"), summarise, mean=mean(steps))
library("ggplot2")
qplot(interval, mean, data=plotdata, facets = dow~., geom="path")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) 
