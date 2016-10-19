### Linear Regression

	# check r-squared value. It has to be maximized for it be a good model. also check significance of variables. Remove
	# insignificant variables. Two tied insignificant vars can be removed by checking pr(>|t|) value. remove
	# insignificant vars with high pr(>|t|) value. check sign of estimate values of independent vars in summary(model)
	# and see if that makes sense. check for multicollinearity among independent vars(>0.7) and remove accordingly one of the two.

	# model

		model <- lm(y ~ x, data = train)
		
		library(Mglmnet)
		model = glmnet(y ~ x,alpha = , lambda = seq(0, 100, 0.1)) # alpha = 0 for ridge, 1 for lasso
		
		pred <- predict(model, test)

	# calc RMSE and R-squared

		sse <- sum((pred - test$target)^2)
		sst <- sum((mean(train$target) - test$target) ^ 2)
		r-squared <- 1-(sse/sst)
		rmse <- sqrt(sse/nrow(data_frame))

	# ROCR and AUC(area under curve)
		
		library("ROCR")
		pred <- predict(model, test)
		ROCR.predict <- prediction(pred, test$target)
		auc <- as.numeric(performance(ROCR.predict,"auc")@y.values)

	# step model

		step.model <- step(model)

==============================================================================================================================

### Logistic Regression
	
	# check for VIC. VIC to be minimized.

	# model

		model <- glm(y ~ x, data = train, family = "binomial")
		pred <- predict(model, test, type = "response")
	
	# confusion matrix for accuracy check
	
		table(test$target, pred>threshold)
	
	# ROCR and AUC(area under curve)
	
		library("ROCR")
		pred <- predict(model, test)
		ROCR.predict <- prediction(pred, test$target)
		auc <- as.numeric(performance(ROCR.predict,"auc")@y.values)

==============================================================================================================================

### Neural Networks
	
	# min-max normalization

		maxs <- apply(data, 2, max) 
		mins <- apply(data, 2, min)
		scaled <- as.data.frame(scale(data_frame, center = mins, scale = maxs - mins))
		train_normalized
		test_normalized

	# model

		library(neuralnet)

		nn <- neuralnet(y ~ x, data = train_normalized, hidden=c(5,3), linear.output=T)
		# hidden specifies the neurons in hidden layers.
		# linear.output = T is for linear regression. It is set to F for classification.
		
		plot(nn)
		nn_predict <- compute(nn, test_normalized[,1:13])

		# de-normalizing the predictions and test set to calculate the accuracy and not to forget to submit denormalized predictions.
		
		nn_predict_denormalized <- nn_predict$net.result*(max(data$target)-min(data$target))+min(data$target)
		test_denormalized <- (test_normalized$target)*(max(data$target)-min(data$target))+min(data$target)
		rmse

==============================================================================================================================

### SVM

		library(e1071)

		svm_model <- svm(target ~ ., data = train, kernel = "radial", cost = 1, gamma = 0.1)
		svm_predictions <- predict(svm_model, test)
		table(test$target, svm_predictions)

==============================================================================================================================

### Decision Trees
	
	# for classification use method = "class" in rpart and type = "class" in predict
		library("rpart")
		library("rpart.plot")
		
	# model
		
		model <- rpart(y ~ x, data = train, method = "class", minbucket/cp = ) # large min bucket=> overfitting, smaller minbucket => biasing.
		prp(model) 
		pred <- predict(model, test, type = "class")
		table(test$target, pred)

	# ROCR and AUC(area under curve)
		
		pred <- predict(model, test) # type = "class" not to be included here
		ROCR.predict <- prediction(pred[,2], test$target)
		auc <- as.numeric(performance(ROCR.predict,"auc")@y.values)

==============================================================================================================================

### Random Forest
		
		library("randomForest")
		set.seed(10)
		train$target <- as.factor(train$target)
		test$target <- as.factor(test$target)
	
	# model

		model <- randomForest(as.factor(y) ~ x, data = train, ntree = 200, nodesize = 5/25/15/20) #nodesize similar to minbucket here.
		pred <- predict(model, test)
		table(test$target, pred)

	# party-cforest
		library(party)

		cforest_model = cforest(targetVariable ~ . , data = train, controls=cforest_unbiased(ntree=1000))

		cforest_prediction = predict(cforest_model, test, OOB=TRUE, type = "response")


==============================================================================================================================

### Clustering

	data_frame -> as.matrix -> as.vector -> clustering -> dim(vector) -> image.output
	data_frame -> as.matrix -> as.vector -> test

	# after clustering we can subset the data according to the clusters(for hierarchical especially) to study number of rows in each cluster.
	# cluster1 <- subset(train , cluster == nth.cluster.number)

	# Hierarchical

		distances <- dist(movie[2:20] OR vector, method = "euclidean")
		hcluster <- hclust(distances, method = "ward.D")
		plot(hcluster)
		hclusterGroups <- cutree(cluster, k = no.of.clusters)
		hclusterGroups[index.of.var.to.see.which.cluster.it.belongs.to]

	# K-Means

		library("flesclust")
		set.seed(1)
		kcluster <- kmeans(vector OR data_frame, centers = no.of.centroids, iter.max = no.of.max.iterations)
		str(kcluster)
		
		# rest of this for cluster-then-predict
		# in cluster-then-predict problems, (remove target var->normalize(optional)->cluster->kcca)=>build "k" train and test sets using subset from original train according to clusters.
		# example -> newTrain1 = subset(originalTrain, kclusterPredictTrain == 1), newTrain2 = subset(originalTrain, kclusterPredictTest == 2)
		
		kclusterKcca <- as.kcca(kcluster, originalDataFrame OR originalVector)
		kclusterPredictTrain <- predict(kclusterKcca)
		kclusterPredictTest <- predict(kclusterKcca, newdata = test.data.as.vector)

		#easy way to split according to clusters in k-means
		
		KmeansCluster = split(data_frame, kcluster$cluster) # KmeansCluster[[1]] and so on to access 1st and other successive clusters

==============================================================================================================================

### Ensembling

	# 1. Split data into train, cv and test. Create multiple models using different ml algorithms on train set.
	# 2. Make a data frame ensemble_train which includes predictions of these multiple models (on cv) in each column along with the cv target variable.
	# ex: suppose model1 and model2 predict pred1_cv and pred2_cv on cv set then this ensemble_train contains pred1_cv,pred2_cv,cv_targetVariable.

	# 3. Now make predictions on test set using each of these models, say pred_test1, pred_test2 and make a data frame called ensemble_test where these are the columns.
	# 4. Now make a model ensemble_model which is built using ensemble_train and predicts on ensemble_test. This is the final prediction.

==============================================================================================================================

### K-fold cross validation
		
		# to calculate optimum value of cp for using it in CART.

		library(caret)
		set.seed(1)
		fitControl = trainControl( method = "cv", number = 10 )
		cartGrid = expand.grid( .cp = seq(0.002,0.1,0.002))
		train( targetVariable ~ . , data = train, method = "rpart", trControl = fitControl, tuneGrid = cartGrid )

==============================================================================================================================

### Splitting data set randomly

		# sample.split balacnes partitions keeping in mind the outcome variable

		library("caTools")
		set.seed(10)
		split <- sample.split(data_frame$target, splitRatio = 0.75)
		train <- subset(data_frame,split == TRUE)
		test <- subset(data_frame, split == FALSE)

==============================================================================================================================

### Multiple imputation- Filling NA's randomly

		# To be run only on variables having missing value. For convinience run on every variable except for dependent variable.
		
		library("mice")
		set.seed(10)
		imputed <- complete(mice(data_frame))

==============================================================================================================================

### Removing Variables from data frame
		
		nonvars <- c("x1","x2","x3")
		train <- train[, !(names(data_frame) %in% nonvars)]
		test <- test[, !(names(data_frame) %in% nonvars)]
					
					OR

		data_frame$var <- NULL

					OR
		new_data_frame <- setdiff(names(data_frame),c("x1","x2","x3"))

==============================================================================================================================

### Plotting
	
	plot(x, y, xlab = , ylab = , main = , col = )
	hist(x, xlab = , xlim = c(low.limit, up.limit), breaks = 100)
	boxplot(y ~ x, xlab = , ylab = , main = )

==============================================================================================================================

### Misc
	
	which.max(var)
	which.min(var)
	outliers <- subset(data_frame, data_frame$var >=< condition)
	tapply(function.applied.to.this.var, result.displayed.according.to.this.var, function)
	match("value",var)
	which(var == "value")

	# choosing x random rows from a data set. given that nrow(train > x)
	trainSmall <- train[sample(nrow(train), x), ]

==============================================================================================================================

### Normalization 
	
		# this prevents to dominate vars with low values by the vars with high values. It sets the mean to 0 and SD to 1.

		library(caret)

		preproc <- preProcess(Train)

		normTrain <- predict(preproc, Train)

		normTest <- predict(preproc, Test)

==============================================================================================================================

### One-Hot Encoding

		library(caret)

		dummies <- dummyVars(targetVariable ~ ., data = data_frame)
		temp <- as.data.frame(predict(dummies, data_frame))

==============================================================================================================================

### Correlation matrix
	
	library(corrplot)

		nums <- sapply(data_frame, is.numeric)
		cordata<-data_frame[,nums]
		cordata <-na.omit(cordata)
		cor_matrix <- cor(cordata) # to see correlation table
		corrplot(cor_matrix, method = "square", type = "lower") # to visualize correlation matrix

==============================================================================================================================

### Boruta Analysis to find importance of variable in a data set
	
	library(Boruta)	

		set.seed(13)
		bor <- Boruta(targetVariable ~ ., data = train, maxRuns=101, doTrace=0)
		summary(bor)
		boruta_cor_matrix <- attStats(bor)

==============================================================================================================================

### Binning
	
		library("Hmisc")

		bins <- cut2(data_frame$variable, g = number_of_bins) # cuts the variable on basis of quantile.

==============================================================================================================================

### dplyr
		
		library("dplyr")

		select(data_frame, column_names)
		filter(data_frame, condition)
		arrange(data_frame, desc(factor_variable)) # ascending by default
		mutate(data_frame, new_variable_name = equation)
		group_by(data_frame, factor_variable)
		summarize(data_frame, function())
		count(data_frame, variable) # works similar table
		top_n(20, data_frame) # for top 20 rows

		piping : data_frame %>% operation1 %>% operation2 and so on...

==============================================================================================================================

### Date and Time
	
	data_frame$date <- strptime(variable, format = "") # format can be - "%d/%m/%Y %H:%M:%S"
	data_frame$day <- format(data_frame$date, "%d") # day
	data_frame$month <- format(data_frame$date, "%m") # month
	data_frame$year <- format(data_frame$date, "%Y") # year
	data_frame$hour <- format(data_frame$date, "%H") # hour

==============================================================================================================================

### To check for constant factors

		d <- sapply(data_frame, function(x) is.factor(x))

		m <- data_frame[,names(which(d=="TRUE"))]

		ifelse(n<-sapply(m,function(x)length(levels(x)))==1,"999999999","0")
		table(n)

==============================================================================================================================

### caret

		library("caret")
		
		fitControl = trainControl( method = "repeatedcv", number = 10, repeats = 3 )
		cartGrid = expand.grid( model specific parameters)
		model <-train(y ~ x , data = , method = "lm, glm, rpart, rf, nnet, svm, gbm", trControl = , tuneGrid = )
		predictions <- train.predict(model, test)
		confusionMatrix(predictions, test$target)

==============================================================================================================================

### GBM

		fitControl <- trainControl(method = "repeatedcv", number = 10, repeats = 10)

		gbmGrid <-  expand.grid(interaction.depth = c(1, 5, 9), n.trees = (1:30)*50, shrinkage = 0.1, n.minobsinnode = 10)

		gbmFit <- train(target ~ ., data = train,
                method = "gbm",
                trControl = fitControl,
                verbose = FALSE,
                tuneGrid = gbmGrid,
                metric = "ROC")

==============================================================================================================================

### xgboost
		
		library(xgboost)

		xgb_grid <- expand.grid(
	    eta = c(0.01, 0.001, 0.0001), 
		max_depth = c(5, 10, 15), 
	    gamma = c(1, 2, 3), 
	    colsample_bytree = c(0.4, 0.7, 1.0), 
	    min_child_weight = c(0.5, 1, 1.5),
	    nrounds = 2 

		)

		xgb_trcontrol <- trainControl(
	    method="repeatedcv",
	    number = 10,
	    repeats = 3,
	    verboseIter = TRUE,
	    returnData=FALSE,
	    returnResamp = "all",
	    allowParallel = TRUE

		)

		xgb_model <- train(x = as.matrix(data_frame %>% select(-c(Id,targetVariable))),
	    y = data_frame$targetVariable,
	    trControl = xgb_trcontrol,
	    tuneGrid = xgb_grid,
	    method="xgbTree" or "xgbLinear"
		)

		xgb_predictions_test <- predict(xgb_model, data.matrix(test))

		imp <- varImp(xgb_model)

==============================================================================================================================










==============================================================================================================================

Notes :

## If you set a random seed, split, set the seed again to the same value, and then split again, you will get the same
## split. However, if you set the seed and then split twice, you will get different splits. If you set the seed to
## different values, you will get different splits.

==============================================================================================================================





