---
title: "PA1_template.md"
author: "Levin Dyson"
date: "July 21, 2019"
output:
  html_document: default
  pdf_document: default
---



Read and process the data file
```{r}

activity <- readr:::read_csv("C:/Users/levin/Downloads/repdata_data_activity (1).zip")

activity$day <- weekdays(as.Date(activity$date))
activity$DateTime<- as.POSIXct(activity$date, format="%Y-%m-%d")
```

remove nas from data
```{r}
data_clean <- activity[!is.na(activity$steps),]

steps_Date <- aggregate(activity$steps ~ activity$date, FUN=sum, )
colnames(steps_Date)<- c("Date", "Steps")
```

Graph number of steps then find mean/median
```{r}
#histogram of total steps per day
hist(steps_Date$Steps, breaks = 15, xlab = "Steps", main = "Total Steps per Day", col = "darkred")

#Mean number of steps per day
mean_steps <- as.integer(mean(steps_Date$Steps))

#Median number of steps per day
median_steps <- as.integer(median(steps_Date$Steps))
```

Aggregate function for mean over all days, for each interval
```{r}
activity_steps_mean <- aggregate(steps ~ interval, data = activity, FUN = mean, na.rm = TRUE)
#Plot
plot(activity_steps_mean$interval, activity_steps_mean$steps, type = "l", col = "blue", xlab = "Intervals", 
     ylab = "Total steps per interval", main = "Average Number of Steps per Interval")
```


maximum of steps on a given interval

```{r}
max_steps <-max(activity_steps_mean$steps)

#Interval that contains highest number of steps per interval
max_interval <- activity_steps_mean$interval[which(activity_steps_mean$steps == max_steps)]
max_steps <- round(max_steps, digits = 2)
max_steps
max_interval
```

## What is mean total number of steps taken per day?


```{r}
#calcualte number of missing values
sum(is.na(activity))
```


#Devise a strategy for filling in all of the missing values in the dataset


```{r}
#subset general dataset with missing values only
missing_values <- subset(activity, is.na(steps))

#plot repartition, by date or by intervals
par(mfrow = c(2,1), mar = c(2, 2, 1, 1))
hist(missing_values$interval, main="NAs repartition per interval")
hist(as.numeric(missing_values$date), main = "NAs repartion per date", breaks = 61)

# calculate mean of steps per interval
MeanStepsPerInterval <- tapply(activity$steps, activity$interval, mean, na.rm = TRUE)

# cut the 'activity' dataset in 2 parts (with and without NAs)
activity_NAs <- activity[is.na(activity$steps),]
activity_non_NAs <- activity[!is.na(activity$steps),]

#replace missing values in activity_NAs
activity_NAs$steps <- as.factor(activity_NAs$interval)
levels(activity_NAs$steps) <- MeanStepsPerInterval

#change the vector back as integer 
levels(activity_NAs$steps) <- round(as.numeric(levels(activity_NAs$steps)))
activity_NAs$steps <- as.integer(as.vector(activity_NAs$steps))

#merge/rbind the two datasets together
imputed_activity <- rbind(activity_NAs, activity_non_NAs)

#Plotting parameters to place previous histogram and new one next to each other
par(mfrow = c(1,2))

#Plot again the histogram from the first part of the assignment
activity_steps_day <- aggregate(steps ~ date, data = activity, FUN = sum, na.rm = TRUE)
hist(activity_steps_day$steps, xlab = "Steps per Day", main = "NAs REMOVED - Total steps/day", col = "lightblue")

#Plot new histogram, with imputed missing values
imp_activity_steps_day <- aggregate(steps ~ date, data = imputed_activity, FUN = sum, na.rm = TRUE)
hist(imp_activity_steps_day$steps, xlab = "Steps per Day", main = "NAs IMPUTED - Total steps/day", col = "lightblue")

#Calculate mean and median steps and store the new and old results in table to compare
imp_mean_steps <- mean(imp_activity_steps_day$steps)
imp_median_steps <- median(imp_activity_steps_day$steps)

#we set a normal number format to display the results
imp_mean_steps <- format(imp_mean_steps,digits=1)
imp_median_steps <- format(imp_median_steps,digits=1)

#store the results in a dataframe
results_mean_median <- data.frame(c(mean_steps, median_steps), c(imp_mean_steps, imp_median_steps))
colnames(results_mean_median) <- c("NA removed", "Imputed NA values")
rownames(results_mean_median) <- c("mean", "median")
library(xtable)
table <- xtable(results_mean_median)
print(table, type  = "html")
```

#Are there differences in activity patterns between weekdays and weekends?

```{r}
#elseif function to categorize Saturday and Sunday as factor level "weekend", all the rest as "weekday"
imputed_activity$dayType <- ifelse(weekdays(imputed_activity$date) %in% c("Saturday", "Sunday"), "weekend", "weekday")
imputed_activity$dayType <- factor(imputed_activity$dayType)

#Aggregate a table showing mean steps for all intervals, acrlss week days and weekend days
steps_interval_dayType <- aggregate(steps ~ interval + dayType, data = imputed_activity, FUN = mean)
#verify new dataframe 
head(steps_interval_dayType)

#add descriptive variables
names(steps_interval_dayType) <- c("interval", "day_type", "mean_steps")
#plot with ggplot2
library(ggplot2)
plot <- ggplot(steps_interval_dayType, aes(interval, mean_steps))
plot + geom_line(color = "tan3") + facet_grid(day_type~.) + labs(x = "Intervals", y = "Average Steps", title = "Activity Patterns")
```

