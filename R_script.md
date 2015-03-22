R script:


library(dplyr)

features <- read.table("./R-data/Course Project/UCI HAR Dataset/features.txt")
activity.labels <- read.table("./R-data/Course Project/UCI HAR Dataset/activity_labels.txt", col.names = c("Activity", "ActivityName"))

x_train <- read.table("./R-data/Course Project/UCI HAR Dataset/train/X_train.txt", col.names = features[,2])
y_train <- read.table("./R-data/Course Project/UCI HAR Dataset/train/y_train.txt", col.names = "Activity")
subject.train <- read.table("./R-data/Course Project/UCI HAR Dataset/train/subject_train.txt", col.names = "Subject")

x_test <- read.table(file = "./R-data/Course Project/UCI HAR Dataset/test/X_test.txt", col.names = features[,2])
y_test <- read.table(file = "./R-data/Course Project/UCI HAR Dataset/test/y_test.txt", col.names = "Activity")
subject.test <- read.table(file = "./R-data/Course Project/UCI HAR Dataset/test/subject_test.txt", col.names = "Subject")

train <- bind_cols(subject.train, y_train, x_train)
test <- bind_cols(subject.test, y_test, x_test)
merged_data <- bind_rows(train, test)

req.features <- filter(features, grepl('mean\\(|std\\(', V2))
req.features <- c(c(1:2), req.features[, 1]+2)
merged_data <- merged_data[, req.features]

merged_data <- inner_join(activity.labels, merged_data, by = "Activity")
merged_data <- select(merged_data, -Activity)

colnames(merged_data) <- gsub("BodyBody", "Body", colnames(merged_data))
colnames(merged_data) <- gsub("\\.mean\\.\\.\\.", "-mean()-", colnames(merged_data))
colnames(merged_data) <- gsub("\\.std\\.\\.\\.", "-std()-", colnames(merged_data))
colnames(merged_data) <- gsub("\\.mean\\.\\.", "-mean()", colnames(merged_data))
colnames(merged_data) <- gsub("\\.std\\.\\.", "-std()", colnames(merged_data))

merged_data.mean <- merged_data %>% group_by(ActivityName, Subject) %>% summarise_each(funs(mean))

write.table(merged_data.mean, file = "./R-data/Course Project/HAR_merged_data.txt", row.name = FALSE)
