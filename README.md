# DS-Capstone-Project
DS capstone project code


#Code For Capstone Project
#load libraries
library(mice)
library(VIM) 
library(psych)
library(corrplot)
library("PerformanceAnalytics")
library(caret)
library(gvlma)
library(lmtest)
library(glmnet)
library(dplyr)
library(xgboost)
library(lme4)
library(lvmisc)
library(car)
library(data.table)
library(matlab)
library(leaps)
#load required
require(pracma)
require(mice)
require (VIM) 
require (psych)
require (corrplot)
require ("PerformanceAnalytics")
require (caret)
require (gvlma)
require (lmtest)
require (glmnet)
require (dplyr)
require (xgboost)
require(mboost)
require(leaps)


set.seed(100) # set seed same answers
#load in data
training = read.csv("C:/Users/Dad/Desktop/DS785/widsdatathon2022/train.csv")
testing = read.csv("C:/Users/Dad/Desktop/DS785/widsdatathon2022/test.csv")
training_cleaned = training #rename so if mistake is made do not lose main files
test_cleaned = testing #rename so if mistake is made do not lose main files
attach(training_cleaned) #attach data easier to access variables 
summary(training_cleaned) #informtion on all columns
str(training_cleaned) #information on all classes 
training_cleaned$year_built[training_cleaned$year_built == 0] = NA #remove suspicious values
training_cleaned$year_built = as.numeric(training_cleaned$year_built) #make the column numeric
boxplot(training_cleaned$site_eui, main = "Boxplot of Site (EUI)") #checking for outliers
#getting information to remove outliers
Q1 = quantile(training_cleaned$site_eui, .25)
Q3 = quantile(training_cleaned$site_eui, .75)
IQR = IQR(training_cleaned$site_eui)
#removing outliers
training_cleaned = subset(training_cleaned, training_cleaned$site_eui >= (Q1 - 1.5*IQR) & training_cleaned$site_eui <= (Q3 + 1.5*IQR))
dim(training_cleaned) #getting new shape of data frame
boxplot(training_cleaned$site_eui, main = "Boxplot of Site (EUI) After Outliers Removed")
sum(is.na(training_cleaned)) # looking for missing values
colSums(is.na(training_cleaned)) #counting missing values
#plotting missing values 
par(mar=c(2,2,2,2))
aggr_plot = aggr(training_cleaned, col=c('navyblue','red'), numbers=TRUE, sortVars=TRUE, labels=names(data), cex.axis=.7, gap=3, ylab=c("Histogram of missing data","Pattern"))
# changing character classes
training_cleaned$State_Factor [training_cleaned $State_Factor == "State_1"] = "1"
training_cleaned$State_Factor [training_cleaned $State_Factor == "State_2"] = "2"
training_cleaned$State_Factor [training_cleaned $State_Factor == "State_4"] = "4"
training_cleaned$State_Factor [training_cleaned $State_Factor == "State_6"] = "6"
training_cleaned$State_Factor [training_cleaned $State_Factor == "State_8"] = "8"
training_cleaned$State_Factor [training_cleaned $State_Factor == "State_10"] = "10"
training_cleaned$State_Factor [training_cleaned $State_Factor == "State_11"] = "11"
training_cleaned $State_Factor = as.numeric(training_cleaned $State_Factor)
training_cleaned$building_class = ifelse(training_cleaned$building_class == "Commercial", 1, 2)
training_cleaned$building_class = as.numeric(training_cleaned$building_class)
training_cleaned$facility_type = as.factor(training_cleaned$facility_type)
training_cleaned$facility_type = as.numeric(training_cleaned$facility_type)
#Finding best variables
training_cleaned_best_model = regsubsets(site_eui~., data =training_cleaned, nbest = 1, nvmax = 30, force.in = NULL, force.out = NULL, method = "exhaustive", really.big=T)
training_cleaned = training_cleaned_best_model
#Creating best set of dependent variables
training_cleaned = subset (training_cleaned, select = -c(1, 2, 8, 9, 11, 12, 14, 15, 17, 18, 20, 21, 23, 24, 26, 27, 29, 30, 32, 33, 35, 36, 38, 39, 41, 42, 44, 47, 48, 49, 59, 60, 61, 62, 64))
training_cleaned[] = lapply(training_cleaned, function(x) as.numeric(as.character(x))) #changing to numeric
#Inserting missing values creating 5 datasets 
tempData_training = mice(training_cleaned,m=5,maxit=50,meth='lasso.norm')
training_cleaned_mice1Data = complete(tempData_training,1)
training_cleaned_mice2Data = complete(tempData_training,2)
training_cleaned_mice3Data = complete(tempData_training,3)
training_cleaned_mice4Data = complete(tempData_training,4)
training_cleaned_mice5Data = complete(tempData_training,5)

#TEST DATA
#cleaning of the test data
test_cleaned$year_built[test_cleaned$year_built == 0] = NA
sum(is.na(test_cleaned))
colSums(is.na(test_cleaned))
aggr_plot = aggr(test_cleaned, col=c('navyblue','red'), numbers=TRUE, sortVars=TRUE, labels=names(data), cex.axis=.7, gap=3, ylab=c("Histogram of missing data","Pattern"))
test_cleaned$State_Factor [test_cleaned $State_Factor == "State_1"] = "1"
test_cleaned $State_Factor [test_cleaned $State_Factor == "State_2"] = "2"
test_cleaned $State_Factor [test_cleaned $State_Factor == "State_4"] = "4"
test_cleaned $State_Factor [test_cleaned $State_Factor == "State_8"] = "8"
test_cleaned $State_Factor [test_cleaned $State_Factor == "State_10"] = "10"
test_cleaned $State_Factor [test_cleaned $State_Factor == "State_11"] = "11"
test_cleaned $State_Factor = as.numeric(test_cleaned $State_Factor)
test_cleaned$building_class = ifelse(test_cleaned$building_class == "Commercial", 1, 2)
test_cleaned$building_class = as.numeric(test_cleaned$building_class)
test_cleaned$facility_type = as.factor(test_cleaned$facility_type)
test_cleaned$facility_type = as.numeric(test_cleaned$facility_type)
test_cleaned = subset (test_cleaned, select = -c(1, 2, 8, 9, 11, 12, 14, 15, 17, 18, 20, 21, 23, 24, 26, 27, 29, 30, 32, 33, 35, 36, 38, 39, 41, 42, 44, 47, 48, 49, 59, 60, 61, 62, 63))
test_cleaned[] = lapply(test_cleaned, function(x) as.numeric(as.character(x)))
tempData_test = mice(test_cleaned,m=5,maxit=50,meth='lasso.norm')
test_cleaned_mice1Data = complete(tempData_test, 1)
test_cleaned_mice2Data = complete(tempData_test, 2)
test_cleaned_mice3Data = complete(tempData_test, 3)
test_cleaned_mice4Data = complete(tempData_test, 4)
test_cleaned_mice5Data = complete(tempData_test, 5)

#MODEL DATA
plot(year_built,energy_star_rating) #linearity check
abline(v = 1950, col = "red", lwd = 3)
hist(training$site_eui) #normal distribution
hist(training_cleaned$site_eui)
cor(training_cleaned$floor_area, training_cleaned$energy_star_rating, use = "complete.obs") # homoscedasticity check 


#MULTIPLE LINEAR REGRESSION
#building of the linear models
LR_model1 = lm(site_eui ~ ., data = training_cleaned_mice1Data)
summary(LR_model1)
LR_model2 = lm(site_eui ~ ., data = training_cleaned_mice2Data)
summary(LR_model2)
LR_model3 = lm(site_eui ~ ., data = training_cleaned_mice3Data)
summary(LR_model3)
LR_model4 = lm(site_eui ~ ., data = training_cleaned_mice4Data)
summary(LR_model4)
LR_model5 = lm(site_eui ~ ., data = training_cleaned_mice5Data)
summary(LR_model5)

LR_predict_model1 = predict(LR_model1, newdata = test_cleaned_mice1Data)
LR_predict_model2 = predict(LR_model2, newdata = test_cleaned_mice2Data)
LR_predict_model3 = predict(LR_model3, newdata = test_cleaned_mice3Data)
LR_predict_model4 = predict(LR_model4, newdata = test_cleaned_mice4Data)
LR_predict_model5 = predict(LR_model5, newdata = test_cleaned_mice5Data)


RMSE_LR_model1 = RMSE(LR_predict_model1, training_cleaned_mice1Data$site_eui[0:9705])
RMSE_LR_model2 = RMSE(LR_predict_model2, training_cleaned_mice2Data$site_eui[0:9705])
RMSE_LR_model3 = RMSE(LR_predict_model3, training_cleaned_mice3Data$site_eui[0:9705])
RMSE_LR_model4 = RMSE(LR_predict_model4, training_cleaned_mice4Data$site_eui[0:9705])
RMSE_LR_model5 = RMSE(LR_predict_model5, training_cleaned_mice5Data$site_eui[0:9705])
RMSE_LR_model1
RMSE_LR_model2
RMSE_LR_model3
RMSE_LR_model4
RMSE_LR_model5


par(mar=c(2,2,2,2))
plot.new()
par(mfrow=c(2,2))
plot(LR_model1) #best model
avPlots(LR_model3) #best model
 
train_control = trainControl(method = "cv", number =10)
 LR_cv_model1 = train(site_eui ~., data = training_cleaned_mice1Data,
               method = "lm", trControl = train_control)
LR_cv_model2 = train(site_eui ~., data = training_cleaned_mice2Data,
               method = "lm", trControl = train_control)
LR_cv_model3 = train(site_eui ~., data = training_cleaned_mice3Data,
               method = "lm", trControl = train_control)
LR_cv_model4 = train(site_eui ~., data = training_cleaned_mice4Data,
               method = "lm", trControl = train_control)
LR_cv_model5 = train(site_eui ~., data = training_cleaned_mice5Data,
               method = "lm", trControl = train_control)
LR_predict_cv_model1 = predict(LR_cv_model1, newdata = test_cleaned_mice1Data)
LR_predict_cv_model2 = predict(LR_cv_model2, newdata = test_cleaned_mice2Data)
LR_predict_cv_model3 = predict(LR_cv_model3, newdata = test_cleaned_mice3Data)
LR_predict_cv_model4 = predict(LR_cv_model4, newdata = test_cleaned_mice4Data)
LR_predict_cv_model5 = predict(LR_cv_model5, newdata = test_cleaned_mice5Data)

RMSE_LR_cv_model1 = RMSE(LR_predict_cv_model1, training_cleaned_mice1Data$site_eui[0:9705])
RMSE_LR_cv_model2 = RMSE(LR_predict_cv_model2, training_cleaned_mice2Data$site_eui[0:9705])
RMSE_LR_cv_model3 = RMSE(LR_predict_cv_model3, training_cleaned_mice3Data$site_eui[0:9705])
RMSE_LR_cv_model4 = RMSE(LR_predict_cv_model4, training_cleaned_mice4Data$site_eui[0:9705])
RMSE_LR_cv_model5 = RMSE(LR_predict_cv_model5, training_cleaned_mice5Data$site_eui[0:9705])
RMSE_LR_cv_model1
RMSE_LR_cv_model2
RMSE_LR_cv_model3
RMSE_LR_cv_model4
RMSE_LR_cv_model5
par(mar=c(2,2,2,2))
plot.new()
par(mfrow=c(2,2))
plot(LR_model1) #best model
avPlots(LR_model1) #best model
bptest(LR_model1)



LASSO
Lasso_y1 = training_cleaned_mice1Data$site_eui
Lasso_x1 = data.matrix(training_cleaned_mice1Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Lasso_y2 = training_cleaned_mice2Data$site_eui
Lasso_x2 = data.matrix(training_cleaned_mice2Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Lasso_y3 = training_cleaned_mice3Data$site_eui
Lasso_x3 = data.matrix(training_cleaned_mice3Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Lasso_y4 = training_cleaned_mice4Data$site_eui
Lasso_x4 = data.matrix(training_cleaned_mice4Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Lasso_y5 = training_cleaned_mice5Data$site_eui
Lasso_x5 = data.matrix(training_cleaned_mice5Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Lasso_new1 = data.matrix(test_cleaned_mice1Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Lasso_new2 = data.matrix(test_cleaned_mice2Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Lasso_new3 = data.matrix(test_cleaned_mice3Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Lasso_new4 = data.matrix(test_cleaned_mice4Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Lasso_new5 = data.matrix(test_cleaned_mice5Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])

Lasso_model1 = glmnet(Lasso_x1, Lasso_y1, alpha = 0)
Lasso_model2 = glmnet(Lasso_x2, Lasso_y2, alpha = 0)
Lasso_model3 = glmnet(Lasso_x3, Lasso_y3, alpha = 0)
Lasso_model4 = glmnet(Lasso_x4, Lasso_y4, alpha = 0)
Lasso_model5 = glmnet(Lasso_x5, Lasso_y5, alpha = 0)
Lasso_predict_model1 = predict(Lasso_model1, newx = Lasso_new1, alpha = 0)
Lasso_predict_model2 = predict(Lasso_model2, newx = Lasso_new2, alpha = 0)
Lasso_predict_model3 = predict(Lasso_model3, newx = Lasso_new3, alpha = 0)
Lasso_predict_model4 = predict(Lasso_model4, newx = Lasso_new4, alpha = 0)
Lasso_predict_model5 = predict(Lasso_model5, newx = Lasso_new5, alpha = 0)
RMSE_Lasso_model1 = RMSE(Lasso_predict_model1, Lasso_y1[0:9705])
RMSE_Lasso_model2 = RMSE(Lasso_predict_model2, Lasso_y2[0:9705])
RMSE_Lasso_model3 = RMSE(Lasso_predict_model3, Lasso_y3[0:9705])
RMSE_Lasso_model4 = RMSE(Lasso_predict_model4, Lasso_y4[0:9705])
RMSE_Lasso_model5 = RMSE(Lasso_predict_model5, Lasso_y5[0:9705])
RMSE_Lasso_model1 
RMSE_Lasso_model2 
RMSE_Lasso_model3 
RMSE_Lasso_model4 
RMSE_Lasso_model5

Lasso_cv_model1 = cv.glmnet(Lasso_x1, Lasso_y1, alpha = 1)
Lasso_best_lambda1 = Lasso_cv_model1$lambda.min
Lasso_best_lambda1
plot(Lasso_cv_model1)
Lasso_best_model1 = glmnet(Lasso_x1, Lasso_y1, alpha = 1, lambda = Lasso_best_lambda1)
coef(Lasso_best_model1)

Lasso_cv_model2 = cv.glmnet(Lasso_x2, Lasso_y2, alpha = 1)
Lasso_best_lambda2 = Lasso_cv_model2$lambda.min
Lasso_best_lambda2
plot(Lasso_cv_model2)
Lasso_best_model2 = glmnet(Lasso_x1, Lasso_y2, alpha = 1, lambda = Lasso_best_lambda2)
coef(Lasso_best_model2)

Lasso_cv_model3 = cv.glmnet(Lasso_x3, Lasso_y3, alpha = 1)
Lasso_best_lambda3 = Lasso_cv_model3$lambda.min
Lasso_best_lambda3
plot(Lasso_cv_model3)
Lasso_best_model3 = glmnet(Lasso_x3, Lasso_y3, alpha = 1, lambda = Lasso_best_lambda3)
coef(Lasso_best_model3)

Lasso_cv_model4 = cv.glmnet(Lasso_x4, Lasso_y4, alpha = 1)
Lasso_best_lambda4 = Lasso_cv_model4$lambda.min
Lasso_best_lambda4
plot(Lasso_cv_model4)
Lasso_best_model4 = glmnet(Lasso_x4, Lasso_y4, alpha = 1, lambda = Lasso_best_lambda4)
coef(Lasso_best_model4)

Lasso_cv_model5 = cv.glmnet(Lasso_x5, Lasso_y5, alpha = 1)
Lasso_best_lambda5 = Lasso_cv_model5$lambda.min
Lasso_best_lambda5
plot(Lasso_cv_model5)
Lasso_best_model5 = glmnet(Lasso_x5, Lasso_y5, alpha = 1, lambda = Lasso_best_lambda5)
coef(Lasso_best_model5)


Lasso_predicted1 = predict(Lasso_best_model1, s = Lasso_best_lambda1, newx = Lasso_new1)
Ly2 = Lasso_y1[0:9705]
Lasso_predict_RMSE1 = RMSE(Lasso_predicted1, Ly2)
Lasso_predicted2 = predict(Lasso_best_model2, s = Lasso_best_lambda2, newx = Lasso_new2)
Ly2 = Lasso_y2[0:9705]
Lasso_predict_RMSE2 = RMSE(Lasso_predicted2, Ly2)
Lasso_predicted3 = predict(Lasso_best_model3, s = Lasso_best_lambda3, newx = Lasso_new3)
Ly2 = Lasso_y3[0:9705]
Lasso_predict_RMSE3 = RMSE(Lasso_predicted3, Ly2)
Lasso_predicted4 = predict(Lasso_best_model4, s = Lasso_best_lambda4, newx = Lasso_new4)
Ly2 = Lasso_y4[0:9705]
Lasso_predict_RMSE4 = RMSE(Lasso_predicted4, Ly2)
Lasso_predicted5 = predict(Lasso_best_model5, s = Lasso_best_lambda5, newx = Lasso_new5)
Ly2 = Lasso_y5[0:9705]
Lasso_predict_RMSE5 = RMSE(Lasso_predicted5, Ly2)
Lasso_predict_RMSE1
Lasso_predict_RMSE2
Lasso_predict_RMSE3
Lasso_predict_RMSE4
Lasso_predict_RMSE5
plot(Lasso_model1, xvar = "lambda", label=T, main="Lasso Coefficients")
abline(v=Lasso_cv_model4$lambda.min, col = "red", lty=2)
abline(v=Lasso_cv_model4$lambda.1se, col="blue", lty=2)




#RIDGE 
Ridge_y1 = training_cleaned_mice1Data$site_eui
Ridge_x1 = data.matrix(training_cleaned_mice1Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Ridge_y2 = training_cleaned_mice2Data$site_eui
Ridge_x2 = data.matrix(training_cleaned_mice2Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Ridge_y3 = training_cleaned_mice3Data$site_eui
Ridge_x3 = data.matrix(training_cleaned_mice3Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Ridge_y4 = training_cleaned_mice4Data$site_eui
Ridge_x4 = data.matrix(training_cleaned_mice4Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Ridge_y5 = training_cleaned_mice5Data$site_eui
Ridge_x5 = data.matrix(training_cleaned_mice5Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Ridge_new1 = data.matrix(test_cleaned_mice1Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Ridge_new2 = data.matrix(test_cleaned_mice2Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Ridge_new3 = data.matrix(test_cleaned_mice3Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Ridge_new4 = data.matrix(test_cleaned_mice4Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Ridge_new5 = data.matrix(test_cleaned_mice5Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
Ridge_model1 = glmnet(Ridge_x1, Ridge_y1, alpha = 0)
Ridge_model2 = glmnet(Ridge_x2, Ridge_y2, alpha = 0)
Ridge_model3 = glmnet(Ridge_x3, Ridge_y3, alpha = 0)
Ridge_model4 = glmnet(Ridge_x4, Ridge_y4, alpha = 0)
Ridge_model5 = glmnet(Ridge_x5, Ridge_y5, alpha = 0)
Ridge_predict_model1 = predict(Ridge_model1, newx = Ridge_new1, alpha = 0)
Ridge_predict_model2 = predict(Ridge_model2, newx = Ridge_new2, alpha = 0)
Ridge_predict_model3 = predict(Ridge_model3, newx = Ridge_new3, alpha = 0)
Ridge_predict_model4 = predict(Ridge_model4, newx = Ridge_new4, alpha = 0)
Ridge_predict_model5 = predict(Ridge_model5, newx = Ridge_new5, alpha = 0)
RMSE_Ridge_model1 = RMSE(Ridge_predict_model1, Ridge_y1[0:9705])
RMSE_Ridge_model2 = RMSE(Ridge_predict_model2, Ridge_y2[0:9705])
RMSE_Ridge_model3 = RMSE(Ridge_predict_model3, Ridge_y3[0:9705])
RMSE_Ridge_model4 = RMSE(Ridge_predict_model4, Ridge_y4[0:9705])
RMSE_Ridge_model5 = RMSE(Ridge_predict_model5, Ridge_y5[0:9705])
RMSE_Ridge_model1 
RMSE_Ridge_model2 
RMSE_Ridge_model3 
RMSE_Ridge_model4 
RMSE_Ridge_model5

Ridge_cv_model1 = cv.glmnet(Ridge_x1, Ridge_y1, alpha = 0)
Ridge_best_lambda1 = Ridge_cv_model1$lambda.min
Ridge_best_lambda1
plot(Ridge_cv_model1)
Ridge_best_model1 = glmnet(Ridge_x1, Ridge_y1, alpha = 0, lambda = Ridge_best_lambda1)
coef(Ridge_best_model1)

Ridge_cv_model2 = cv.glmnet(Ridge_x2, Ridge_y2, alpha = 0)
Ridge_best_lambda2 = Ridge_cv_model2$lambda.min
Ridge_best_lambda2
plot(Ridge_cv_model2)
Ridge_best_model2 = glmnet(Ridge_x2, Ridge_y2, alpha = 0, lambda = Ridge_best_lambda2)
coef(Ridge_best_model1)

Ridge_cv_model3 = cv.glmnet(Ridge_x3, Ridge_y3, alpha = 0)
Ridge_best_lambda3 = Ridge_cv_model3$lambda.min
Ridge_best_lambda3
plot(Ridge_cv_model3)
Ridge_best_model3 = glmnet(Ridge_x3, Ridge_y3, alpha = 0, lambda = Ridge_best_lambda3)
coef(Ridge_best_model3)

Ridge_cv_model4 = cv.glmnet(Ridge_x4, Ridge_y4, alpha = 0)
Ridge_best_lambda4 = Ridge_cv_model4$lambda.min
Ridge_best_lambda4
plot(Ridge_cv_model4)
Ridge_best_model4 = glmnet(Ridge_x4, Ridge_y4, alpha = 0, lambda = Ridge_best_lambda4)
coef(Ridge_best_model4)

Ridge_cv_model5 = cv.glmnet(Ridge_x5, Ridge_y5, alpha = 0)
Ridge_best_lambda5 = Ridge_cv_model5$lambda.min
Ridge_best_lambda5
plot(Ridge_cv_model5)
Ridge_best_model5 = glmnet(Ridge_x5, Ridge_y5, alpha = 0, lambda = Ridge_best_lambda5)
coef(Ridge_best_model5)

Ridge_predicted1 = predict(Ridge_best_model1, s = Ridge_best_lambda1, newx = Ridge_new1)
Ry2 = Ridge_y1[0:9705]
Ridge_predict_RMSE1 = RMSE(Ridge_predicted1, Ry2)
Ridge_predicted2 = predict(Ridge_best_model2, s = Ridge_best_lambda2, newx = Ridge_new2)
Ry2 = Ridge_y2[0:9705]
Ridge_predict_RMSE2 = RMSE(Ridge_predicted2, Ry2)
Ridge_predicted3 = predict(Ridge_best_model3, s = Ridge_best_lambda3, newx = Ridge_new3)
Ry2 = Ridge_y3[0:9705]
Ridge_predict_RMSE3 = RMSE(Ridge_predicted3, Ry2)
Ridge_predicted4 = predict(Ridge_best_model4, s = Ridge_best_lambda4, newx = Ridge_new4)
Ry2 = Ridge_y4[0:9705]
Ridge_predict_RMSE4 = RMSE(Ridge_predicted4, Ry2)
Ridge_predicted5 = predict(Ridge_best_model5, s = Ridge_best_lambda5, newx = Ridge_new5)
Ry2 = Ridge_y5[0:9705]
Ridge_predict_RMSE5 = RMSE(Ridge_predicted5, Ry2)
Ridge_predict_RMSE1
Ridge_predict_RMSE2
Ridge_predict_RMSE3
Ridge_predict_RMSE4
Ridge_predict_RMSE5
plot(Ridge_model5, xvar = "lambda", label=T, main = "Ridge Coefficients")
abline(v=Ridge_cv_model5$lambda.min, col = "red", lty=2)
abline(v=Ridge_cv_model5$lambda.1se, col="blue", lty=2)


#XGBoost 
XGBoost_y1 = training_cleaned_mice1Data$site_eui
XGBoost_x1 = data.matrix(training_cleaned_mice1Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
XGBoost_y2 = training_cleaned_mice2Data$site_eui
XGBoost_x2 = data.matrix(training_cleaned_mice2Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
XGBoost_y3 = training_cleaned_mice3Data$site_eui
XGBoost_x3 = data.matrix(training_cleaned_mice3Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
XGBoost_y4 = training_cleaned_mice4Data$site_eui
XGBoost_x4 = data.matrix(training_cleaned_mice4Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
XGBoost_y5 = training_cleaned_mice5Data$site_eui
XGBoost_x5 = data.matrix(training_cleaned_mice5Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
XGBoost_new1 = data.matrix(test_cleaned_mice1Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
XGBoost_new2 = data.matrix(test_cleaned_mice2Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
XGBoost_new3 = data.matrix(test_cleaned_mice3Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
XGBoost_new4 = data.matrix(test_cleaned_mice5Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])
XGBoost_new5 = data.matrix(test_cleaned_mice5Data[, c("building_class", "facility_type", "floor_area", "year_built", "energy_star_rating", "january_avg_temp", "february_avg_temp", "march_avg_temp", "april_avg_temp", "may_avg_temp", "june_avg_temp", "july_avg_temp", "august_avg_temp", "september_avg_temp", "october_avg_temp",  "november_avg_temp", "december_avg_temp", "cooling_degree_days", "heating_degree_days", "avg_temp", "days_below_30F", "days_below_20F", "days_below_10F", "days_below_0F", "days_above_80F", "days_above_90F", "days_above_100F", "days_above_110F")])

#Parameters
eta=c(0.01, 0.02, 0.03, 0.04, 0.05, 0.1, 0.2, 0.5, 1)
cs=c(1/8, 1/4, 1/3, 2/3, 1)
md=c(2, 3, 4, 5, 6)
standard=c(2,2,3)

#ETA XGB_cv_eta_model1 
conv_eta = matrix(NA,500,length(eta)) 
pred_eta = matrix(NA,length(XGBoost_new1), length(eta)) 
colnames(conv_eta) = colnames(pred_eta) = eta 
for(i in 1:length(eta)){
params=list(eta = eta[i], colsample_bylevel=cs[standard[2]],
subsample = .35, max_depth = md[standard[3]],
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_eta_model1 =xgboost(XGBoost_x1, label = XGBoost_y1, nrounds = 500, params = params)
conv_eta[,i] = XGB_cv_eta_model1$evaluation_log$train_rmse
pred_eta[,i] = predict(XGB_cv_eta_model1, XGBoost_new1)
} 
conv_eta = data.frame(iter=1:500, conv_eta) 
conv_eta = melt(conv_eta, id.vars = "iter") 
ggplot(data = conv_eta) + geom_line(aes(x = iter, y = value, color = variable))
RMSE_eta_model1 = sqrt(colMeans((XGBoost_y1[0:9705]-pred_eta)^2))
RMSE_eta_model1

#ColSAMPLE XGB_cv_cs_model1
conv_cs = matrix(NA,500,length(cs))
pred_cs = matrix(NA,length(XGBoost_new1), length(cs))
colnames(conv_cs) = colnames(pred_cs) = cs
for(i in 1:length(cs)){
params = list(eta = eta[standard[1]], colsample_bylevel = cs[i],
subsample = .35, max_depth = md[standard[3]], 
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_cs_model1 =xgboost(XGBoost_x1, label = XGBoost_y1,
 nrounds = 500, params = params)
conv_cs[,i] = XGB_cv_cs_model1$evaluation_log$train_rmse
pred_cs[,i] = predict(XGB_cv_cs_model1, XGBoost_new1)
}
conv_cs = data.frame(iter=1:500, conv_cs)
conv_cs = melt(conv_cs, id.vars = "iter")
ggplot(data = conv_cs) + geom_line(aes(x = iter, y = value, color = variable))
RMSE_cs_model1 = sqrt(colMeans((XGBoost_y1[0:9705] -pred_cs)^2))
RMSE_cs_model1


#MAXDEPTH XGB_cv_md_model1
conv_md=matrix(NA,500,length(md))
pred_md=matrix(NA,length(XGBoost_new1),length(md))
colnames(conv_md)=colnames(pred_md)=md
for(i in 1:length(md)){
params=list(eta=eta[standard[1]],colsample_bylevel=cs[standard[2]],
subsample=.35,max_depth=md[i],
min_child_weigth=1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_md_model1 = xgboost(XGBoost_x1, label = XGBoost_y1, nrounds = 500, params=params)
conv_md[,i] = XGB_cv_md_model1$evaluation_log$train_rmse
pred_md[,i] = predict(XGB_cv_md_model1, XGBoost_new1)
}
conv_md=data.frame(iter=1:500, conv_md)
conv_md=melt(conv_md,id.vars = "iter")
ggplot(data=conv_md)+geom_line(aes(x=iter,y=value,color=variable))
RMSE_md_model1 = sqrt(colMeans((XGBoost_y1[0:9705] -pred_cs)^2))
RMSE_md_model1

#Final Model XGB_cv_final_model1
params=list(eta=.01, colsample_bylevel=.125,
subsample= .35, max_depth=2,
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma = 20, lambda=5, objective ="reg:squarederror", nfold = 10)
XGB_cv_final_model1 = xgboost(XGBoost_x1, label = XGBoost_y1, nrounds = 500, params=params)
pred_final_model1 = predict(XGB_cv_final_model1, XGBoost_new1)
RMSE_XGB_cv_final_model1 = RMSE(XGBoost_y1[0:9705], pred_final_model1)
RMSE_XGB_cv_final_model1

#ETA XGB_cv_eta_model2 
conv_eta = matrix(NA,500,length(eta)) 
pred_eta = matrix(NA,length(XGBoost_new2), length(eta)) 
colnames(conv_eta) = colnames(pred_eta) = eta 
for(i in 1:length(eta)){
params=list(eta = eta[i], colsample_bylevel=cs[standard[2]],
subsample = .35, max_depth = md[standard[3]],
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_eta_model2 =xgboost(XGBoost_x2, label = XGBoost_y2, nrounds = 500, params = params)
conv_eta[,i] = XGB_cv_eta_model2$evaluation_log$train_rmse
pred_eta[,i] = predict(XGB_cv_eta_model2, XGBoost_new2)
} 
conv_eta = data.frame(iter=1:500, conv_eta) 
conv_eta = melt(conv_eta, id.vars = "iter") 
ggplot(data = conv_eta) + geom_line(aes(x = iter, y = value, color = variable))
RMSE_eta_model2 = sqrt(colMeans((XGBoost_y2[0:9705]-pred_eta)^2))
RMSE_eta_model2

#ColSAMPLE XGB_cv_cs_model2
conv_cs = matrix(NA,500,length(cs))
pred_cs = matrix(NA,length(XGBoost_new2), length(cs))
colnames(conv_cs) = colnames(pred_cs) = cs
for(i in 1:length(cs)){
params = list(eta = eta[standard[1]], colsample_bylevel = cs[i],
subsample = .35, max_depth = md[standard[3]], 
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_cs_model2 =xgboost(XGBoost_x2, label = XGBoost_y2,
 nrounds = 500, params = params)
conv_cs[,i] = XGB_cv_cs_model2$evaluation_log$train_rmse
pred_cs[,i] = predict(XGB_cv_cs_model2, XGBoost_new2)
}
conv_cs = data.frame(iter=1:500, conv_cs)
conv_cs = melt(conv_cs, id.vars = "iter")
ggplot(data = conv_cs) + geom_line(aes(x = iter, y = value, color = variable))
RMSE_cs_model2 = sqrt(colMeans((XGBoost_y2[0:9705] - pred_cs)^2))
RMSE_cs_model2


#MAXDEPTH XGB_cv_md_model2
conv_md=matrix(NA,500,length(md))
pred_md=matrix(NA,length(XGBoost_new2),length(md))
colnames(conv_md)=colnames(pred_md)=md
for(i in 1:length(md)){
params=list(eta=eta[standard[1]],colsample_bylevel=cs[standard[2]],
subsample=.35, max_depth=md[i],
min_child_weigth=1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_md_model2 = xgboost(XGBoost_x2, label = XGBoost_y2, nrounds = 500, params=params)
conv_md[,i] = XGB_cv_md_model2$evaluation_log$train_rmse
pred_md[,i] = predict(XGB_cv_md_model2, XGBoost_new2)
}
conv_md=data.frame(iter=1:500, conv_md)
conv_md=melt(conv_md,id.vars = "iter")
ggplot(data=conv_md)+geom_line(aes(x=iter,y=value,color=variable))
RMSE_md_model2=sqrt(colMeans((XGBoost_y2[0:9705] - pred_md)^2))
RMSE_md_model2

#Final Model XGB_cv_model2
params=list(eta=.01, colsample_bylevel=.125,
subsample= .35, max_depth=2,
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma = 20, lambda=5, objective ="reg:squarederror", nfold = 10)
XGB_cv_final_model2 = xgboost(XGBoost_x2, label = XGBoost_y2, nrounds = 500, params=params)
pred_final_model2 = predict(XGB_cv_final_model2, XGBoost_new2)
RMSE_XGB_cv_final_model2 = RMSE(XGBoost_y2[0:9705], pred_final_model2)
RMSE_XGB_cv_final_model2


#ETA XGB_cv_eta_model3 
conv_eta = matrix(NA,500,length(eta)) 
pred_eta = matrix(NA,length(XGBoost_new3), length(eta)) 
colnames(conv_eta) = colnames(pred_eta) = eta 
for(i in 1:length(eta)){
params=list(eta = eta[i], colsample_bylevel=cs[standard[2]],
subsample = .35, max_depth = md[standard[3]],
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_eta_model3 =xgboost(XGBoost_x3, label = XGBoost_y3, nrounds = 500, params = params)
conv_eta[,i] = XGB_cv_eta_model3$evaluation_log$train_rmse
pred_eta[,i] = predict(XGB_cv_eta_model3, XGBoost_new3)
} 
conv_eta = data.frame(iter=1:500, conv_eta) 
conv_eta = melt(conv_eta, id.vars = "iter") 
ggplot(data = conv_eta) + geom_line(aes(x = iter, y = value, color = variable))
RMSE_eta_model3 = sqrt(colMeans((XGBoost_y3[0:9705]-pred_eta)^2))
RMSE_eta_model3

#ColSAMPLE XGB_cv_cs_model3
conv_cs = matrix(NA,500,length(cs))
pred_cs = matrix(NA,length(XGBoost_new3), length(cs))
colnames(conv_cs) = colnames(pred_cs) = cs
for(i in 1:length(cs)){
params = list(eta = eta[standard[1]], colsample_bylevel = cs[i],
subsample = .35, max_depth = md[standard[3]], 
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_cs_model3 =xgboost(XGBoost_x3, label = XGBoost_y3,
 nrounds = 500, params = params)
conv_cs[,i] = XGB_cv_cs_model3$evaluation_log$train_rmse
pred_cs[,i] = predict(XGB_cv_cs_model3, XGBoost_new3)
}
conv_cs = data.frame(iter=1:500, conv_cs)
conv_cs = melt(conv_cs, id.vars = "iter")
ggplot(data = conv_cs) + geom_line(aes(x = iter, y = value, color = variable))
RMSE_cs_model3= sqrt(colMeans((XGBoost_y3[0:9705] - pred_cs)^2))
RMSE_cs_model3


#MAXDEPTH XGB_cv_md_model3
conv_md=matrix(NA,500,length(md))
pred_md=matrix(NA,length(XGBoost_new3),length(md))
colnames(conv_md)=colnames(pred_md)=md
for(i in 1:length(md)){
params=list(eta=eta[standard[1]],colsample_bylevel=cs[standard[2]],
subsample=.35, max_depth=md[i],
min_child_weigth=1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_md_model3 = xgboost(XGBoost_x3, label = XGBoost_y3, nrounds = 500, params=params)
conv_md[,i] = XGB_cv_md_model3$evaluation_log$train_rmse
pred_md[,i] = predict(XGB_cv_md_model3, XGBoost_new3)
}
conv_md=data.frame(iter=1:500, conv_md)
conv_md=melt(conv_md,id.vars = "iter")
ggplot(data=conv_md)+geom_line(aes(x=iter,y=value,color=variable))
RMSE_md_model3=sqrt(colMeans((XGBoost_y3[0:9705] - pred_md)^2))
RMSE_md_model3

#Final Model XGB_cv_model3
params=list(eta=.01, colsample_bylevel=.125,
subsample= .35, max_depth=2,
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma = 20, lambda=5, objective ="reg:squarederror", nfold = 10)
XGB_cv_final_model3 = xgboost(XGBoost_x3, label = XGBoost_y3, nrounds = 500, params=params)
pred_final_model3 = predict(XGB_cv_final_model3, XGBoost_new3)
RMSE_XGB_cv_final_model3 = RMSE(XGBoost_y3[0:9705], pred_final_model3)
RMSE_XGB_cv_final_model3

#ETA XGB_cv_eta_model4 
conv_eta = matrix(NA,500,length(eta)) 
pred_eta = matrix(NA,length(XGBoost_new4), length(eta)) 
colnames(conv_eta) = colnames(pred_eta) = eta 
for(i in 1:length(eta)){
params=list(eta = eta[i], colsample_bylevel=cs[standard[2]],
subsample = .35, max_depth = md[standard[3]],
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_eta_model4 =xgboost(XGBoost_x4, label = XGBoost_y4, nrounds = 500, params = params)
conv_eta[,i] = XGB_cv_eta_model4$evaluation_log$train_rmse
pred_eta[,i] = predict(XGB_cv_eta_model4, XGBoost_new4)
} 
conv_eta = data.frame(iter=1:500, conv_eta) 
conv_eta = melt(conv_eta, id.vars = "iter") 
ggplot(data = conv_eta) + geom_line(aes(x = iter, y = value, color = variable))
RMSE_eta_model4 = sqrt(colMeans((XGBoost_y4[0:9705]-pred_eta)^2))
RMSE_eta_model4

#ColSAMPLE XGB_cv_cs_model4
conv_cs = matrix(NA,500,length(cs))
pred_cs = matrix(NA,length(XGBoost_new4), length(cs))
colnames(conv_cs) = colnames(pred_cs) = cs
for(i in 1:length(cs)){
params = list(eta = eta[standard[1]], colsample_bylevel = cs[i],
subsample = .35, max_depth = md[standard[3]], 
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_cs_model4 =xgboost(XGBoost_x4, label = XGBoost_y4,
 nrounds = 500, params = params)
conv_cs[,i] = XGB_cv_cs_model4$evaluation_log$train_rmse
pred_cs[,i] = predict(XGB_cv_cs_model4, XGBoost_new4)
}
conv_cs = data.frame(iter=1:500, conv_cs)
conv_cs = melt(conv_cs, id.vars = "iter")
ggplot(data = conv_cs) + geom_line(aes(x = iter, y = value, color = variable))
RMSE_cs_model4 = sqrt(colMeans((XGBoost_y4[0:9705] - pred_cs)^2))
RMSE_cs_model4


#MAXDEPTH XGB_cv_md_model4
conv_md=matrix(NA,500,length(md))
pred_md=matrix(NA,length(XGBoost_new4),length(md))
colnames(conv_md)=colnames(pred_md)=md
for(i in 1:length(md)){
params=list(eta=eta[standard[1]],colsample_bylevel=cs[standard[2]],
subsample=.35, max_depth=md[i],
min_child_weigth=1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_md_model4 = xgboost(XGBoost_x4, label = XGBoost_y4, nrounds = 500, params=params)
conv_md[,i] = XGB_cv_md_model4$evaluation_log$train_rmse
pred_md[,i] = predict(XGB_cv_md_model4, XGBoost_new4)
}
conv_md=data.frame(iter=1:500, conv_md)
conv_md=melt(conv_md,id.vars = "iter")
ggplot(data=conv_md)+geom_line(aes(x=iter,y=value,color=variable))
RMSE_md_model4=sqrt(colMeans((XGBoost_y4[0:9705] - pred_md)^2))
RMSE_md_model4

#Final Model XGB_cv_model4
params=list(eta=.01, colsample_bylevel=.125,
subsample= .35, max_depth=2,
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma = 20, lambda=5, objective ="reg:squarederror", nfold = 10)
XGB_cv_final_model4 = xgboost(XGBoost_x4, label = XGBoost_y4, nrounds = 500, params=params)
pred_final_model4 = predict(XGB_cv_final_model4, XGBoost_new4)
RMSE_XGB_cv_final_model4 = RMSE(XGBoost_y4[0:9705], pred_final_model4)
RMSE_XGB_cv_final_model4


#ETA XGB_cv_eta_model5 
conv_eta = matrix(NA,500,length(eta)) 
pred_eta = matrix(NA,length(XGBoost_new5), length(eta)) 
colnames(conv_eta) = colnames(pred_eta) = eta 
for(i in 1:length(eta)){
params=list(eta = eta[i], colsample_bylevel=cs[standard[2]],
subsample = .35, max_depth = md[standard[3]],
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_eta_model5 =xgboost(XGBoost_x5, label = XGBoost_y5, nrounds = 500, params = params)
conv_eta[,i] = XGB_cv_eta_model5$evaluation_log$train_rmse
pred_eta[,i] = predict(XGB_cv_eta_model5, XGBoost_new5)
} 
conv_eta = data.frame(iter=1:500, conv_eta) 
conv_eta = melt(conv_eta, id.vars = "iter") 
ggplot(data = conv_eta) + geom_line(aes(x = iter, y = value, color = variable))
RMSE_eta_model5 = sqrt(colMeans((XGBoost_y5[0:9705]-pred_eta)^2))
RMSE_eta_model5

#ColSAMPLE XGB_cv_cs_model5
conv_cs = matrix(NA,500,length(cs))
pred_cs = matrix(NA,length(XGBoost_new5), length(cs))
colnames(conv_cs) = colnames(pred_cs) = cs
for(i in 1:length(cs)){
params = list(eta = eta[standard[1]], colsample_bylevel = cs[i],
subsample = .35, max_depth = md[standard[3]], 
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_cs_model5 =xgboost(XGBoost_x5, label = XGBoost_y5,
 nrounds = 500, params = params)
conv_cs[,i] = XGB_cv_cs_model5$evaluation_log$train_rmse
pred_cs[,i] = predict(XGB_cv_cs_model5, XGBoost_new5)
}
conv_cs = data.frame(iter=1:500, conv_cs)
conv_cs = melt(conv_cs, id.vars = "iter")
ggplot(data = conv_cs) + geom_line(aes(x = iter, y = value, color = variable, title = "XGBoost-cv=model5 colsample grpah "))
RMSE_cs_model5 = sqrt(colMeans((XGBoost_y5[0:9705] - pred_cs)^2))
RMSE_cs_model5


#MAXDEPTH XGB_cv_md_model5
conv_md=matrix(NA,500,length(md))
pred_md=matrix(NA,length(XGBoost_new5),length(md))
colnames(conv_md)=colnames(pred_md)=md
for(i in 1:length(md)){
params=list(eta=eta[standard[1]],colsample_bylevel=cs[standard[2]],
subsample=.35, max_depth=md[i],
min_child_weigth=1, cv = 10,  n_estimators=300,
gamma=20, lambda=5, objective ="reg:squarederror")
XGB_cv_md_model5 = xgboost(XGBoost_x5, label = XGBoost_y5, nrounds = 500, params=params)
conv_md[,i] = XGB_cv_md_model5$evaluation_log$train_rmse
pred_md[,i] = predict(XGB_cv_md_model5, XGBoost_new5)
}
conv_md=data.frame(iter=1:500, conv_md)
conv_md=melt(conv_md,id.vars = "iter")
ggplot(data=conv_md)+geom_line(aes(x=iter,y=value,color=variable))
RMSE_md_model5=sqrt(colMeans((XGBoost_y5[0:9705] - pred_md)^2))
RMSE_md_model5

#Final Model XGB_cv_model5
params=list(eta=.01, colsample_bylevel=.125,
subsample= .35, max_depth=2,
min_child_weigth = 1, cv = 10,  n_estimators=300,
gamma = 20, lambda=5, objective ="reg:squarederror", nfold = 10)
XGB_cv_final_model5 = xgboost(XGBoost_x5, label = XGBoost_y5, nrounds = 500, params=params)
pred_final_model5 = predict(XGB_cv_final_model5, XGBoost_new5)
RMSE_XGB_cv_final_model5 = RMSE(XGBoost_y5[0:9705], pred_final_model5)
RMSE_XGB_cv_final_model5

eta=ggplot(data = conv_eta) + geom_line(aes(x = iter, y = value, color = variable)) 
eta + labs(title = "XGBoost_cv_model5 eta graphs")

cs=ggplot(data = conv_cs) + geom_line(aes(x = iter, y = value, color = variable)) 
cs + labs(title = "XGBoost_cv_model5 colsample graphs")

md=ggplot(data = conv_md) + geom_line(aes(x = iter, y = value, color = variable)) 
md + labs(title = "XGBoost_cv_model5 max_depth graphs")
