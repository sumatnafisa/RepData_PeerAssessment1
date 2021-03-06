Installing packages and libraries for the code.

```{r}
install.packages("reshape2", repos='http://cran.us.r-project.org')
install.packages("ggplot2", repos='http://cran.us.r-project.org')
library("reshape2")
library("ggplot2")
```

Download activity.csv file


```{r}
activity <- read.csv("C:/Users/Sumat/Desktop/Reproducible Research/activity.csv")
```

What is mean total number of steps taken per day?
```{r}
meltedActivity <- melt(activity, id=c("date"), na.rm=TRUE, measure.vars="steps")
castedActivity <- dcast(meltedActivity, date ~ variable, sum)
hist(castedActivity$steps)
actMean <- format(round(mean(castedActivity$steps), 2), nsmall = 2)
actMedian <- median(castedActivity$steps)
```
The mean is 10766.19 and the median is 10765.

What is the average daily activity pattern?
```{r}
meltedInterval <- melt(activity, id=c("interval"), na.rm=TRUE, measure.vars="steps")
castedInterval <- dcast(meltedInterval, interval ~ variable, mean)
plot( castedInterval$interval, castedInterval$steps, type="l")
maxRow <- castedInterval[castedInterval$steps==max(castedInterval$steps),]
```

The maximum number of steps is 206.1698 at time interval 835.

Imputing missing values
```{r}
x <- activity$steps
x1 <- length(which(is.na(x)))
```

There are 2304 missing values. I assigned the average number of steps for each interval into those intervals with NA's

```{r}
activityNa <- is.na(activity$steps)
castedIntervalAdj <- cbind(castedInterval, as.integer(round(castedInterval$steps)))
nonNaActivity <- activity[!activityNa,]
NaActivity <- activity[activityNa,]
NaResolved <- merge(NaActivity, castedIntervalAdj, by.x = "interval", by.y = "interval", all=FALSE )
NaResolved$steps.x <- NULL
NaResolved$steps.y <- NULL
names(NaResolved)[3] <- paste("steps")
NaResolvedActivity <- rbind(NaResolved, nonNaActivity)
meltedActivity <- melt(NaResolvedActivity, id=c("date"), na.rm=TRUE, measure.vars="steps")
castedActivity <- dcast(meltedActivity, date ~ variable, sum)
hist(castedActivity$steps)

impMean <- format(round(mean(castedActivity$steps), 2), nsmall = 2)
impMedian <- median(castedActivity$steps)
```

The mean is 10765.64 and the median is 10762

Are there differences in activity patterns between weekdays and weekends?
```{r}
wd <- !(weekdays(as.Date(NaResolvedActivity$date)) %in% c('Saturday','Sunday'))
wdwe <- c("", "")
for (i in 1:length(wd)) {
  if (wd[i]) {wdwe[i] <- "Weekday"} else {wdwe[i] <- "Weekend"}
}
NaResolvedActivity[, "dayType"] <- factor(wdwe)
p <- ggplot(NaResolvedActivity, aes(x=interval, y=steps)) + geom_line()
melted <- melt(NaResolvedActivity, id=c("interval", "dayType"), na.rm=TRUE, measure.vars="steps")
casted <- dcast(melted, interval + dayType ~ variable, mean)
p <- ggplot(casted, aes(x=interval, y=steps)) + geom_line() + ylab("Number of Steps")
p + facet_wrap(~ dayType, ncol=1)
```


