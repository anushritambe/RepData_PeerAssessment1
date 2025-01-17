
**Reproducible Research: Peer Assessment 1**
  
======================================================

## Loading and preprocessing the data

```r
activity<-read.csv("activity.csv")
activity$date<-as.Date(activity$date,"%Y-%m-%d")
```

## What is mean total number of steps taken per day?

```r
library(dplyr)
library(ggplot2)
activity1<- activity %>% group_by(date) %>% summarise(stepsum=sum(steps))
```
Histogram of Steps per day

```r
g<- ggplot(activity1,aes(stepsum))
g+geom_histogram(fill="blue")+ggtitle("Total no. of Steps per Day")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 8 rows containing non-finite values (stat_bin).
```

![plot of chunk histogram](figure/histogram-1.png)

```r
dev.copy(png, file = "plot1.png")
```

```
## png 
##   5
```

```r
dev.off()
```

```
## RStudioGD 
##         2
```
Mean and Median total number of steps taken per day

```r
meansteps<- mean(activity1$stepsum,na.rm=TRUE)
mediansteps<- median(activity1$stepsum,na.rm=TRUE)
print(meansteps)
```

```
## [1] 10766.19
```

```r
print(mediansteps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

```r
steps_interval<- activity %>% group_by(interval) %>% summarise(stepsum=mean(steps,na.rm=TRUE))
plot(steps_interval$interval,steps_interval$stepsum,xlab="Interval",ylab="Steps",main="Average Daily Activity Pattern",type="l")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)

```r
dev.copy(png, file = "plot2.png")
```

```
## png 
##   5
```

```r
dev.off()
```

```
## RStudioGD 
##         2
```

The maximum number of steps and its corresponding interval.

```r
maxsteps<-steps_interval%>%filter(stepsum==max(stepsum))
print(maxsteps)
```

```
## # A tibble: 1 x 2
##   interval stepsum
##      <int>   <dbl>
## 1      835    206.
```


## Imputing missing values
To count total NA values in steps

```r
NA_values<-sum(is.na(activity$steps))
print(NA_values)
```

```
## [1] 2304
```
Filling missing data with the mean number of steps across all days with available data for that particular interval.

```r
actmerge<-merge(activity,steps_interval,by="interval") 
for(i in 1:length(actmerge$steps)){
  if(is.na(actmerge$steps[i])){
    actmerge$steps[i]=actmerge$stepsum[i]
  }
    else{
      actmerge$steps[i]
      }
}
newactivity<-actmerge%>%select(-stepsum)
```
Total number of steps per day.

```r
newactivity1<- newactivity %>% group_by(date) %>% summarise(newstepsum=sum(steps))
g<- ggplot(newactivity1,aes(newstepsum))
g+geom_histogram(fill="green")+ggtitle("Total no. of Steps per Day after imputing NA")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk afterimputing](figure/afterimputing-1.png)

```r
dev.copy(png, file = "plot3.png")
```

```
## png 
##   5
```

```r
dev.off()
```

```
## RStudioGD 
##         2
```

New mean and meadian after imputing NA values

```r
newmeansteps<- mean(newactivity1$newstepsum,na.rm=TRUE)
newmediansteps<- median(newactivity1$newstepsum,na.rm=TRUE)
print(newmeansteps)
```

```
## [1] 10766.19
```

```r
print(newmediansteps)
```

```
## [1] 10766.19
```
The mean remains the same as before imputation, while the median value increased slightly.

## Are there differences in activity patterns between weekdays and weekends?
Creating a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
actmerge1<-newactivity
for(i in 1:length(actmerge1$date)){
  if(weekdays(as.Date(actmerge1$date[i]))=="Saturday"|weekdays(as.Date(actmerge1$date[i]))=="Sunday"){
    actmerge1$day[i]="weekend"
  }
  else{
    actmerge1$day[i]="weekday"
  }
}
```
Panel plot containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days 

```r
weekdata<-actmerge1 %>% group_by(day,interval)%>% summarise(stepmean=mean(steps))
ggplot(weekdata,aes (interval, stepmean)) + geom_line() +facet_wrap(day~.,nrow=2,ncol=1)+ggtitle("Mean Steps by Interval depending on Day")
```

![plot of chunk plot](figure/plot-1.png)

```r
dev.copy(png,file ="plot4.png")
```

```
## png 
##   5
```

```r
dev.off()
```

```
## RStudioGD 
##         2
```


