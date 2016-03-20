# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

# downloading and unziping the csv file
    
    ```r
        if (file.exists("activity.csv")==FALSE)
          {
          temp <- tempfile()
          URL <- c("https://github.com/talkalovski/RepData_PeerAssessment1/blob/ea9cc6573bd7e4212a709c1168b573a42a1726ac/activity.zip?raw=true.zip")
          download.file(URL,temp, mode="wb")
          unzip(temp, "activity.csv")
          }
    ```
# reading and processing the data
    
    
    ```r
        act <- read.csv("activity.csv")
        act$date <- as.Date(act$date, format = "%Y-%m-%d")
    ```

## What is mean total number of steps taken per day?

# Calculate the total number of steps taken per day
    
    
    ```r
      sumday <- setNames(aggregate(act$steps~act$date, FUN=sum),c("date","sum_steps"))
    ```
    
# Make a histogram of the total number of steps taken each day
    
    
    ```r
    hist(sumday$sum_steps,breaks = 8)
    ```
    
    ![](PA1_Template_files/figure-html/unnamed-chunk-4-1.png)
# Calculate and report the mean and median of the total number of steps taken per day

    
    ```r
    mean(sumday$sum_steps)
    ```
    
    ```
    ## [1] 10766.19
    ```
    
    ```r
    median(sumday$sum_steps)
    ```
    
    ```
    ## [1] 10765
    ```
    
## What is the average daily activity pattern?

# Calculate the average number of steps taken per interval across all days.

    
    ```r
      intmean <- setNames(aggregate(act$steps~act$interval, FUN=mean),c("interval","steps_mean"))
    ```
    
# plot the average daily activity - number of steps taken per interval.
    
    
    ```r
    library("ggplot2")
    ggplot(intmean, aes(interval, steps_mean)) + geom_line()+
    xlab("Daily 5min intervals")+ylab("Average # of steps")
    ```
    
    ![](PA1_Template_files/figure-html/unnamed-chunk-7-1.png)
    
# calculate the 5-minute interval that contains the maximum number of steps on average.

    
    ```r
    subset(intmean, steps_mean==max(steps_mean))
    ```
    
    ```
    ##     interval steps_mean
    ## 104      835   206.1698
    ```
## Imputing missing values

# clculating the number of NA's in thedataset (missing values)
    
    
    ```r
    sum(is.na(act$steps))
    ```
    
    ```
    ## [1] 2304
    ```
# a strategy for filling in all of the missing values in the dataset.
I think the best way is to impute the mean value of the specific interval across all other days.

# Create a new dataset that is equal to the original dataset but with the missing data filled in.

    
    ```r
    fillact <- act
    fillact$Msteps <- intmean$steps_mean
    
    for (i in 1:nrow(fillact))
      if (is.na(fillact$steps[i])) 
        fillact$steps[i] = fillact$Msteps[i] 
    ```
    
# Make a histogram of the total number of steps taken each day.
    
    
    ```r
      fsumday <- setNames(aggregate(fillact$steps~fillact$date, FUN=sum),c("date","sum_steps"))
      hist(fsumday$sum_steps,breaks = 8)
    ```
    
    ![](PA1_Template_files/figure-html/unnamed-chunk-11-1.png)
    
# Calculate and report the mean and median of the total number of steps taken per day
    
    
    ```r
      mean(fsumday$sum_steps)
    ```
    
    ```
    ## [1] 10766.19
    ```
    
    ```r
      median(fsumday$sum_steps)
    ```
    
    ```
    ## [1] 10766.19
    ```
    
## Are there differences in activity patterns between weekdays and weekends?

# Create a new factor variable in the dataset with two levels - "weekday" and "weekend.
    
    
    ```r
        fillact$date <- as.Date(act$date, format = "%Y-%m-%d")
        fillact$weekday <- weekdays(fillact$date)
        
       
         for (i in 1:nrow(fillact))
            if  (fillact$weekday[i]=="Friday"|fillact$weekday[i]=="Saturday") 
            fillact$weekpart[i] ="Weekend" else fillact$weekpart[i] ="Weekday"
    ```

# Make time serise plot to compare the average dayly actiity between weekdays and weekend.

    
    ```r
        wd <- fillact [fillact$weekpart=="Weekday",]
        we <- fillact [fillact$weekpart=="Weekend",]
        
        
        wdintmean <- setNames(aggregate(wd$steps~wd$interval, FUN=mean),c("interval","steps_mean"))
        weintmean <- setNames(aggregate(we$steps~we$interval, FUN=mean),c("interval","steps_mean"))
        
        ggplot(wdintmean,aes(interval, steps_mean))+geom_line(aes(color="Weekdays"),size=1)+
          geom_line(data=weintmean,aes(color="Weekend"),size=1,alpha=3/4)+
          labs(color="Legend text")+   xlab("Daily 5min intervals") + ylab("Average # of steps")
    ```
    
    ![](PA1_Template_files/figure-html/unnamed-chunk-14-1.png)
    
I think the comparison shows that this person tesnd to sleep late in the weekend, but then on during the day he is more active.
