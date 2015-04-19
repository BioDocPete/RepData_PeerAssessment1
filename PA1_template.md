# Reproducible Research: Peer Assessment 1

**Date:  **2015-04-19
<br />
_____________________________________________________________________________________________________________________________
_____________________________________________________________________________________________________________________________
<br />

#### Load packages

```r
library(timeDate); library(ggplot2); library(plyr)
```

<br />
_____________________________________________________________________________________________________________________________
<br />

#### Load and preprocess the data

```r
# load data
dat <- read.csv(file="activity.csv",head=TRUE, sep=",")
# remove rows with intervals having no measurements of steps (i.e., NA) ; but keeps nlevels of days (i.e., 61) even though a day such as 2012-10-01 was only NA. Measurements of steps was made on only 53 days 
noNA <- dat[!is.na(dat$steps),]
```
<br />
_____________________________________________________________________________________________________________________________
<br />

#### What is mean total number of steps taken per day (8 days with no measurements remain; i.e., denominator has 8 more days than it should; so steps per day is underestimated). 

```r
meanStepsPerDay_61days <- sum(noNA$steps)/nlevels(as.factor(dat$date))
meanStepsPerDay_61days <- round(meanStepsPerDay_61days, 0)
```
+ **Mean steps per day over 61 days =  **9354
<br />
_____________________________________________________________________________________________________________________________
<br />

#### Histogram of the total number of steps taken each day:

```r
# make df with 2 columns: date and total steps
TotalStepsPerDay <- aggregate(steps ~ date, dat, sum) # NAs removed
hist(TotalStepsPerDay$steps)
```

<img src="PA1_template_files/figure-html/unnamed-chunk-3-1.png" title="" alt="" style="display: block; margin: auto;" />
<br />
_____________________________________________________________________________________________________________________________
<br />

#### Mean and median of the total number of steps taken per day:

```r
mean <- as.integer(mean(TotalStepsPerDay$steps)) # 
median <- as.integer(median(TotalStepsPerDay$steps)) # 
```
+ **Mean total number of steps taken per day =  **10766
+ **Median total number of steps taken per day =  **10765
<br />
_____________________________________________________________________________________________________________________________
<br />

#### What is the average daily activity pattern? Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
interval <- ddply(dat, c("interval"), summarise,
                    N    = sum(!is.na(steps)),
                    tot = sum(steps, na.rm=TRUE),
                    mean = mean(steps, na.rm=TRUE),
                    median = median(steps, na.rm=TRUE),
                    sd   = sd(steps, na.rm=TRUE),
                    se   = sd / sqrt(N)
)
plot(interval$mean~ interval$interval, type="l", main="Steps Per Interval: Average of All Days")
```

<img src="PA1_template_files/figure-html/AveStepsPerInterval-1.png" title="" alt="" style="display: block; margin: auto;" />
<br />
_____________________________________________________________________________________________________________________________
<br />

#### Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
maxMeanInterval<-with(interval, interval[mean== max(mean)]) # interval with maximum steps
maxSteps<-ddply(interval, "mean", subset, mean==max(mean)) # steps for all intervals
mostStepsPerInterval<-maxSteps[nrow(maxSteps),]$tot
```

+ **The most steps per interval was** 10927**, which were in the** 835th **interval**. 
<br />
_____________________________________________________________________________________________________________________________
<br />

#### Imputing missing values
##### Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

##### Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
nrowNA <- nrow(dat[dat$steps==NA,])
```

+ **the total number of missing values in the dataset (i.e. the total number of rows with NAs) =** 17568

<br />
_____________________________________________________________________________________________________________________________
<br />

#### Devise a strategy for filling in all of the missing values in the dataset. I used the mean of the 5-minute interval to replace NAs, and created a new dataset equal to the original but with the missing data filled in.


```r
# Use the strategy of substituting NA steps with mean steps for the 5-minute interval 
# Make a df with mean steps per interval across all days
meanStepsPerInterval <- aggregate(dat$steps, list(interval = dat$interval), mean, na.rm=TRUE)
# Add meanStepsPerInterval column to the full dat. Note this column repeats for each day. 
dat$meanStepsPerInterval <- meanStepsPerInterval$x
# replace NAs with value from the MeanStepsPerInterval column
datInterpolated <- within(dat, steps <- ifelse(is.na(steps), meanStepsPerInterval, steps))
# Remove the repeated column of mean steps per interval accross days to "Create a new dataset that is equal to the original dataset but with the missing data filled in"
datInterpolated[4] <- NULL
```

<br />
_____________________________________________________________________________________________________________________________
<br />

#### Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

#### For the interpolated data, histogram of the total number of steps taken each day:

```r
# make df with 2 columns: date and total steps
TotalStepsPerDay2 <- aggregate(steps ~ date, datInterpolated, sum) # NAs removed
hist(TotalStepsPerDay2$steps)
```

<img src="PA1_template_files/figure-html/unnamed-chunk-8-1.png" title="" alt="" style="display: block; margin: auto;" />

#### For the interpolated data, mean and median of the total number of steps taken per day:

```r
mean2 <- as.integer(mean(TotalStepsPerDay2$steps)) # 
median2 <- as.integer(median(TotalStepsPerDay2$steps)) # 
```
+ **Mean total number of steps taken per day =  **10766
+ **Median total number of steps taken per day =  **10766  
The impact of imputing the missing values was small. Only the median changed, increasing by one step.

<br />
_____________________________________________________________________________________________________________________________
<br />

#### Are there differences in activity patterns between weekdays and weekends? For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
datInterpolated$weekday<-isWeekday(datInterpolated$date) # does the character need to be converted to dateTime?
datInterpolated$weekday<-as.factor(datInterpolated$weekday)
levels(datInterpolated$weekday) <- c("Weekend","Weekday")
weektime <- ddply(datInterpolated, c("interval", "weekday"), summarise,
                  N    = sum(!is.na(steps)),
                  tot = sum(steps, na.rm=TRUE),
                  mean = mean(steps, na.rm=TRUE),
                  median = median(steps, na.rm=TRUE),
                  sd   = sd(steps, na.rm=TRUE),
                  se   = sd / sqrt(N)
)

ggplot(weektime, aes(x=interval, y=mean, group=weekday)) + geom_line() + facet_wrap(~weekday, ncol=1) + ggtitle("Average Number of Steps Taken: Weekends vs. Weekdays\n")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-10-1.png" title="" alt="" style="display: block; margin: auto;" />

##### On the weekends, it looks like sleeping-in, greater activity, and staying up late. 



