title: "kaggle digit recognizer之logistic regression解法"
date: 2015-09-23 18:16:02
tags: [kaggle, 机器学习]
---

## logistic regression ##
>**逻辑回归**是一种常用的分类算法（虽然名称带着回归，其实是分类），在广告行业中运用非常广泛，主要用来判断用户是否点击某个广告，以此实现广告的最佳投放效果。逻辑回归一般用来解决二分类的问题，即某件事情只有两种可能，通过训练数据得到预测数据发生某种情况的可能性，一般如果大于0.5我们认为该事件会发生，小于0.5则该事件的互斥事件会发生。

## 本题思路 ##
由于数字有10种可能，所以这是一种多分类的场景。逻辑回归在这种场景下的一般做法是：如果有n种分类，则做n次逻辑回归，每次把是该种类的视为一类，把非该种类的视为另一类，这样可以计算出n个可能性，取可能性最大的作为分类。针对本题我也是这么做的，但是效果非常不理想，正确率仅为0.80（tnn的还运行了好几个小时）。但是作为通过这次练习，大致熟悉了逻辑回归。

## 代码 ##

```
train <- read.csv("data/train.csv", header=TRUE)
test <- read.csv("data/test.csv", header=TRUE)

labels <- train[,1]
train <- train[,-1]

#create formula
xnam <- paste0("pixel", c(0:783))
formula <- as.formula(paste("lr.labels ~ ", paste(xnam, collapse = "+")))

pre <- list()
for (i in 0 : 9) {
    lr.labels <- ifelse(labels == i, 1, 0)
    logic.fit <- glm(formula = formula, data = cbind(lr.labels, train), family=binomial(link="logit"))
    p <- predict(logic.fit, test)
    pre <- cbind(pre, exp(p) / 1 + exp(p))
}

result <- max.col(pre) - 1
```

## 说明 ##
glm是R广义线性模型函数，可以处理很多种回归，通过family参数控制回归类型，主要有：
1. binomal(link='logit')         ----响应变量服从二项分布，连接函数为logit，即logistic回归
2. binomal(link='probit')       ----响应变量服从二项分布，连接函数为probit
3. poisson(link='identity')     ----响应变量服从泊松分布，即泊松回归

还有是对R的formula不熟悉，最后参考[这里](http://site.douban.com/182577/widget/notes/10567181/note/318916395/)算是理解了吧。

## 参考 ##
1. [逻辑回归](http://blog.csdn.net/pakko/article/details/37878837)
1. [R广义线性模型](http://www.cnblogs.com/runner-ljt/p/4574275.html)
2. [R中的formula与Formula](http://site.douban.com/182577/widget/notes/10567181/note/318916395/)


