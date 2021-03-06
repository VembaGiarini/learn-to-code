#load library
library(readr)
library(dplyr)
library(lubridate)
library(tidyr)
library(visdat)
library(naniar)
library(caTools)
library(psych)
library(dplyr)
library(glmnet)

#import data
prod_demand <- read.csv("Historical Product Demand.csv")

#looking through all the columns with glimpse()
glimpse(prod_demand)

#checking the class
class(prod_demand$Date)

#change the 'string' column Date into 'date'
prod_demand$Date <- as.Date(prod_demand$Date)

#change the 'string' column Order_demand into 'int'
prod_demand$Order_Demand <- as.integer(prod_demand$Order_Demand)

#print prod_demand
prod_demand %>%
  glimpse()

#finding missing values
#is.na(prod_demand)


#counting missing values
sum(is.na(prod_demand))

#observing missing value
vis_miss(prod_demand, warn_large_data = FALSE)

#removing missing values
product_demand <- na.omit(prod_demand)

#counting missing values
sum(is.na(product_demand))

# split training and test data
library(caTools)
set.seed(123)
sample <- sample.split(product_demand$Order_Demand, SplitRatio = .80)
train <- subset(product_demand, sample == TRUE)
test <- subset(product_demand, sample == FALSE)

# correlation study
pairs.panels(train, 
             method = "pearson", # correlation method
             hist.col = "#00AFBB",
             density = TRUE,  # show density plots
             ellipses = TRUE # show correlation ellipses
       
# drop correlated columns
library(dplyr)
drop_cols <- c('Product_Category')

train <- train %>% select(-drop_cols)
test <- test %>% select(-drop_cols)

# feature preprocessing
# to ensure we handle categorical features
x <- model.matrix(Order_Demand ~ ., train)[,-1]
y <-  train$Order_Demand

# Ridge Regression with lambda = 0.1
# fit ridge regression (alpha = 0)
library(glmnet)
ridge_reg <- glmnet(x, y, alpha = 0, lambda = 0.1)
coef(ridge_reg)

# evaluation
# Make predictions on the test data
x_test <- model.matrix(Order_Demand ~., test)[,-1]
y_test <- test$Order_Demand

predictions <- predict(ridge_reg, x_test)

# Rsquared <-  cor(true_values, predicted_values)^2
# on training data
R2 <- as.numeric(cor(y, predict(ridge_reg, x)) ^ 2)
R2

# RMSE <- sqrt(mean((true_values - predicted_values)^2))
# on test data
RMSE <- sqrt(mean((y_test - predict(ridge_reg, x_test))^2))
RMSE

# LASSO with lambda = 0.1
# fit LASSO (alpha = 1)
lasso_reg <- glmnet(x, y, alpha = 1, lambda = 0.1)
coef(lasso_reg)

# REVISED workflow
# further split train data to train and validation
set.seed(123)
sample <- sample.split(demand$Product_Demand, SplitRatio = .80)
pre_train <- subset(demand, sample == TRUE)
sample_train <- sample.split(pre_train$Product_Demand, SplitRatio = .80)

# train-validation data
train <- subset(pre_train, sample_train == TRUE)
validation <- subset(pre_train, sample_train == FALSE)

# test data
test <- subset(demand, sample == FALSE)

# drop correlated columns
drop_cols <- c('Product_Category')

train <- train %>% select(-drop_cols)
validation <-  validation %>% select(-drop_cols)
test <- test %>% select(-drop_cols)

# feature preprocessing
# to ensure we handle categorical features
x <- model.matrix(Product_Demand ~ ., train)[,-1]
y <-  train$Product_Demand

# ridge regression
# fit multiple ridge regression with different lambda
# lambda = [0.01, 0.1, 1, 10]
ridge_reg_pointzeroone <- glmnet(x, y, alpha = 0, lambda = 0.01)
coef(ridge_reg_pointzeroone)

ridge_reg_pointone <- glmnet(x, y, alpha = 0, lambda = 0.1)
coef(ridge_reg_pointone)

ridge_reg_one <- glmnet(x, y, alpha = 0, lambda = 1)
coef(ridge_reg_pointone)

ridge_reg_ten <- glmnet(x, y, alpha = 0, lambda = 10)
coef(ridge_reg_ten)

# comparison on validation data
# to choose the best lambda

# Make predictions on the validation data
x_validation <- model.matrix(Product_Demand ~., validation)[,-1]
y_validation <- validation$Product_Demand

RMSE_ridge_pointzeroone <- sqrt(mean((y_validation - predict(ridge_reg_pointzeroone, x_validation))^2))
RMSE_ridge_pointzeroone 

RMSE_ridge_pointone <- sqrt(mean((y_validation - predict(ridge_reg_pointone, x_validation))^2))
RMSE_ridge_pointone 

RMSE_ridge_one <- sqrt(mean((y_validation - predict(ridge_reg_one, x_validation))^2))
RMSE_ridge_one 

RMSE_ridge_ten <- sqrt(mean((y_validation - predict(ridge_reg_ten, x_validation))^2))
RMSE_ridge_ten 

# true evaluation on test data
# using the best model --> RMSE_ridge_pointzeroone
x_test <- model.matrix(Product_Demand ~., test)[,-1]
y_test <- test$Product_Demand

# RMSE
RMSE_ridge_best <- sqrt(mean((y_test - predict(ridge_reg_pointzeroone, x_test))^2))
RMSE_ridge_best

# MAE
MAE_ridge_best <- mean(abs(y_test-predict(ridge_reg_pointzeroone, x_test)))
MAE_ridge_best

# MAPE
MAPE_ridge_best <- mean(abs((predict(ridge_reg_pointzeroone, x_test) - y_test))/y_test) 
MAPE_ridge_best

############## LASSO
# lasso regression
# fit multiple lasso regression with different lambda
# lambda = [0.01, 0.1, 1, 10]
lasso_reg_pointzeroone <- glmnet(x, y, alpha = 1, lambda = 0.01)
coef(lasso_reg_pointzeroone) 

lasso_reg_pointone <- glmnet(x, y, alpha = 1, lambda = 0.1)
coef(lasso_reg_pointone) 

lasso_reg_one <- glmnet(x, y, alpha = 1, lambda = 1)
coef(lasso_reg_pointone)

lasso_reg_ten <- glmnet(x, y, alpha = 1, lambda = 10)
coef(lasso_reg_ten)
