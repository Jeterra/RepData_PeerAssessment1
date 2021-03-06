# Reproducible Research P1
José Eugenio Terra  
28 de outubro de 2017  



## Summary of Dataset

```r
activity <- read.csv("activity.csv")
summary(activity)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

## Question 1 - What is mean total number of steps taken per day?


```r
steps_x_Day <- aggregate(steps ~ date, activity, sum, na.rm=TRUE)
hist(steps_x_Day$steps,main = "Total number of steps taken per day", xlab = "Total steps taken per day", col = "green",ylim = c(0,30))
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

Mean of the total number of steps taken per day:

```r
mean(steps_x_Day$steps)
```

```
## [1] 10766.19
```

Median of the total number of steps taken per day:

```r
median(steps_x_Day$steps)
```

```
## [1] 10765
```

## Question 2 - What is the average daily activity pattern?

Make a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)



```r
steps_X_Interval<-aggregate(steps~interval, data=activity, mean, na.rm=TRUE)
plot(steps~interval, data=steps_X_Interval, type="l",col="green", lwd = 2, xlab="Interval", ylab="Average number of steps", main="Average number of steps per intervals")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->


Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
steps_X_Interval[which.max(steps_X_Interval$steps), ]$interval
```

```
## [1] 835
```

## 3. Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```


2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
getMeanSteps_x_Interval <- function(interval){
    steps_X_Interval[steps_X_Interval$interval==interval, ]$steps
}
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
activityNew<-activity
for(i in 1:nrow(activityNew)){
    if(is.na(activityNew[i,]$steps)){
        activityNew[i,]$steps <- getMeanSteps_x_Interval(activityNew[i,]$interval)
    }
}
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
totalSteps_x_DayNew <- aggregate(steps ~ date, data=activityNew, sum)
hist(totalSteps_x_DayNew$steps, col = "darkblue", xlab = "Total steps per day", ylim = c(0,30), main = "Total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

The mean didn’t change after the replacements of NAs.


```r
mean(totalSteps_x_DayNew$steps)
```

```
## [1] 10766.19
```

The median changes a little and become equal to the mean.


```r
median(totalSteps_x_DayNew$steps)
```

```
## [1] 10766.19
```

##4. Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
activity$date <- as.Date(strptime(activity$date, format="%Y-%m-%d"))
activity$datetype <- sapply(activity$date, function(x) {
        if (weekdays(x) == "sábado" | weekdays(x) =="domingo") 
                {y <- "Weekend"} else 
                {y <- "Weekday"}
                y
        })
```

Make a panel plot containing a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
activity_by_date <- aggregate(steps~interval + datetype, activity, mean, na.rm = TRUE)


plot<- ggplot(activity_by_date, aes(x = interval , y = steps, color = datetype)) +
       geom_line() +
       labs(title = "Average daily steps by type of date", x = "Interval", y = "Average number of steps") +
       facet_wrap(~datetype, ncol = 1, nrow=2)
print(plot)
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png)<!-- -->


