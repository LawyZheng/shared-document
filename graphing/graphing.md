## 写在最前面

正好最近在写论文，需要对语料库进行一些数据分析。所以也在琢磨着Python的一些关于数据可视化和数理统计分析方面的库。

学校里的论文数据分析指导课教我们的一般都是R、SPSS居多。很多人会纠结用哪个。我只能说用哪个都一样，只要你玩的6就都可以。话虽如此，我最推荐的当然还是Python，毕竟这门语言现在可是个如日中天的玩意儿，而且入门很简单，用处很广，第三方库的开发做的也比较完善。

废话不多说了，先写一些关于数据可视化的介绍吧。有时间再更新关于数理统计的内容。

在这之前先介绍几个需要的第三方库，不然没工具怎么凭空造火箭呢？

**pandas**: 这个就不多说了，进行数据处理必不可少的一个库。在安装pandas的时候，似乎会自动帮你安装scipy这个库，这个库在之后的数理统计分析的时候会用到。pandas库的具体使用我就不详细介绍了，会在过程中慢慢说明一些用法什么的。毕竟这个库水太深了，要玩的风生水起的话，得不知道费多少力气。**但切记！这个库是基础中的基础！！！！**

**jupyter**: 全称 jupyter notebook。这个库其实不是必须的，但我个人还是相当喜欢的。当然你依然可以用vscode、pycharm、sublime text进行代码的编写。但 jupyter notebook 是一个实时交互的编辑器。在我们进行数据分析的时候，经常需要对数据进行筛选以及处理，用了它你就不必每次都让代码从头开始重新跑一次了。而且它图片显示，代码导出什么的都做的非常棒。

**plotly 或者 matplotlib**: 这两个库都是用来绘图的，选一个就行。我比较喜欢plotly，因为画的图比较好看。而且对于jupyter notebook的支持也比matplotlib要好很多。需要注意一点的是 plotly的安装方法和别的库有点不一样，安装的时候需要跟上版本号。截止于写这篇文章的时候，plotly的最新版本为4.1.0。所以pip安装的时候就需要输入 pip3 install plotly==4.1.0 。
> \>pip3 install plotly==4.1.0

## 撸起袖子，准备开干

先别急，准备开干的意思就是还不能开干，因为还有一些工作得完成。别嫌我啰嗦，毕竟我是为小白们写的。

在数据可视化分析之前，总得先有数据吧。在这里我用了老美某电商网站的黑色星期五的购物数据，只是个样本而已。你可以准备你手头任何的数据进行练习的，数据格式可以是csv，可以是excel，也可以是text。不过一般来说，应该都是以csv和excel为主的吧。

准备好你的数据之后，就打开你的命令行，cd到你存放数据的文件夹下。然后在那个目录下面输入命令 jupyter notebook。

> \>jupyter notebook

不出意外的话你的浏览器会自动打开，并弹出以下这个页面。

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic1.png)

可以看到我当前的文件夹下有一个存放着我数据的csv文件，另外那个.ipynb的文件是jupyter的项目文件，这个文件是需要自己创建的。创建的方法就是点击New -> Python3。 然后就会跳转到一个新的页面了。

跳转到新页面之后，在左上角jupyter的logo旁边，会有以各untitle的文字，点击这里就可以修改你的项目名称了。这里我就改成了blackfriday的名称。

都完成了之后，你就可以尽情的写python代码了。注意，在jupyter里面，按enter键是换行，想要运行代码得按 shift + enter 才可以。

当然，一切的故事都得从导入第三方库开始。

```python
import pandas
import plotly.graph_objects as go
import plotly.express as px 
```
写到这里，我就忍不住想吐槽一下plotly这个库。它的图形有的在 plotly.graph_objects 里，有的却在 plotly.express。也说不好哪个在哪，基本上我感觉是比较基础统计图（像是频率分布、折线、箱、散点都基本在plotly.express里），但建议大家看的懂官方文档的话，就去文档查一下，看不懂的话就花时间记一下。

另外还有一点就是在plotly.graph_objects下的图都是大写字母开头，而在plotly.express下的图都是小写字母开头，这样的设计其实让人挺头疼的。

## 数据预处理

先导入数据呗。

```python
data = pandas.read_csv('BlackFriday.csv') # 注意不要搞错文件的路径了。
```

pandas读取csv文件，excel文件，都有对应的 pandas.read_xxx 方法。记得对应上自己存数据的文件类型。

导入数据之后，先查看一下数据吧。因为数据太多了，所以用dataframe.head()查看前面几个就好。想看几个，就在head()里输入几就好了。

```python
data.head(10)
```
![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic2.png)

因为在我后面的数据处理中，没有用到 Stay&#95;In&#95;Current&#95;City&#95;Years 、Marital&#95;Status 、Product&#95;Category&#95;1 、 Product&#95;Category&#95;2 、 Product_Category&#95;3 这些列里的数据，所以先把他们筛选除去。

```python
data = data[['User_ID', 'Product_ID', 'Gender' ,
           'Age', 'Occupation', 'City_Category',
           'Purchase']]
data.head(10)
``` 

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic3.png)

可以看到只剩下我想要的这几列数据了。接着我们要养成数据分析前的好习惯，查看以下数据的个数啊，以及有没有缺失值啊等基本的信息。

```python
data.shape # 查看数据的形状，返回结果为 (数据条数，列数)
```

再看看是否有丢失的数据什么的。

```python
data.isna().sum()
```

如果你看到的数字都是 0 的话，那恭喜你，你的数据非常完整。如果有看到有数字的话，说明数据里还是有缺失值的。对于缺失的数据，可以采取很多种办法，比如赋于平均值啊什么的。当然，最暴力的办法就是直接舍弃那一条数据，那样的话直接用 dataframe.dropna() 就可以了。

很荣幸的是，我这里并没有数据的丢失。所以就可以直接进入下一步了。

## 数据可视化

事实上数据可视化的难点**并不在于要怎么写代码实现可视化，而是在于想要得到一个怎样的数据图，而去绘制这个图又需要怎样去构造这个数据**。然后分清楚x轴和y轴分别要表示什么。那绘图基本不是什么难事了。

### 各年龄段的消费情况

例如，在我的数据中。我想观察一下每个年龄段的消费情况，并且我希望这个数据能同时体现男女性别的差异。然后我再观察一下我的原始数据。如果把 同年龄段并且同性别下的 Purchase（消费金额） 列中的数据全都相加。是不是就能得到我想要的数据了。

```python
gender_data = pandas.pivot_table(data, values="Purchase",
                                 index=['Gender', 'Age'],
                                 aggfunc='sum').reset_index()
```

pandas筛选数据的方法很多，这里用了pivot_table进行筛选，用这个函数主要是因为它可以对数据进行分类重构。

解释一下这个函数里出现几个参数的意思。

首先需要传入原始的DataFrame数据，在我这里就是data。

**values**: 传入进行计算的列的名称。由于目标是获取Purchase的累加，所以需要计算的值是Purchase列下的数据。也可以和index参数一样传入列表，这样就会对传入的那几列数据都进行操作。

**index**: 传入分类的标准。最终目标的分类依据有两个，一个是性别，另一个是年龄段。

**aggfunc**: 这个参数代表想要进行的运算。功能很强大，也很复杂。可以传入字符串（单种运算），列表（多种运算），字典（指定列进行指定运算）。能不能传入自定义函数我倒没试过，有兴趣可以去看看资料。这里我传入了 sum ，表示我要将 values 参数里的列中的值进行累加运算。

记住这三个参数一定要弄清楚作用，**尤其是 values 和 index 参数，一定要弄明白其中的逻辑！不然会出事的！！！**

**最后别忘了调用一下reset_index()方法！！**需要对DataFrame进行重新排列，不然得到的结果会是一个多重表格。

历经千辛万苦，终于得到自己想要的数据了。打印出来看一下。

```
gender_data
```

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic4.png)

可以看到结果非常棒，筛选出了我想要的数据。并且Purchase列中的数值都被赋值成了累加的结果。

那么接下来就该准备绘图了。先说一下绘图的思路。如果要绘制的图体现性别的差异的话，我们的绘制方法就是先绘制一张男性关于年龄段的消费图，再绘制一张女性关于年龄段的消费图。再两张图加在一起，是不是就可以了。

```python
# 初始化一个列表，用于存放两张图
traces = list()

# 循环迭代绘图，分别绘制男、女的图表
for gender in gender_data.Gender.unique():
    trace = go.Bar(x = gender_data.Purchase[gender_data.Gender == gender],
                   y = gender_data.Age,
                   name = gender,
                   orientation = 'h'
                  )
    # 将图添加进列表
    traces.append(trace)
    
# 将两张图绘制在画布上
go.Figure(data=traces,
         layout=go.Layout(title='各年龄段购买总额')).show()
```

先解释一下for循环的部分。

先用gender&#95;data.Gender.unique()获取数据表中所有的性别种类，即F和M。然后就进入了for循环。

go.Bar()函数用来绘制柱状图。需要绘制的图为 x轴代表Purchase, y轴代表Age，所以在参数里分别传入了 gender&#95;data.Purchase 和 gender&#95;data.Age。而对于gender&#95;data.Purchase我们需要进行筛选，因为两次绘制的分别是代表F和M的图表，所以在绘制的时候我们需要筛选出对应的Purchase数据。

name参数是给不同图表进行命名区分。orientation参数是表示把图横过来看，因为Purchase的取值范围要远远广于Age的取值范围，所以把图横过来看要比竖起来更美观一些。

for循环绘制好图之后，需要通过go.Figure()函数把图画在画布上。data参数就是我们绘制好的两张图。layout参数要不要都无所谓，它可以设置一些样式，这里就是给表格加上了个标题而已。

最后再通过show()方法展示出来就可以了。

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic5.png)

好吧，不得不说。老美的男人要比女人狠太多了。

### 综合看看各个年龄段

查看不分性别的情况下，消费金额随年龄的变化情况，以及每个年龄段消费金额的占比。

还是得先筛选数据，但这回不用对性别进行筛选了。具体思路和上面的是一样的。

```python
age_data = pandas.pivot_table(data, values = 'Purchase',
                             index = 'Age',
                             aggfunc='sum').reset_index()
age_data
```

筛选完了之后，绘制一个折线图，看看变化情况。

```python
px.line(age_data, y='Purchase', x='Age', 
		 title='购买总额随年龄的变化').show()
```

对于这种常见统计图的绘制，看起来要比前面的柱状图简单得多了。

参数也很好理解，数据表、y轴、x轴，以及图标的标题（可有可无）。

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic8.png)

再用饼状图看看各个年龄段的消费占比情况。

```python
trace = go.Pie(labels = age_data.Age,
              values = age_data.Purchase,
              hole = 0.4)
go.Figure(data=trace,
         layout=go.Layout(title='各年龄段购买总额占比')).show()
```

饼状图的绘制和柱状图是一样的。所以会稍微麻烦一点点。不过也还是不难理解的，labels参数表示类别，values参数表示具体的值，hole参数（可选可不选）可以设置饼状图中间的空洞大小（范围是0到1，值越大，环就越细）。我个人是觉得中间空出一点点的图看上去要舒服一点。

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic9.png)

### 每个人的消费情况

想查看一下每个人的购物情况，同时体现出性别的差异。

先观看一下原始数据，可以发现每个人都有一个唯一的 User&#95;ID ,而一个人会购买多个不同的产品。所以需要整理出来的数据就是，对于每个相同的 User&#95;ID,都让 Purchase 列中的值进行累加，得到最终的数据就可以了。

```python
gender_dataframe = pandas.pivot_table(data, values='Purchase',
                                     index=['User_ID','Gender'],
                                     aggfunc='sum').reset_index()
gender_dataframe
```

就不仔细解释代码了，思路和之前的都是一样的。筛选完了之后记得打印出来看一下筛选的结果。

因为需要根据性别绘制不同的统计图，所以在 index 参数里依然传入了 Gender。但事实上性别并不会影响最终的数据结果，因为一个 User&#95;ID 肯定只会对应一个性别的，毕竟一个人总不可能有两个性别吧。

先来绘制箱状图，看看数据的中位数什么的好了。

```python
px.box(gender_dataframe, y='Purchase', x='Gender',
	   title='每个人的消费金额').show()
```

参数很简单，第一个是需要绘制的DataFrame数据，第二个是y轴对应的列，第三个是x轴对应的分类（其实这个参数可以省略的，因为这里想分别查看男性和女性的数据情况，所以加入了x轴参数进行分类），第四个就是图的标题。最后不要忘了用show()展示出来就行了。

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic6.png)

可以看到一分位，中位数，四分位，都展示出来了。而且男女也被分开展示出来了。

接着再绘制一个直方图看看看。

```python
px.histogram(gender_dataframe, x='Purchase',
			   color='Gender', title='每个人的消费金额分布图').show()
```

直方图的y轴一般都是频率，所以只用传入x轴的数据。然后用不同的颜色区分出性别，所以加上一个color参数就可以了。

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic7.png)

可以看到大部分人的消费金额都落在 100k 到 500k 这个区别里。不知道这个数据的金额单位是不是美元，要是美元的话老美的人也太富裕了吧。

### 购买数量、购买金额和购买城市之间的关系

先来看看每种城市的消费额情况。

```python
city_data = pandas.pivot_table(data, values = 'Purchase',
                              index='City_Category',
                              aggfunc='sum').reset_index()
city_data
```

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic10.png)

可以看到，消费总额来说的话，B类城市 > C类城市 > A类城市。

回到目标上来，需要一个数据表，里面有每个人买了几件东西，花了多少钱，以及来自哪个城市的数据。所以需要对原始的列表，根据同一个 User&#95;ID 进行累加操作，得到每个人消费的金额数。再计算每个 User&#95;ID 都出现了几次，这样就可以算出每个人购买的数量了。

一步步来，先计算每个人的消费金额。怎么计算相信大家都已经掌握了吧，就不多说了。

```python
money_data = pandas.pivot_table(data, values='Purchase',
                             index=['User_ID','Age','City_Category'],
                             aggfunc='sum').reset_index()
money_data
```

接着计算每个人的购买数量。这里需要多一步操作。因为需要新的一列去记录每个人的购买数量，所以要先在原始数据上加上一列空的数据。

```python
# 给原始数据添加一列，用于记录购买数量
data['Counting'] = 0

counting_data = pandas.pivot_table(data, values='Counting',
                             index=['User_ID', 'Age','City_Category'],
                             aggfunc='count').reset_index()
counting_data
```

这样我们就得到了两个数据表，一个记录着消费金额，一个记录着购买数量，然后将这两个表横向融合起来就可以了。列表的横向融合一般都是基于有相同列的，在这里相同的列就是 User&#95;ID、Age、City&#95;Category。

```python
money_counting_data = money_data.merge(counting_data)
money_counting_data
```

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic11.png)

可以看到，数据表中已经有了消费金额和购买数量两列数据。不过事实上这里可以通过pivot&#95;table的aggfunc参数进行一步到位的筛选。但因为想多普及一下pandas的基础知识，就用了merge的办法。

数据选好了，想看三者的关系怎么办。想要观察杂乱数据的关系，当然是看散点图了。

```python
px.scatter(money_counting_data, x='Purchase', y='Counting',
			color='City_Category').show()
```

散点图属于常用的统计图，所以绘制的参数并不复杂。只要搞清楚x轴和y轴，其他的参数和绘制直方图是一样的。

![](https://raw.githubusercontent.com/LawyZheng/shared-document/master/graphing/pic12.png)

综合之前查看的每类城市的消费总额，不难看出来。虽然A类城市的消费总额最少，但是买的个数最多，消费金额最多的人基本都是来自A类城市的。而B类城市是购买的绝对主力军。C类城市虽然有这不少的消费总额，但每个人都是小额小量的购买。

结果虽然无法很明显的分类出三种城市的消费情况，但是嘛，还是可以从整体看出消费金额和购买数量是呈线性关系的（这显然是屁话，买的越多当然花的也越多，两者不呈线性关系才有鬼）。

[GitHub源码地址](https://github.com/LawyZheng/greedyai_learning/tree/master/greedyai_week8)

[知乎链接](https://zhuanlan.zhihu.com/p/83647657)
