title: "k近邻法-统计学习方法笔记（二）"
date: 2015-11-05 18:34:36
tags: [机器学习]
---

##算法介绍
k近邻法（KNN）是一种常用的机器学习方法，可用于分类，也可用于回归。对于分类，简单来讲就是对于待分类点找出与之最邻近的k个，根据这k个的类比以一定规则决定这个点的类别，一般用的是选举的策略。回归的话基本一致，只是最后取值规则修改下，可以采用平均值或者加权平均值。
通过上面的简介，可以看出来k近邻法有以下三个基本要素：
1. k值的选择。当k较大时，可以假设当k为样本容量N时，就是求均值，模型过于简单了。如果k太小，极端情况下k取1，那么很容易被噪声干扰，本质上是k太小，导致模型过于复杂，出现了过拟合。
2. 距离的度量。常用的欧式距离，还有曼哈顿距离，更为普遍的是Minkowski距离。
3. 分类决策规则。分类一般是少数服从多数，回归的话取均值或者按距离加权均值，距离越近权值越大。

##kd树
对于给定样本点，当要查找预测点的k近邻时，可以直接遍历样本，选出最近的k个即可。但是这样每次都要遍历样本，当样本规模较大时，效率太低了。于是提出了KD树（k dimension tree）算法，kd数可以看作是多维度下的二叉排序树。对于一维的情况，就是平时所见的二叉排序树。当多维时，每次选取其中一维进行分割。至于选那一维，可以选方差最大的维度，方差越大说明数据分布越散，越容易分割。kd树生成算法描述如下：
**算法：构建k-d树（createKDTree）**
**输入：数据点集Data-set和其所在的空间Range**
**输出：Kd，类型为k-d tree**
1. If Data-set为空，则返回空的k-d tree
2. 调用节点生成程序：
　　（1）确定split域：对于所有描述子数据（特征矢量），统计它们在每个维上的数据方差。以SURF特征为例，描述子为64维，可计算64个方差。挑选出最大值，对应的维就是split域的值。数据方差大表明沿该坐标轴方向上的数据分散得比较开，在这个方向上进行数据分割有较好的分辨率；
　　（2）确定Node-data域：数据点集Data-set按其第split域的值排序。位于正中间的那个数据点被选为Node-data。此时新的Data-set' = Data-set\Node-data（除去其中Node-data这一点）。|
3. dataleft = {d属于Data-set' && d[split] ≤ Node-data[split]}
   Left_Range = {Range && dataleft}
   dataright = {d属于Data-set' && d[split] > Node-data[split]}
   Right_Range = {Range && dataright}|
4. left = 由（dataleft，Left_Range）建立的k-d tree，即递归调用createKDTree（dataleft，Left_
   Range）。并设置left的parent域为Kd；
   right = 由（dataright，Right_Range）建立的k-d tree，即调用createKDTree（dataleft，Left_
   Range）。并设置right的parent域为Kd。
   
###kd树结构
```
class Node:
    def __init__(self, point, split):
        self.left = None               #左子树
        self.right = None              #右子树
        self.point = point             #分割点
        self.split = split             #分割维度
```

###kd树构造
```
#中位数
def median(lst):
    if len(lst) % 2:
        return np.median(lst)
    return np.median(lst[1:])


#欧氏距离
def os_distance(v1, v2):
    return np.linalg.norm(np.array(v1) - np.array(v2))


def build_kdtree(points, d):
    if len(points) == 0:
        return None

    mid = median(np.array(points[:, d]))
    left_points = np.array([m for m in points if m[d] < mid])
    right_points = np.array([m for m in points if m[d] > mid])
    points = np.array([m for m in points if m[d] == mid])

    n = Node(points[0], d)
    n.left = build_kdtree(np.concatenate((left_points.reshape((len(left_points), len(points[0]))), points[1:])),
                          (d+1) % len(points[0]))
    n.right = build_kdtree(right_points, (d+1) % len(points[0]))
    return n
```

###kd树查找（最近邻）
```
def search_nearest(kd_tree, target):
    if not kd_tree:
        return None
    search_path = []
    nearest = kd_tree.point

    #构造查找路劲
    while kd_tree:
        search_path.append(kd_tree)
        if os_distance(target, nearest) > os_distance(target, kd_tree.point):
            nearest = kd_tree.point
        if target[kd_tree.split] < kd_tree.point[kd_tree.split]:
            kd_tree = kd_tree.left
        else:
            kd_tree = kd_tree.right
    #回溯查找
    while search_path:
        tree = search_path.pop()
        if abs(target[tree.split] - tree.point[tree.split]) < os_distance(target, nearest):
            if target[tree.split] < tree.point[tree.split]:
                to_add = tree.right
            else:
                to_add = tree.left
        if to_add:
            search_path.append(to_add)
        if os_distance(target, nearest) > os_distance(target, tree.point):
            nearest = tree.point
    return nearest
```

##总结
k近邻法要掌握它的三要素，算法部分是kd树，kd树设计到很多知识，包括kd树的增删改查，kd树的改进。k近邻法在分类领域是一个基本的常用算法，上次做kaggle的手写字体识别，knn准确率最高，达到了0.97左右。
 
