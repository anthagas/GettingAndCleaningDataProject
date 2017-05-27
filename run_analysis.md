## R script for getting and cleaning data project
## 28-05-2017
## By anthagas

## Step 1: Download an unzip to the dataset:

## Check wd
setwd("~/DataScience/Coursera")

## Check if directory already exists?
if(!file.exists("./projectdata")){
    dir.create("./projectdata")
}
Url<-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"

## Check if zip has already been downloaded in projectdata directory?
if(!file.exists("./projectdata/projectdataset.zip")){
  download.file(Url,destfile="./projectdata/projectdataset.zip",mode = "wb")
}

## Check if zip has already been unzipped?
if(!file.exists("./projectdata/UCI HAR Dataset")){
  unzip(zipfile="./projectdata/projectdataset.zip",exdir="./projectdata")
}

## List all the filesof UCI HAR Dataset folder
path<-file.path("projectdata","UCI HAR Dataset")
files<-list.files(path, recursive = TRUE)

## The files that will be used to load data are listed as follows:
# test/subject_test.txt
# text/X_text.txt
# text/y_text.txt
# train/subject_train.txt
# train/X_train.txt
# train/y_train.txt

## Step 2: Load activity, subject and featura info
## read data from the files into the variables

## 1. Read the activity files
activitytest<-read.table(file.path(path, "test", "Y_test.txt"), header = FALSE)
activitytrain<-read.table(file.path(path, "train", "Y_train.txt"), header = FALSE)

## 2. Read the subject files
subjecttest<-read.table(file.path(path, "test", "subject_test.txt"), header = FALSE)
subjecttrain<-read.table(file.path(path, "train", "subject_train.txt"), header = FALSE)

## 3. Read features files
featurestest<-read.table(file.path(path, "test", "X_test.txt"), header = FALSE)
featuretrain<-read.table(file.path(path, "train", "X_train.txt"), header = FALSE)

## Test: Check properties

##str(activitytest)
##str(activitytrain)
##str(subjecttrain)
##str(featurestest)
##str(featurestrain)

## Step 3: Merges the trining and the test sets to create one data set

## 1. Concatenate the data tables by rows
datasubject<-rbind(subjecttrain,subjecttest)
dataactivity<-rbind(activitytrain,activitytest)
datafeatures<-rbind(featuretrain,featurestest)

## 2. Set names to variables
names(datasubject)<-c("subject")
names(dataactivity)<-c("activity")
datafeaturesnames<-read.table(file.path(path, "features.txt"),head=FALSE)
names(datafeatures)<-datafeaturesnames$V2

## 3. Merge columns to get the dataframe Data for all data
datacombine<-cbind(datasubject,dataactivity)
data<-cbind(datafeatures,datacombine)

## Step 4: Extracts only the measurements on the mean and standard deviation for each measurement
## 1. Subset Name of features by measurements on the mean and standard deviation
## i.e. taken Names of features with "mean()" or "std"
## Extract using grep
subdatafeaturesnames<-datafeaturesnames$V2[grep("mean\\(\\)|std\\(\\)",datafeaturesnames$V2)]

## 2. Subset the dataframe Data by selected names of features
selectednames<-c(as.character(subdatafeaturesnames), "subject","activity")
data<-subset(data,select = selectednames)

## 3. Test: Check the structures of the dataframe Data
##str(data)

## Step 5: Uses descriptive activity names to name the activities in the data set
## 1. Read descriptive activity names from "activity_labels.txt"
activitylabels<-read.table(file.path(path, "activity_labels.txt"), header = FALSE)

## 2. Factorize variable activity in the dataframe Data using descriptive activity names
data$activity<-factor(data$activity,labels = activitylabels[,2])

## 3. Test: Print
head(data$activity,30)

## Step 6: Appropriately labels the dataset with descriptive variable names
names(data)<-gsub("^t", "time", names(data))
names(data)<-gsub("^f", "frequency", names(data))
names(data)<-gsub("Acc", "Accelerometer", names(data))
names(data)<-gsub("Gyro","Gyroscope", names(data))
names(data)<-gsub("Mag","Magnitude", names(data))
names(data)<-gsub("BodyBoby","Body", names(data))

## Test: Print
names(data)

## Step 7: Creates an Idependent tidy data set
newdata<-aggregate(.~subject+activity,data,mean)
newdata<-newdata[order(newdata$subject,newdata$activity),]
write.table(newdata,file = "tidydata.txt",row.name=FALSE,quote = FALSE,sep = '\t')
