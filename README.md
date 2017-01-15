### Calling libraries

    library(dplyr)

    ## Warning: package 'dplyr' was built under R version 3.3.2

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(tidyr)

    ## Warning: package 'tidyr' was built under R version 3.3.2

### Downloading and unzipping files

    fileurl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"

    download.file(fileurl,destfile = "./UCI HAR Dataset.zip", mode = "wb")

    unzip("./UCI HAR Dataset.zip")

### Reading files into tables

    X_test <- read.table("./UCI HAR Dataset/test/X_test.txt")
    X_train <- read.table("./UCI HAR Dataset/train/X_train.txt")

    Y_test <- read.table("./UCI HAR Dataset/test/Y_test.txt")
    Y_train <- read.table("./UCI HAR Dataset/train/Y_train.txt")

    subject_test <- read.table("./UCI HAR Dataset/test/subject_test.txt")
    subject_train <- read.table("./UCI HAR Dataset/train/subject_train.txt")

    features <- read.table("./UCI HAR Dataset/features.txt")
    activity_labels <- read.table("./UCI HAR Dataset/activity_labels.txt")

### Combining train and test tables of each content set. Converting to table format

    X <- rbind(X_test, X_train);                    X <- tbl_df(X)
    Y <- rbind(Y_test, Y_train);                    Y <- tbl_df(Y)

    subject <- rbind(subject_test, subject_train);  subject <- tbl_df(subject)

    features <- tbl_df(features)
    activity_labels <- tbl_df(activity_labels)

### Removing redundant tables

    rm(X_test, X_train, Y_test, Y_train, subject_test, subject_train)

### Setting Variable names to features' descriptions

    column_names <- sapply(features$V2, as.character)
    names(X) <- column_names

### Extracting only the measurements on the mean and standard deviation for each measurement.

    subset_X <- X[,grep("mean|std", names(X))]
            subset_X <- subset_X[,-grep("meanFreq()|Jerk|Mag|^f",names(subset_X))]

Here we filter out measures which are not raw measurements, but rather
derived Such as Jerk, Magnitude, Furier Transformation and Mean
Frequency

### Making variable names readable

    names(subset_X) <- sub("Acc", "Acceleration", names(subset_X))
    names(subset_X) <- sub("Gyro", "Gyroscope", names(subset_X))
    names(subset_X) <- sub("^t", "Time", names(subset_X))
    names(subset_X) <- sub("-", "Signal", names(subset_X))
    names(subset_X) <- sub("\\()", "", names(subset_X))

### Using activity Names

    Y <- Y %>% inner_join(activity_labels, by = "V1") %>%
            transmute(activity = V2)

### Subject variable name change from "V1" to "subject"

    names(subject) <- "subject"

### Combining subject, subset\_X, and Y tables

    df <- subject %>% cbind(Y) %>% cbind(subset_X) %>% tbl_df()

### Making data narrow to perform averaging across subject, activity, and measure

    tidy <- df %>% gather(measure, value, 3:19)
    tidy <- tidy %>% group_by(subject, activity, measure) %>%
            summarize(meanvalue = mean(value))

### Reverting data back to the wide format when in each column there is a different measurement mean

    final_table <- tidy %>% spread(measure, meanvalue)
