# Reproducible Research: Peer Assessment 1

```r
  options(warn=-1)
  library(ggplot2)
  library(plyr)
  options(warn=0)
```

## Loading and preprocessing the data
The following code was used to loade the data:


```r
  setwd("C:/Users/elarsen/Coursera work")
  data<-read.csv("reproducible research/activity.csv")
  
  ##get the set of dates and intervals measured
  
  dates<-unique(data[,2])
  intervals<-unique(data[,3]) 
  
  ## Calculated totals by day and make a tidy table
  dtot<-tapply(data$steps,data$date,sum)
  dmean<-tapply(data$steps,data$date,mean, na.rm=TRUE)
  dmedian<-tapply(data$steps,data$date,median, na.rm=TRUE)
  dailydata<-data.frame(dates, dtot, dmean, dmedian, na.rm=TRUE)
```


## What is mean total number of steps taken per day?



```r
  qplot(dtot, binwidth=800, main="Total Steps per Day Distribution", xlab="Total Steps")
```

![](./PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

```r
  summary(dmean)[3:5]
```

```
##  Median    Mean 3rd Qu. 
##   37.38   37.38   46.16
```


```r
## Calculated totals by interval and make a tidy table
  itot<-tapply(data$steps,data$interval,sum, na.rm=TRUE)
  imean<-tapply(data$steps,data$interval,mean, na.rm=TRUE)
  imedian<-tapply(data$steps,data$interval,median, na.rm=TRUE)
  intdata<-data.frame(intervals, itot, imean, imedian)
```


The highest number of steps on average occur during the following interval:


```r
## order by the average steps per interval desc and display the interval
  intdata<-arrange(intdata, desc(imean))
  
  intdata[1,1]
```

```
## [1] 835
```



## What is the average daily activity pattern?

```r
  g<-ggplot(data, aes(x=interval,y=steps))
  g<- g+stat_summary(fun.y="sum", geom="line", size=1,color="limegreen")
  g<- g+labs(title="Activity Timeline")
  g<- g+labs(y="Number of Steps")
  print(g)
```

```
## Warning: Removed 2304 rows containing missing values (stat_summary).
```

![](./PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

A significant number of values were missing in the data set:

```r
  ## Count the NAs
  totna<-sum(is.na(data$steps))
  
  print(totna)
```

```
## [1] 2304
```

The missing data was filled by taking the average steps for the same interval on other days.  As shown below this had minimal overall impact on averages, but may affect the weekend/weekday pattern as some days were entirely NA (such as the first day) and this method will make them entirely "average".  Still this seemed less disruptive than implanting non time of day sensitive estimates.


```r
## Imputing missing values
## replaces NAs with average value for that time interval from other days.
data2<-data
for (i in 1:dim(data)[1]){
 if(is.na(data[i,1])){
    data2[i,1]<-subset(intdata, intervals==data[i,3], select=imean)
  }
}

## Calculated totals by day and make a tidy table
  dtot2<-tapply(data2$steps,data2$date,sum)
  dmean2<-tapply(data2$steps,data2$date,mean, na.rm=TRUE)
  dmedian2<-tapply(data2$steps,data2$date,median, na.rm=TRUE)

  dailydata2<-data.frame(dates, dtot2, dmean2, dmedian2, na.rm=TRUE)
```

The modified histogram was minimally impacted.


```r
  qplot(dtot2, binwidth=800, main="Total Steps per Day Distribution\n (Data holes filled)", xlab="")
```

![](./PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

With the method of filling in the missing data the averages are not impacted, though the quartiles are. The unmodifed and completed data profiels are below:


```r
  summary(dmean)[3:5]
```

```
##  Median    Mean 3rd Qu. 
##   37.38   37.38   46.16
```

```r
  summary(dmean2)[3:5]
```

```
##  Median    Mean 3rd Qu. 
##   37.38   37.38   44.48
```
  
## Are there differences in activity patterns between weekdays and weekends?

As shown in the plot below, the activity level on weekdays is substantially higher:


```r
day<-vector(length=dim(data)[1])
for (i in 1:dim(data)[1]){

  if(weekdays(as.Date(data2$date[i]))=="Saturday"){
    day[i]<-"weekend"
    }
  else if(weekdays(as.Date(data2$date[i]))=="Sunday"){
    day[i]<-"weekend"
    }
  
  else{
    day[i]<-"weekday"
    }
}
data3<-cbind(data2,day)

 g<-ggplot(data3, aes(x=interval,y=steps, day))
 ## g<- g+stat_summary(fun.y="mean", geom="point", size=3,color="red")
  g<- g+stat_summary(fun.y="sum", geom="line", size=1,color="limegreen")
  g<- g+labs(title="Weekday versus Weekend Activity \nby Time of Day")
  g<- g+labs(y="Number of Steps")
  g<- g+facet_grid(day~.)
  print(g)
```

![](./PA1_template_files/figure-html/unnamed-chunk-11-1.png) 


