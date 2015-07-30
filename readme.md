# Getting and Cleaning Data Course Project

## Goal and Description of the Project

Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone.

A full description is available at the site where the data was obtained:

http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

The dataset for the project:

https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

THe goal of the project is to create one R script called run_analysis.R that does the following:1. Merges the training and the test sets to create one data set. 2. Extracts only the measurements on the mean and standard deviation for each measurement.3. Uses descriptive activity names to name the activities in the data set. 4. Appropriately labels the data set with descriptive variable names. 5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

The project repo contains project code for `Getting and Cleaning Data`. (Readme.MD, run_analysis.R and tidydata.txt)

## Script

The R code `run_analysis.R` proceeds under the assumption that the zip file available at https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip is downloaded and extracted in the R Working Directory.

## Preparation
###Loading Libraries

The libraries used in this code are `data.table` and `dplyr.` We prefered `data.table` as it is efficient in handling large data as tables. `dplyr` was used to aggregate variables to create the tidy data.

```r
library(data.table)
library(dplyr)
```
###Reading Supporting Metadata
The supporting metadata in the data are the name of the features and the name of the activities. They are loaded into variables `feature.Labels` and `activity.Labels`.

```r
feature.Labels <- read.table("UCI HAR Dataset/features.txt")
activity.Labels <- read.table("UCI HAR Dataset/activity_labels.txt", header = FALSE)
```
###Formating training and test data sets
Both training and test data sets are split up into subject, activity and features. They are present in three different files.

###Read training data

```r
subject.Train <- read.table("UCI HAR Dataset/train/subject_train.txt", header = FALSE)
activity.Train <- read.table("UCI HAR Dataset/train/y_train.txt", header = FALSE)
features.Train <- read.table("UCI HAR Dataset/train/X_train.txt", header = FALSE)
```
###Read test data
```r
subject.Test <- read.table("UCI HAR Dataset/test/subject_test.txt", header = FALSE)
activity.Test <- read.table("UCI HAR Dataset/test/y_test.txt", header = FALSE)
features.Test <- read.table("UCI HAR Dataset/test/X_test.txt", header = FALSE)
```

## Step 1 : Merge the training and the test sets to create one data set

We can combine the respective data in training and test data sets corresponding to subject, activity and features. The results are stored in `subject`, `activity` and `features`.

```r
subject <- rbind(subject.Train, subject.Test)
activity <- rbind(activity.Train, activity.Test)
features <- rbind(features.Train, features.Test)
```
###Naming the columns
The columns in the features data set can be named from the metadata in `featureLabels`

```r
colnames(features) <- t(feature.Labels[2])
```

###Merge the data
The data in `features`,`activity` and `subject` are merged and the complete data is now stored in completeData
```r
colnames(activity) <- "Activity"
colnames(subject) <- "Subject"
completeData <- cbind(features,activity,subject)
```
## Step 2 : Extracts only the measurements on the mean and standard deviation for each measurement
Extract the column indices that have either mean or std in them

```r
columnsWithMeanSTD <- grep(".*Mean.*|.*Std.*", names(completeData), ignore.case=TRUE)
```
Add activity and subject columns to the list and look at the dimension of `completeData`
```r
requiredColumns <- c(columnsWithMeanSTD, 562, 563)
dim(completeData)
```
We create `extractedData` with the selected columns in `requiredColumns`. And again, we look at the dimension of `requiredColumns`.
```r
extractedData <- completeData[,requiredColumns]
dim(extractedData)
```
## Step 3 : Uses descriptive activity names to name the activities in the data set

The `activity` field in `extractedData` is originally of numeric type. We need to change its type to character so that it can accept activity names. The activity names are taken from metadata `activityLabels`.

```r
extractedData$Activity <- as.character(extractedData$Activity)
for (i in 1:6){
extractedData$Activity[extractedData$Activity == i] <- as.character(activityLabels[i,2])
}
```
We need to factor the `activity` variable, once the activity names are updated.

```r
extractedData$Activity <- as.factor(extractedData$Activity)
```
## Step 4 : Appropriately labels the data set with descriptive variable names

Here are the names of the variables in `extractedData`
```r
names(extractedData)
```

By examining extractedData, we can say that the following acronyms can be replaced:
`Acc` can be replaced with Accelerometer    
`Gyro` can be replaced with Gyroscope
`BodyBody` can be replaced with Body
`Mag` can be replaced with Magnitude
Character `f` can be replaced with Frequency
Character `t` can be replaced with Time

```r
names(extractedData)<-gsub("Acc", "Accelerometer", names(extractedData))
names(extractedData)<-gsub("Gyro", "Gyroscope", names(extractedData))
names(extractedData)<-gsub("BodyBody", "Body", names(extractedData))
names(extractedData)<-gsub("Mag", "Magnitude", names(extractedData))
names(extractedData)<-gsub("^t", "Time", names(extractedData))
names(extractedData)<-gsub("^f", "Frequency", names(extractedData))
names(extractedData)<-gsub("tBody", "TimeBody", names(extractedData))
names(extractedData)<-gsub("-mean()", "Mean", names(extractedData), ignore.case = TRUE)
names(extractedData)<-gsub("-std()", "STD", names(extractedData), ignore.case = TRUE)
names(extractedData)<-gsub("-freq()", "Frequency", names(extractedData), ignore.case = TRUE)
names(extractedData)<-gsub("angle", "Angle", names(extractedData))
names(extractedData)<-gsub("gravity", "Gravity", names(extractedData))
```

Here are the names of the variables in `extractedData` after they are edited

```r
names(extractedData)
```

## Step 5 : From the data set in step 4, creates a tidy data set with the average of each variable for each activity and each subject

Firstly, let us set `Subject` as a factor variable.

```r
extractedData$Subject <- as.factor(extractedData$Subject)
extractedData <- data.table(extractedData)
```

We create `tidyData` as a data set with average for each activity and subject. Then, we order the enties in `tidyData` and write it into data file `tidydata.txt` and `tidydata.csv` that contain the processed data.

```r
tidyData <- aggregate(. ~Subject + Activity, extractedData, mean)
tidyData <- tidyData[order(tidyData$Subject,tidyData$Activity),]
write.table(tidyData, file = "tidydata.txt", row.names = FALSE)
write.csv(tidyData, file = "tidydata.csv", row.names = FALSE)
```

## Variables in raw and tidy data set

 Raw data set | Tidy data set
 -------------|--------------
 `subject` | `subject`
 `label` | `label`
 `tBodyAcc-mean()-X` | `TimeBodyAccelerometerMean()-X`
 `tBodyAcc-mean()-Y` | `TimeBodyAccelerometerMean()-Y`
 `tBodyAcc-mean()-Z` | `TimeBodyAccelerometerMean()-Z`
 `tBodyAcc-std()-X` | `TimeBodyAccelerometerSTD()-X`
 `tBodyAcc-std()-Y` | `TimeBodyAccelerometerSTD()-Y`
 `tBodyAcc-std()-Z` | `TimeBodyAccelerometerSTD()-Z`
 `tGravityAcc-mean()-X` | `TimeGravityAccelerometerMean()-X`
 `tGravityAcc-mean()-Y` | `TimeGravityAccelerometerMean()-Y`
 `tGravityAcc-mean()-Z` | `TimeGravityAccelerometerMean()-Z`
 `tGravityAcc-std()-X` | `TimeGravityAccelerometerSTD()-X`
 `tGravityAcc-std()-Y` | `TimeGravityAccelerometerSTD()-Y`
 `tGravityAcc-std()-Z` | `TimeGravityAccelerometerSTD()-Z`
 `tBodyAccJerk-mean()-X` | `TimeBodyAccelerometerJerkMean()-X`
 `tBodyAccJerk-mean()-Y` | `TimeBodyAccelerometerJerkMean()-Y`
 `tBodyAccJerk-mean()-Z` | `TimeBodyAccelerometerJerkMean()-Z`
 `tBodyAccJerk-std()-X` | `TimeBodyAccelerometerJerkSTD()-X`
 `tBodyAccJerk-std()-Y` | `TimeBodyAccelerometerJerkSTD()-Y`
 `tBodyAccJerk-std()-Z` | `TimeBodyAccelerometerJerkSTD()-Z`
 `tBodyGyro-mean()-X` | `TimeBodyGyroscopeMean()-X`
 `tBodyGyro-mean()-Y` | `TimeBodyGyroscopeMean()-Y`
 `tBodyGyro-mean()-Z` | `TimeBodyGyroscopeMean()-Z`
 `tBodyGyro-std()-X` | `TimeBodyGyroscopeSTD()-X`
 `tBodyGyro-std()-Y` | `TimeBodyGyroscopeSTD()-Y`
 `tBodyGyro-std()-Z` | `TimeBodyGyroscopeSTD()-Z`
 `tBodyGyroJerk-mean()-X` | `TimeBodyGyroscopeJerkMean()-X`
 `tBodyGyroJerk-mean()-Y` | `TimeBodyGyroscopeJerkMean()-Y`
 `tBodyGyroJerk-mean()-Z` | `TimeBodyGyroscopeJerkMean()-Z`
 `tBodyGyroJerk-std()-X` | `TimeBodyGyroscopeJerkSTD()-X`
 `tBodyGyroJerk-std()-Y` | `TimeBodyGyroscopeJerkSTD()-Y`
 `tBodyGyroJerk-std()-Z` | `TimeBodyGyroscopeJerkSTD()-Z`
 `tBodyAccMag-mean()` | `TimeBodyAccelerometerMagnitudeMean()`
 `tBodyAccMag-std()` | `TimeBodyAccelerometerMagnitudeSTD()`
 `tGravityAccMag-mean()` | `TimeGravityAccelerometerMagnitudeMean()`
 `tGravityAccMag-std()` | `TimeGravityAccelerometerMagnitudeSTD()`
 `tBodyAccJerkMag-mean()` | `TimeBodyAccelerometerJerkMagnitudeMean()`
 `tBodyAccJerkMag-std()` | `TimeBodyAccelerometerJerkMagnitudeSTD()`
 `tBodyGyroMag-mean()` | `TimeBodyGyroscopeMagnitudeMean()`
 `tBodyGyroMag-std()` | `TimeBodyGyroscopeMagnitudeSTD()`
 `tBodyGyroJerkMag-mean()` | `TimeBodyGyroscopeJerkMagnitudeMean()`
 `tBodyGyroJerkMag-std()` | `TimeBodyGyroscopeJerkMagnitudeSTD()`
 `fBodyAcc-mean()-X` | `FrequencyBodyAccelerometerMean()-X`
 `fBodyAcc-mean()-Y` | `FrequencyBodyAccelerometerMean()-Y`
 `fBodyAcc-mean()-Z` | `FrequencyBodyAccelerometerMean()-Z`
 `fBodyAcc-std()-X` | `FrequencyBodyAccelerometerSTD()-X`
 `fBodyAcc-std()-Y` | `FrequencyBodyAccelerometerSTD()-Y`
 `fBodyAcc-std()-Z` | `FrequencyBodyAccelerometerSTD()-Z`
 `fBodyAccJerk-mean()-X` | `FrequencyBodyAccelerometerJerkMean()-X`
 `fBodyAccJerk-mean()-Y` | `FrequencyBodyAccelerometerJerkMean()-Y`
 `fBodyAccJerk-mean()-Z` | `FrequencyBodyAccelerometerJerkMean()-Z`
 `fBodyAccJerk-std()-X` | `FrequencyBodyAccelerometerJerkSTD()-X`
 `fBodyAccJerk-std()-Y` | `FrequencyBodyAccelerometerJerkSTD()-Y`
 `fBodyAccJerk-std()-Z` | `FrequencyBodyAccelerometerJerkSTD()-Z`
 `fBodyGyro-mean()-X` | `FrequencyBodyGyroscopeMean()-X`
 `fBodyGyro-mean()-Y` | `FrequencyBodyGyroscopeMean()-Y`
 `fBodyGyro-mean()-Z` | `FrequencyBodyGyroscopeMean()-Z`
 `fBodyGyro-std()-X` | `FrequencyBodyGyroscopeSTD()-X`
 `fBodyGyro-std()-Y` | `FrequencyBodyGyroscopeSTD()-Y`
 `fBodyGyro-std()-Z` | `FrequencyBodyGyroscopeSTD()-Z`
 `fBodyAccMag-mean()` | `FrequencyBodyAccelerometerMagnitudMean()`
 `fBodyAccMag-std()` | `FrequencyBodyAccelerometerMagnitudSTD()`
 `fBodyBodyAccJerkMag-mean()` | `FrequencyBodyAccelerometerJerkMagnitudMean()`
 `fBodyBodyAccJerkMag-std()` | `FrequencyBodyAccelerometerJerkMagnitudSTD()`
 `fBodyBodyGyroMag-mean()` | `FrequencyBodyGyroMagnitudMean()`
 `fBodyBodyGyroMag-std()` | `FrequencyBodyGyroMagnitudSTD()`
 `fBodyBodyGyroJerkMag-mean()` | `FrequencyBodyGyroJerkMagnitudSMean()`
 `fBodyBodyGyroJerkMag-std()` | `FrequencyBodyGyroJerkMagnitudSTD`

### Code book

No code book is provided, since this README file contains all the required
information. However, the code book is still provided 