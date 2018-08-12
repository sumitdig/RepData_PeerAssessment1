Reproducible Research : Week2 Assignment
========================================

Set the global options for R code here:

    knitr::opts_chunk$set(echo = TRUE)

1. Loading and preprocessing data
---------------------------------

1.a) Let's set the working directory for the assignment.

      setwd("~/Data_Science/Course5_ReproducibleResearch/Week2/Project")

1.b) Manmually download the repdata\_data\_activity.zip file form the
link given in the assignment. 1.c) Unzip the file.

    unzip("repdata_data_activity.zip")

1.d) Load the file into a data frame.

    activity <- read.csv("activity.csv")

1.e) Clean the data by removing NAs

    activityclean <- activity[is.na(activity$steps) ==0, ]

2. Mean total number of steps taken per day
-------------------------------------------

Now we will try to answer the question - What is mean total number of
steps taken per day?

2.a) First, we will calculate the total number of steps taken per day.

    totstepsbyday <- aggregate(activityclean$steps, by=list(activityclean$date), FUN=sum)
    colnames(totstepsbyday) <- c("date", "TotSteps")

2.b) Next, we will generate a Histogram of Total steps taken per day. We
will use ggplot2 package for this purpose. Therefore, let's load the
library.

    library(ggplot2)

Now that the library has been loaded, we can generate the plot.

    ggplot(totstepsbyday, aes(x=TotSteps)) + 
      geom_histogram(color="blue", fill="green", binwidth = 500) +
      labs(x="Total Steps taken by Day")

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

2.c) Lets calculate the Mean and Median of the total number of steps
taken per day.

    stepsbyday_mean1 <- mean(totstepsbyday$TotSteps)
    stepsbyday_median1 <- median(totstepsbyday$TotSteps)

***Mean for total number of steps taken per day is 1.076618910^{4} and
median is 10765.***

3. Average Daily Activity Pattern
---------------------------------

Now we will try to answer the question - What is the average daily
activity pattern?

3.a) First, we will calculate the average number of steps taken by
interval (averaged across all days).

    avgstepsbyinterval <- aggregate(activityclean$steps, by=list(activityclean$interval), FUN=mean)
    colnames(avgstepsbyinterval) <- c("interval", "AvgSteps")

3.b) Now we will create the time Series Plot.

    plot(AvgSteps~interval, avgstepsbyinterval, type="l", xlab="Time Interval", ylab="Average Steps Taken", axes=FALSE)
    axis(side=1, at=c(0, 100, 200, 300, 400, 500, 600, 700, 800, 900, 1000, 1100, 1200, 1300, 1400, 1500, 1600, 1700, 1800, 1900, 2000, 2100, 2200, 2300, 2400))
    axis(side=2, at=c(0, 50, 100, 150, 200, 250))

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-10-1.png)

3.c) Get the Time Interval with the highest Average Number of Steps.

    maxstepsbyinterval <- avgstepsbyinterval[which.max(avgstepsbyinterval$AvgSteps), 1]

***5-minute interval containing, on average, the maximum number of steps
across all the days is 835.***

4 Imputing missing values
-------------------------

4.a) First, we will get the total number of missing values.

    totNA <- sum(is.na(activity$steps))

***Total number of missing values in the dataset is 2304.***

4.b) For fixing the missing values we will replace the missing values
with the mean for that 5-minute interval.

4.c) Create a new dataset that is equal to the original dataset but with
the missing data filled in.

4.c.i) First, make a copy of the data frame.

    library(data.table)
    activitycomplete <- copy(activity)

4.c.ii) Next, find the indexes of the rows with the missing Step values.

    indxNA <- which(is.na(activity$steps), arr.ind=TRUE)

4.c.iii) Finally, replace the missing values with the mean of the
5-minute interval for that particular interval.

    for(i in indxNA)
    {
        activitycomplete[i, 1] <-avgstepsbyinterval[avgstepsbyinterval$interval == activity[i, 3],2]      
    }

4.d) Create a histogram and calculate mean and median of total number of
steps taken per day. 4.d.i) Total number of steps taken per day with
missing values imputed.

    totstepsbydayC <- aggregate(activitycomplete$steps, by=list(activitycomplete$date), FUN=sum)
    colnames(totstepsbydayC) <- c("date", "TotSteps")

4.d.ii) Generate Histogram of Total steps taken per day.

    ggplot(totstepsbydayC, aes(x=TotSteps)) + 
      geom_histogram(color="blue", fill="green", binwidth = 500) +
      labs(x="Total Steps taken by Day", title="Total steps taken per day (missing values imputed)")

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-17-1.png)

4.d.iii) Calculate Mean and Median

    stepsbyday_mean2 <- mean(totstepsbydayC$TotSteps)
    stepsbyday_median2 <- median(totstepsbydayC$TotSteps)

***After imputing the missing vales, the mean for total number of steps
taken per day is 1.076618910^{4} and median is 1.076618910^{4}.***

5 Find differences in activity patterns between weekdays and weekends
---------------------------------------------------------------------

5.a) Create a new factor variable for Weekend/Weekday.

5.a.i) Add Day of the Week column.

    activitycomplete$DOW = weekdays(as.Date(activitycomplete$date))

5.a.ii) Add a new column specifying Weekday OR Weekend.

    activitycomplete$DayType <- ifelse(activitycomplete$DOW == "Saturday" | activitycomplete$DOW == 
                                     "Sunday", "Weekend", "Weekday")

5.a.iii) Convert column to factor.

    activitycomplete$DayType <- factor(activitycomplete$DayType)

5.b) Make a Panel plot.

5.b.i) Calculate the average number of steps taken by interval and day
type (averaged across all days).

    avgstepsbyintervalC <- aggregate(steps~interval+DayType, activitycomplete, mean)

5.b.ii) Generate the plot.

    ggplot(avgstepsbyintervalC, aes(x=interval, y=steps)) + 
      geom_line(color="Blue") + facet_grid(DayType~.) +
      labs(x="Interval", y = "Number of Steps")

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-23-1.png)
