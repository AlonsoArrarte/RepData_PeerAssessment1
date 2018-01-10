RepData peer Assesssment 1
==========================

Loading the data
----------------

-   Loading and preprocessing the data

<!-- -->

    activity=read.csv("activity.csv")
    stepsPerDay<-aggregate(steps~date,data=activity,FUN=sum,na.rm=TRUE)

What is mean total number of steps taken per day?
-------------------------------------------------

-   Make a histogram of the total number of steps taken each day

<!-- -->

    hist(stepsPerDay$steps,main = "Histogram of Steps per day",
         col="gray",  xlab="Total steps per day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)

-   Calculate and report the **mean** and **median** total number of
    steps taken per day

<!-- -->

    mean(stepsPerDay$steps)

    ## [1] 10766.19

    median(stepsPerDay$steps)

    ## [1] 10765

-   The **mean** total number of steps taken per day = 10766.19 steps.
-   The **median** total number of steps taken per day = 10765 steps.

What is the average daily activity pattern?
-------------------------------------------

-   Make a time series plot (i.e. type = "l") of the 5-minute
    interval (x-axis) and the average number of steps taken, averaged
    across all days (y-axis)

<!-- -->

    stepsPerInterval<-aggregate(steps~interval,data=activity,mean,na.rm=TRUE)
    plot(steps~interval,data=stepsPerInterval,type="l")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-4-1.png)

-   Which 5-minute interval, on average across all the days in the
    dataset, contains the maximum number of steps?

<!-- -->

    stepsPerInterval[which.max(stepsPerInterval$steps),]$interval

    ## [1] 835

Imputing missing values
-----------------------

-   Calculate and report the total number of missing values in the
    dataset (i.e. the total number of rows with NAs)

<!-- -->

    sum(is.na(activity$steps))

    ## [1] 2304

-   Devise a strategy for filling in all of the missing values in
    the dataset. The strategy does not need to be sophisticated. For
    example, you could use the mean/median for that day, or the mean for
    that 5-minute interval, etc.

I calculated the 'mean' number of steps for of each interval that
contained missing steps values. Next I appended the 'mean' number of
steps to each interval that was missing a steps value. Rows with a steps
value of 0 were not treated as missing values.

-   Create a new dataset that is equal to the original dataset but with
    the missing data filled in.

<!-- -->

    mean_missing_steps <- merge(subset(activity,is.na(activity$steps)),stepsPerInterval, by=c("interval"))
    activity_fill <- merge(mean_missing_steps,activity, by=c("interval","date"), all=TRUE)
    activity_fill[ is.na(activity_fill$steps) , "steps" ] <- activity_fill[is.na(activity_fill$steps) , "steps.y" ]
    activity_fill <-subset(activity_fill, select=c ("interval","date","steps"))

-   Make a histogram of the total number of steps taken each day and
    Calculate and report the mean and median total number of steps taken
    per day.

*Blue bars indicate data without missing values, Red bars indicate data
where missing values have been replaced with mean value for intervals
missing values.*

    stepsPerDay_filled<-aggregate(steps~date,data=activity_fill,sum)


    hist(stepsPerDay_filled$steps,main = "Histogram of Steps per day (overlay)",
         col="red",  xlab="Total steps per day")
    hist(stepsPerDay$steps, col="blue",add=T  )
    box()

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-8-1.png)

    mean(stepsPerDay_filled$steps)

    ## [1] 10766.19

    median(stepsPerDay_filled$steps)

    ## [1] 10766.19

The **mean** total number of steps taken per day is 10766.19 steps. The
**median** total number of steps taken per day is 10766.19 steps.

-   Do these values differ from the estimates from the first part of the
    assignment? What is the impact of imputing missing data on the
    estimates of the total daily number of steps?

The **mean** value is now the same as the **median** value, this makes
sense as we replaced the missing values with the mean value. We now have
more data falling into the largest mean band as shown by the overlaid
histogram.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

-   Create a new factor variable in the dataset with two levels –
    “weekday” and “weekend” indicating whether a given date is a weekday
    or weekend day.

<!-- -->

    # add name of weekday
    activity_fill$day=weekdays(as.Date(activity_fill$date))
    # classify weekday as 'weekend' or 'weekday'
    activity_fill<- within(activity_fill, {
        day_type = ifelse(day %in% c("Saturday","Sunday"), "weekend", "weekday")
     })

-   Make a panel plot containing a time series plot (i.e. type = "l") of
    the 5-minute interval (x-axis) and the average number of steps
    taken, averaged across all weekday days or weekend days (y-axis).
    The plot should look something like the following, which was
    creating using simulated data:

<!-- -->

    stepsInterval=aggregate(steps~interval+day_type,activity_fill,mean)
    library(lattice)
    xyplot(steps~interval|factor(day_type),data=stepsInterval,aspect=1/2,type="l")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-10-1.png)
