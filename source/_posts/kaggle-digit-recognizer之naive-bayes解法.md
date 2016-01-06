title: "kaggle digit recognizer之naive bayes解法"
date: 2015-09-28 17:58:36
tags: [kaggle, 机器学习]
---

##naive bayes
naive bayes(朴素贝叶斯)是一种常见的分类算法，原理就是概率论一开始讲的贝叶斯原理。之所以naive，是因为它有个很关键的假设：各个特征之间是无关联的。这是一种理想化的假设，所以如果特征之间关联比较密切，就不适合了。朴素贝叶斯在文本分类应用广泛，如分词，垃圾邮件识别等。至于做手写图片识别，其实是不太合适的，因为图片的像素之间关联还是挺大的，但是作为初步学习该方法，还是尝试了下，结果确实不怎么样，不到0.6的正确率，网上看有用朴素贝叶斯达到0.8的，应该是在哪方面做优化了。关于朴素贝叶斯方法的详细介绍，可以看下面参考1的链接，讲的很详细直观。

##实现
R的e1071包有naiveBayes方法，但是使用该方法要注意，label一定得是factor类型的，函数文档也说了，只能做离散值的预测，如果是数字它会认为是连续的。所以可以看到代码中对label做了类型转换。关于该函数的参考文档，可以参考2的链接。

##代码
```
train <- read.csv("data/train.csv", header=TRUE)
test <- read.csv("data/test.csv", header=TRUE)

#注意要对label做类型转换，使用拉普拉斯平滑处理
classifier <- naiveBayes(as.factor(label) ~ ., train, laplace = 10)
result <- predict(classifier, test)

#这个地方很奇怪，写入文件时result都+1了，所以提前-1，还没找到是什么原因
result.table <- data.frame(ImageId=1:nrow(test), Label=as.numeric(result) - 1)
write.csv(result.table, "result/naive_bayes.csv", row.names = F)
```

##心得
朴素贝叶斯算法算是比较简单了，其实之前用的pca、knn、logistic regression、以及线性回归都是非常直观的算法，可能公式推导有的比较复杂。最近也算是了解了它们，然后使用已有的库做练习。后续再熟悉下svm、神经网络，然后就用R或者其他语言实现一遍，还有它们的mapreduce版本也尝试实现下，然后就是能自己用尽可能简单的语言将这些算法尽可能清晰地描述出来。这么一看，任务还是挺多滴。加油！

##参考
1. [朴素贝叶斯](http://www.cnblogs.com/leoo2sk/archive/2010/09/17/naive-bayesian-classifier.html)
2. [R naiveBayes](http://www.inside-r.org/packages/cran/e1071/docs/naiveBayes)
