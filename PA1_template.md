---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
author: "JMTX"
date: "18 juillet 2020"
---



## Loading and preprocessing the data

```r
#load the library
library(dplyr)
library(lubridate)
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.6.3
```
<br>

1. **Load the data (i.e.read.csv())**

```r
#knitr::opts_knit$get("base.dir")
#knitr::opts_knit$get("base.url")
#knitr::opts_chunk$get("fig.path")
#-----------------------------------------------------------------------------------------------
# Step 1: Load the data
#
#Note: fill the "project_dir" variable with path of your unziped project repository 
#-----------------------------------------------------------------------------------------------

#go to project repository
project_dir="."
setwd(project_dir)

#read the data
data_activity<- read.csv("activity.csv")
```
<br>

2. **Process/transform the data (if necessary) into a format suitable for your analysis**

0 data process/transform was used here. (Maybe a conversion of date variable into date format could be interesting).

<br><br>

## What is mean total number of steps taken per day?
<br>

1. **Calculate the total number of steps taken per day**


```r
#-----------------------------------------------------------------------------------------------
# Step 2: plot the Histogram of the total number of steps taken each day
#
#-----------------------------------------------------------------------------------------------
#group data by date and compute the sum of the steps for each day
df_sum_steps<-data_activity%>%group_by(date)%>%summarize(sum_steps=sum(steps,na.rm=TRUE))
```
  
<br>

2. **Make a histogram of the total number of steps taken each day**


```r
#plot the Histogram of the total number of steps taken each day
#with( df_sum_steps, barplot(sum_steps~date, main="Histogram of the total number of steps taken each day", ylab="number of steps"))
with( df_sum_steps, hist(sum_steps,,breaks=10,main="Histogram of the total number of steps taken each day", xlab="number of steps per day"))
```

![](figure/unnamed-chunk-3-1.png)<!-- -->
<br>

3. **Calculate and report the mean and median of the total number of steps taken per day**


```r
#------------------------------------------------------------------------------------------------
# Step 3: Calculate and report the mean and median total number of steps taken per day 
#------------------------------------------------------------------------------------------------
df_mm_day<-df_sum_steps%>%summarize(mean_step = mean(sum_steps,na.rm=TRUE),median_step=median(sum_steps,na.rm=TRUE))
```
The **mean** number of steps taken per day is 9354 and the **median** number of steps taken per day is 10395.  

<br><br>

## What is the average daily activity pattern?
<br>

1.  **Make a time series plot (i.e. \color{red}{\verb|type = "l"|}type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)**


```r
#------------------------------------------------------------------------------------------------
# Step4: Time series plot of the average number of steps taken
#-------------------------------------------------------------------------------------------------
df_average_day_activity<-data_activity%>%group_by(interval)%>%summarize(mean_activity=mean(steps,na.rm=TRUE))
with(df_average_day_activity,plot(mean_activity~interval,type="l",main="Average daily activity pattern"))

#------------------------------------------------------------------------------------------------------
# Step 5: The 5-minute interval that, on average, contains the maximum number of steps
#------------------------------------------------------------------------------------------------------

max_interval<-df_average_day_activity[which.max(df_average_day_activity$mean_activity),1]
abline(v=max_interval,col="red")
#axis(side=1,at=max_interval,col="red",col.lab="red")
mtext(max_interval,at = max_interval,col="red",side=1 )
```

![](figure/unnamed-chunk-5-1.png)<!-- -->
<br>

2. **Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?**  

According to the previous graphic, the 5-minute interval, which on average across all the days in the dataset contains the maximum number of steps is the 835th.  
<br><br>

## Imputing missing values
<br>

1. **Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)**


```r
#--------------------------------------------------------------------------------------------------------
# Step 6: Code to describe and show a strategy for imputing missing data
#--------------------------------------------------------------------------------------------------------
NA_number<-sum(is.na(data_activity$steps))
```
The number of rows which contains NA values is **2304**.

<br>

2. **Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.**  

The strategy chosen to fill missing steps values is to replace NA with the average value of steps for this interval 
computed in the average daily pattern (variable: df_average_day_activity).

<br><br>

3. **Create a new dataset that is equal to the original dataset but with the missing data filled in.**


```r
NA_rows <- is.na(data_activity)
#copy the original data 
data_activity2<-data_activity
#The strategy chosen to fill missing steps values is to replace NA with the average value of steps
#for this interval computed in the average daily pattern (variable: df_average_day_activity).
data_activity2[NA_rows,1] <- df_average_day_activity[match(data_activity2[NA_rows,3],
                            df_average_day_activity$interval),2]
```
<br>

4. **Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?**


```r
#------------------------------------------------------------------------------------------------
# Step 7: Histogram of the total number of steps taken each day after missing values are imputed
#------------------------------------------------------------------------------------------------
#group data by date and compute the sum of the steps for each day
df_sum_steps2<-data_activity2%>%group_by(date)%>%summarize(sum_steps=sum(steps,na.rm=TRUE))

#plot the Histogram of the total number of steps taken each day
#with( df_sum_steps2, barplot(sum_steps~date, main="Histogram of the total number of steps taken each day", ylab="number of steps"))
with( df_sum_steps2, hist(sum_steps,,breaks=10,main="Histogram of the total number of steps taken each day", xlab="number of steps per day"))
```

![](figure/unnamed-chunk-8-1.png)<!-- -->

```r
#Calculate and report the mean and median total number of steps taken per day 
#------------------------------------------------------------------------------------------------
df_mm_day2<-df_sum_steps2%>%summarize(mean_step = mean(sum_steps,na.rm=TRUE),median_step=median(sum_steps,na.rm=TRUE))
```
Without missing values, the **mean** number of steps taken per day is 1.0766\times 10^{4} and the **median** number of steps taken per day is 1.0766189\times 10^{4}.
Both values have increased with our strategy to fill missing values.

<br><br>

## Are there differences in activity patterns between weekdays and weekends?
<br>

1. **Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.**


```r
#------------------------------------------------------------------------------------------------
# Step 8: Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
#---------------------------------------------------------------------------------------------------
#Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given 
#date is a weekday or weekend day
df2<-data_activity2
df2<-mutate(df2,weekdays = as.factor(sapply(as.POSIXlt(data_activity2$date)$wday,
                               function(x) if (x==6 | x==0){"weekend"} else {"weekday"})))
```

<br>

2. ***Make a panel plot containing a time series plot (i.e. \color{red}{\verb|type = "l"|}type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.***


```r
#group by weekdays and interval then compute the average activity pattern for both type of days
df_average<-df2%>%group_by(weekdays,interval)%>%summarize(mean_activity=mean(steps,na.rm=TRUE))

#plot the difference dayly pattern for weekend and weekday using ggplot2
g<-ggplot(df_average,aes(interval,mean_activity))
g + geom_line() + facet_wrap(weekdays~.,nrow=2,ncol=1, strip.position="top")+ylab("Number of steps")#+  facet_grid(weekdays~.,switch="both")+facet_wrap(facets, strip.position="right")
```

![](figure/unnamed-chunk-10-1.png)<!-- -->

We can observe small differences between both patterns.
