**Loading all the packages used**
---------------------------------

    library(dplyr)
    library(knitr)
    library(ggplot2)
    library(Hmisc)

**Loading and preprocessing the data**
======================================

**Loading**
-----------

### Reading the file **activity.csv** into *activity\_data*

    activity_data <- read.csv('./Fitbit/activity.csv')

**Processing**
--------------

### Removing NAs using *na.omit()* and storing in *activity\_data\_na\_omitted*

    activity_data_na_omitted <- na.omit(activity_data)

**Q. What is mean total number of steps taken per day?**
========================================================

**Total number of steps taken per day**
---------------------------------------

    daily_steps <- group_by(activity_data_na_omitted,date) %>% summarise(Total.Steps = sum(steps))
    head(daily_steps)

    ## # A tibble: 6 x 2
    ##   date       Total.Steps
    ##   <fct>            <int>
    ## 1 2012-10-02         126
    ## 2 2012-10-03       11352
    ## 3 2012-10-04       12116
    ## 4 2012-10-05       13294
    ## 5 2012-10-06       15420
    ## 6 2012-10-07       11015

**Histogram of the total number of steps taken each day**
---------------------------------------------------------

    hist(daily_steps$Total.Steps,main="Total number of steps per day",xlab="Total number of steps in a day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-4-1.png)

**Mean and Median of total steps per day**
------------------------------------------

### Mean

    mean(daily_steps$Total.Steps)

    ## [1] 10766.19

### Median

    median(daily_steps$Total.Steps)

    ## [1] 10765

**Q. What is the average daily activity pattern?**
==================================================

**Average number of steps taken vs Interval**
---------------------------------------------

    step_interval <- aggregate(steps ~ interval, activity_data_na_omitted, mean)
    plot(step_interval$interval, step_interval$steps, type='l', 
         main="Average number of steps over all days", xlab="Interval", 
         ylab="Average number of steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

**Interval with maximum number of steps**
-----------------------------------------

    step_interval[which.max(step_interval$steps),]

    ##     interval    steps
    ## 104      835 206.1698

**Imputing missing values**
===========================

**Total number of missing values in the dataset**
-------------------------------------------------

    sum(is.na(activity_data))

    ## [1] 2304

**Imputing steps using impute with mean**
-----------------------------------------

    activity_data$Imputed.Steps <- with(activity_data,impute(steps,mean))

**Creating dataset *imputed\_daily\_steps* **
---------------------------------------------

    imputed_steps <- transform(activity_data,steps=Imputed.Steps)

**Histogram of the total number of steps taken each day**
---------------------------------------------------------

    imputed_daily_steps <- aggregate(steps ~ date, imputed_steps, sum)
    hist(imputed_daily_steps$steps, main="Total number of steps per day (imputed)", 
         xlab="Total number of steps in a day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-12-1.png)

**Mean and Median of total steps per day compared**
---------------------------------------------------

### Mean *Imputed*

    mean(imputed_daily_steps$steps)

    ## [1] 10766.19

### Mean

    mean(daily_steps$Total.Steps)

    ## [1] 10766.19

### Median *Imputed*

    median(imputed_daily_steps$steps)

    ## [1] 10766.19

### Median

    median(daily_steps$Total.Steps)

    ## [1] 10765

**Q. Are there differences in activity patterns between weekdays and weekends?**
================================================================================

**Creating a new factor variable in the dataset with two levels - "weekday" and "weekend"**
-------------------------------------------------------------------------------------------

    imputed_steps['Day.Type'] <- weekdays(as.Date(imputed_steps$date))
    imputed_steps$Day.Type[imputed_steps$Day.Type %in% c('Saturday','Sunday') ] <- "Weekend"
    imputed_steps$Day.Type[imputed_steps$Day.Type != "Weekend"] <- "Weekday"
    imputed_steps$Day.Type <- as.factor(imputed_steps$Day.Type)
    head(imputed_steps)

    ##     steps       date interval Imputed.Steps Day.Type
    ## 1 37.3826 2012-10-01        0       37.3826  Weekday
    ## 2 37.3826 2012-10-01        5       37.3826  Weekday
    ## 3 37.3826 2012-10-01       10       37.3826  Weekday
    ## 4 37.3826 2012-10-01       15       37.3826  Weekday
    ## 5 37.3826 2012-10-01       20       37.3826  Weekday
    ## 6 37.3826 2012-10-01       25       37.3826  Weekday

**Plotting Steps vs Interval for Weekends and Weekdays**
--------------------------------------------------------

    imputed_steps_interval <- aggregate(steps ~ interval + Day.Type, imputed_steps, mean)
    qplot(interval,steps,data=imputed_steps_interval,type = 'l',geom=c("line"),xlab = "Interval",ylab = "Number of steps",main = "") +facet_wrap(~ Day.Type, ncol = 1)

    ## Warning: Ignoring unknown parameters: type

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-18-1.png)
