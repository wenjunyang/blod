title: "kaggle digit recognizer之PCA+KNN解法"
date: 2015-09-16 11:45:27
tags: [kaggle, 机器学习]
---

## 背景 ##
	    一直想通过比赛练习什么的加强对机器学习算法的理解（工作都没用到什么机器学习算法），最近发现[kaggele](https://www.kaggle.com/)是一个不错网站。就找了上面一道名Digit Recognizer习题练手，借鉴多方观点，决定先尝试pca+knn方法（主要是我觉得比较简单利于理解）。以后再试试逻辑回归、SVM、贝叶斯、deep learning吧。

## 思路 ##
	    思路其实很简单了，KNN是一个比较容易理解的分类算法。它的原理说起来也很简单，就是待分类数据的k个已经分好类的邻居，k个邻居中出现最多的就作为它的分类了。其实knn也可以用来做回归，只是不采用投票的方式，而是取k个邻居的均值，更准确点是按距离加权的中值。那么怎么衡量两个数据间的距离呢，算法有很多种，最常用的就是欧式距离拉.那么回到本题，每个图片其实是由28*28个像素组成，即数据的维度是784，如果直接上knn，由于维度太大，计算量也相当高了。目测普通单个机器得个一天吧。那么就可以使用到pca降维了。我选择pca降维度降到64，单机下几分钟就出结果了。提交几次终于成功了，正确率0.97186，二百多名，好开森，知足常乐，哈哈。代码就很简单了，下面贴出来：

## 代码 ##

```
library(class)

train <- read.csv("train.csv", header=TRUE)
test <- read.csv("test.csv", header=TRUE)

train <- as.matrix(train)
test <- as.matrix(test)

labels <- as.numeric(train[,1])
train <- train[,-1]

all <- rbind(train, test)

//计算主成分
pca <- princomp(all)

//对训练数据降维转换
train <- train %*% pca$loadings[,1:64]
//对测试数据降维转换
test <- test %*% pca$loadings[,1:64]

//pre即是最终结果
pre <- knn(train, test, labels, 9)	
```

## 细节 ##
	第一次直接提交，报错了，格式错误。试题明明写的每行只需写出结果就行。搜了下才发现提交的结果要是csv格式的。第一行为固定标题，然后是数据，第一列id，第二列识别结果，id从1递增。比如：
	ImageId,Label
	1,2
	2,0
	3,9
	4,9
	...

## 排名截图 ##
![](/images/kaggle_first.png)

----------
