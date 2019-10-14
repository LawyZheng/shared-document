目标任务：了解K最近邻（KNN）算法，并了解基于此算法的分类器。

说直白点，就是了解一下最简单的人工智能的工作原理。我相信如果你对这部分内容感兴趣的话，那你的编程基础，Python基础都已经很扎实了。所以我就不做这方面的详细说明了。

需要的第三方库：sklearn、numpy、plotly或者matplotlib。

老规矩，造飞机造火箭之前都需要先准备好工具。

```python
import heapq
import numpy
from sklearn.datasets.samples_generator import make_blobs  # 用来生成测试数据
import plotly.graph_objects as go
```

## 什么是KNN算法

在知道什么是KNN算法之前，可以先了解一下，KNN算法是干什么的，了解一下人工智能到底是根据什么原理工作的。

其实人工智能没那么复杂，当然深层次的东西还是很复杂的。但作为最外层的我们来讲，想要明白人工智能的工作原理其实并不难。说白了，就是根据事物的不同特征来给事物分类而已。

这么说可能还是比较抽象，举个具体的例子好了。

Umm... 我是学英语的，就举个这方面的例子吧。例如现在我们需要将一群人的英语能力划分为A、B、C三个等级。我们一般会从听、说、读、写四个方面去评估。例如说某个人的听说读写分别是90分，90分，95分，85分。那么我们就可以下评估他的英语能力为A。但我们都知道，有时候可能分数不会那么平均，会出现 "偏科" 的现象。这时候可能就比较难下判断，这个人到底是属于什么级别的。那我们就可以通过找和他最相似的几个人的评价，再决定把他归到哪类中（虽然事实上人们会采用看总分的方式，因为那样的计算量小啊！）。其实本质上，我们就是通过待测目标听说读写的特征数值，与已经被分类好的数据集中的听说读写特征数值，进行距离的计算，再将待测目标归入特定类中的。

也许这样还不是很明白，那么就用数学的示例来展示一下好了。为了可视化方便，就采用二维的数据。先围绕 （-2, 2）、（2, 2）、（0, 4）三个点，随机创建一些数据好了。

```python
def create_sample_data():
    centers = [[-2, 2], [2, 2], [0, 4]]
    x, y = make_blobs(n_samples=60, centers=centers, random_state=0, cluster_std=0.6)
    return x, y, centers
```

这段代码看不懂也没关系，大致意思就是围绕以上这三个点，以0.6的标准差随机创建60个点的数据。返回的x代表每个点的坐标，y代表每个点属于哪个中心点的辐射范围。直接来看一下图可能会比较好理解一点。

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/knn/plot1.png)

通过图片可以明显看出来，这所有的点都是围绕三个中心点分布的。如果把这所有的数据看成一个整体，那么实际上这些点可以归位三类，就是黄、蓝、红三个群体。

那如果那出现了一个新的点 (0, 2) 那这个点该归到哪个群体中呢？乍一看，wtf，这和每个中心点的距离不都是2吗？怎么归类？

但实际上，中心点是我们假设的东西。也就是说，在现实生活中绝大多数事物的分类标准，其实都是模糊的，没有一个明确的规定，例如你看到一个肤色很黑的人，那我们会判断他大概率是属于黑人（在不检测基因的前提下），而到底多黑我们才会判断是黑人，这个界限其实就是很模糊的。所以抛开绝对的标准，我们更多的想法都是，根据它的特征，它更像是属于xxx类里的。

按照这个思路，我们想想，对于新的点 (0, 2) 最像它的点应该怎么找。对，就是距离，距离越近，也就越相似。

那好，原理都理解了，就写算法呗。

```python
def knn_algorism(target, x, y, n):
    heap = list()

    for i, each_data in enumerate(x):
        # 计算两点间距离
        distance = numpy.sqrt((target[0] - each_data[0]) ** 2 + (target[1] - each_data[1]) ** 2)

        # python默认最小堆，所以要取负号，转化为用最小堆筛选最大值
        if len(heap) < n:
            heapq.heappush(heap, (-distance, i, y[i]))
        elif -distance > heap[0][0]:
            heapq.heapreplace(heap, (-distance, i, y[i]))

    # 取符号，转化为正数，并排序
    heap = [(-distance, i, species) for distance, i, species in heap]
    heap.sort()

    return heap
```

传进来的target就是待测点的坐标 (0, 2)，n代表要找和待测点最近的n个点。最终待测点属于哪个类将会由这n个点来投票决定。这个n的取值是很有讲究的。太大的话会让数据集受到异常数据的影响，结果会偏于保守。太小的话，让一个值就确定它的归类，未免会过于激进了。

算法很简单，就是去遍历每个点，用两点距离公式去计算和每个点之间的距离，然后筛选出最近的n个点。在这里我用了堆的数据结构去排序筛选最近的点。要注意的是，Python的标准库heapq中，默认的是最小堆，最小堆是用来筛选最大值的。而我们要选的是最小值，所以要给数据取负来转化成筛选最大值。

最后返回的是最近的n个点的距离，索引以及每个点所属的类别（即处于哪个中心点的辐射范围）。来看一看结果咋样。

```python
x, y, centers = create_sample_data()
# 转化成numpy.array类
centers = numpy.array(centers)

# 待测点坐标
target = [0, 2]

# 计算最近的5个点
nearest_points = knn_algorism(target, x, y, 5)
print(nearest_points)

```

>[(0.9716978435386295, 16, 0), (1.0395278033630897, 20, 1), (1.057017023526498, 48, 0), (1.0589339125811479, 6, 1), (1.0810626749235541, 23, 0)]

可以看到返回的数据中，最近的是属于类别0，索引为16的点。但其实在最近的5个点中有3个点是属于类别1的。所以这个点大概率是属于类别1，而不是类别0。还是画个图吧，毕竟没图你说个xx呢。

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/knn/plot2.png)

图上可以看出来，距离待测点 (0, 2) 最近的5个点都被用线连接上了，有三条都是和蓝色的群体相连的，有两条是和红色的群体相连的。所以就概率学和统计学的角度来说，那待测点大概率应该是属于蓝色群体的。

这就是KNN算法的原理以及实现方法了。

## 来人，把我的意大利炮拿出来

但事实上，当我们的数据集变得越来越大，当维度变得越来越多的时候，以上这样实现的KNN算法会变得很慢很慢的。所幸的是Python毕竟作为人工智能编程语言的大佬，它为我们提供了很多很多的人工智能算法接口，其中就包括KNN算法。

sklearn.neighbors 中的 KNeighborsClassifier 就是KNN的一个算法接口。

之所以没在一开始就介绍这个接口，是因为想和大家一起感受一下KNN算法的实现过程。这样可以更深刻的体验和理解算法的原理。

话不多说，直接上代码。

```python
from sklearn.neighbors import KNeighborsClassifier

x, y, centers = create_sample_data()
# 转化成numpy.array类
centers = numpy.array(centers)

# 转化成二维数组
target = [[0, 2]]

# 实例化一个分类器，取5个最近邻的值
knn = KNeighborsClassifier(n_neighbors=5)

# 拟合模型
knn.fit(x, y)

# 获取最近邻的值
neighbor_points = knn.kneighbors(target, return_distance=False)
print(neighbor_points[0])

```

是不是很简单，啥都不用你干，输入几个参数就完事了。但**需要注意的是，这里的待测点坐标需要转化成二维数组，不然是会报错的！！！knn.kneighbors() 这个方法的返回值也是一个二维数组！！！设置了return&#95;distance这个参数为False之后，返回的值就是最近邻点的索引。如果没设置的话，会返回索引和距离**。

打印出结果看看。

>[16 20 48  6 23]

可以和前面我们自己写的算法结果对比一下，不出意外的话，这两个结果应该是相同的。

## 过一把人工智能的瘾

经过上面对KNN算法的理解，应该能明白人工智能的工作原理了。那么在实际的应用中，我们该怎么去做这个人工智能的测试呢。

我们总不能写这么一大坨代码，就为了给一个未知事物分个类这么简单吧，那也太不智能了。我们总是希望能建立一个自动化的处理模型，只要我们往这个模型里输入未知事物的特征量，那么模型就能预测出所属的类别，这才是人工智能该办的事嘛。

还是举个具体的例子来演示一下整个流程吧。

在sklearn这个三方库，默认给我们提供了一些测试模型用的数据。比如它里面就有一个鸢尾花类别的数据集。根据花瓣的四种特征量，来预测是哪种类型的鸢尾花。花的具体我也不清楚，反正我们就只用知道，它提供了给我们测试用的数据就行了，就把他想象成玫瑰百合康乃馨也没问题，方便自己理解就行。

当然我们得先导入一下这个小项目里要用到的一些工具。

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier
```

先来获取一下花的数据。

```python
def get_flower_data():
    iris = load_iris()
    # 获取花的特征量数据
    iris_data = iris.data
    # 获取花的类别数据
    iris_type = iris.target
    return iris_data, iris_type
```

这样，在iris&#95;data这个变量里，就存放了关于鸢尾花特征量的数据。在iris&#95;type就存放了关于鸢尾花类别的数据。他们都是一一对应的。有兴趣的朋友可以打印出来看一看，在这里我就不打印了。

然后我们再根据这些数据去做模型拟合和检测模型的准确率。因为我们这会儿我们不再关心每个数据的最近邻数据都是哪些，我们更关注的是这个数据是属于哪个种类，所以直接让算法返回预测结果就好了。

```python
# 筛选训练集
x_train, x_test, y_train, y_test = train_test_split(iris_data, iris_type, test_size=0.25)
# 标准化数据
std = StandardScaler()
x_train = std.fit_transform(x_train)
x_test = std.transform(x_test)

# knn算法，拟合模型
knn = KNeighborsClassifier(n_neighbors=5)
knn.fit(x_train, y_train)

# 获取预测结果
y_predict = knn.predict(x_test)
print(y_predict)
print(y_test)
```

先来看看结果再做解释好了。

> [2 0 1 2 1 0 2 2 0 0 0 0 0 2 2 1 0 0 0 0 2 0 1 0 2 0 1 1 2 0 0 2 2 1 1 0 2
 1]

> [2 0 1 2 1 0 1 2 0 0 0 0 0 2 2 1 0 0 0 0 2 0 1 0 2 0 1 1 2 0 0 2 2 1 1 0 2
 1]
 
第一行是模型的预测结果，第二行是真实的数据结果。可以看得到基本都是一致的，除了偶尔会出错之外。

可以看到这个小项目和之前的点预测程序是有点不一样的。我们先把整个数据集分为了两个部分，一个是用于训练模型的训练集，另一个是用来检测模型准确度的测试集。train&#95;test&#95;split() 这个函数就帮助我们把数据集以1比3的比例进行了分类。

分类之后需要对数据进行标准化处理，这个数据处理在统计学上很常见，就不多做解释了。

但是需要注意的一点是，可以看到，训练集和测试集的标准化所用的方法是不一样的。fit&#95;transform() 这个方法我们一般用来对训练集进行转化，它会自动帮我们计算一些转化所需要的模型啊参数啊什么的。但在对测试集的标准化用的是transform()，原因是我们转化所需要的模型和参数，都已经在转化训练集的时候拟合完成了，我们要统一两次标准化的结果，所以直接转化成标准值就好了。

在这个KNN分类器训练完之后，我们就可以往这个模型里面扔数据，让它智能的帮我们预测结果了。

当然因为这个是官方给出的测试数据，所以模型的准确率还是很高的。但我们平时更多的是带着试一下的心态去建立KNN分类器模型的。所以这时候，我们就需要去看看模型的预测准确率怎么样。

```python
print(knn.score(x_test, y_test))
```

这时候就会返回一个准确率了。

> 0.9210526315789473

如果你的结果和我的不一样，很正常，因为训练集是从数据集中随机筛选的，是肯定会影响到模型的预测结果的。不过也正因为如此，我们在平时的测试中，就可以采用多次进行模型训练的方法，算出平均的准确率，从而获得比较客观稳定的模型预测准确率。

```python
def get_knn_accuracy(iris_data, iris_type, n):
    score = 0
    for i in range(n):
        # 筛选训练集
        x_train, x_test, y_train, y_test = train_test_split(iris_data, iris_type, test_size=0.25)
        # 标准化数据
        std = StandardScaler()
        x_train = std.fit_transform(x_train)
        x_test = std.transform(x_test)

        # knn算法，拟合模型
        knn = KNeighborsClassifier(n_neighbors=5)
        knn.fit(x_train, y_train)

        # 计算当前准确率
        score += knn.score(x_test, y_test)
        
	return score / n
	
	
iris_data, iris_type = get_flower_data()
print(get_knn_accuracy(iris_data, iris_type, 10))
```

这样通过迭代，计算10次预测结果的平均准确率，就可以估算这个模型做的怎么样了。如果准确率很低的话，可能就要考虑改变n&#95; neighbors参数的值啊，或者重新抽取不同的特征量啊什么的方法进行探讨了。

## 小结一下

为什么说人工智能是大数据下的产物。从上面也可以看到，分类器模型拟合过程中，需要的数据是很大量的。换句话说，数据量越大，那模型的准确率也会相应提高。但是也可以看到，能做出模型的前提是有数据，而这些数据都是需要事先人为的进行打标签分类的。这其实是一种有监督的机器学习方法，当然相对的肯定也有无监督的机器学习。

说到这，那人工智能的本质是什么呢。就是通过对特征量的进行一些计算，将目标事物进行了归类，给出一个判断而已。这是从最简单，最本质的视角去解释人工智能，当然在这简单的本质背后，是需要大量的深入研究作为基础的。比如，要怎么抽取特征量才能最准确的分出类别，要怎么设置模型的参数，这些都是需要研发人员去深究的。

[GitHub源码链接](https://github.com/LawyZheng/greedyai_learning/tree/master/greedyai_week9)

[知乎链接](https://zhuanlan.zhihu.com/p/84886024)


