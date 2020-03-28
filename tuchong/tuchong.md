偶尔写写爬虫也算是打磨无聊生活的一种方式了。

之前写了一个用100多行Python爬虫看世界的帖子，有兴趣的朋友可以看一下。  
[带你用100多行Python爬虫看看今天的世界（上）](https://zhuanlan.zhihu.com/p/81010673)  
[带你用100多行Python爬虫看看今天的世界（下）](https://zhuanlan.zhihu.com/p/81260856)

Em..今天呢就来写一个用30行Python爬虫看<del>喵星人</del>（划掉，plmm）的帖子好了。

先来介绍一下这次爬虫的主角，[图虫网](https://tuchong.com)。懂的人都懂，摄影爱好者的天堂。我有一半的摄影构图和后期调色技巧都是这上面学的。

最重要的是，上面还有相当多好看的照片（plmm）。众所周知，图虫上多了营养是会不够的。

当然，我上图虫主要是为了学摄影技术，偶尔看看喵星人的（正经脸）。

不啰嗦了，赶紧进入正题。

这次的爬虫相当简单，只用一个requests库全部就能搞定了。

```python
import requests
```

## 暗中观察

爬虫最重要的工作就是前期观察，找到数据的接口在哪里，有什么特点，是以怎样的方式传输的。

先去官网看看。

<img src="https://raw.githubusercontent.com/LawyZheng/shared-document/master/tuchong/1.png" width="100%">

点社区，不要直接在图库里搜索关键字，别问我为什么知道的。有兴趣的话，可以自己试一下爬取图库里的数据。

<img src="https://raw.githubusercontent.com/LawyZheng/shared-document/master/tuchong/2.png" width="100%">

在上面的导航栏里输入想搜索的关键字。我想爬一些喵星人的照片，那就搜猫就行了。

然后把页面往下拉一拉，可以发现照片是以瀑布流的方式加载的（边看边加载）。那就很自然而然的想到，这种加载方式，一般都是通过AJAX来动态获得数据的。

所以按F12，调出浏览器的控制台，求证一下自己的假设。

<img src="https://raw.githubusercontent.com/LawyZheng/shared-document/master/tuchong/3.png" width="50%"><img src="https://raw.githubusercontent.com/LawyZheng/shared-document/master/tuchong/4.png" width="50%">

在左图里可以看到，本来是没有任何数据的，但在滚轮向下滚的过程中，发现数据慢慢被加载出来了。

就你了，那个postxxxpage=2的玩意儿，看名字像不像是第几页的意思。所以点进去看看。

<img src="https://raw.githubusercontent.com/LawyZheng/shared-document/master/tuchong/5.png" width="100%">

可以把data里的数据翻出来看看，里面都是关于图片的一些数据信息。然后去找这个数据的URL接口就可以啦。

<img src="https://raw.githubusercontent.com/LawyZheng/shared-document/master/tuchong/6.png" width="100%">

可以看到请求这个数据的URL就是这个了。但我们想要一般都是可以自己定制的动态的URL,而不是静态的固定的URL。所以我们可以观察一下URL的构成。

URL就是由域名+"?"+参数构成的。看到URL里带问号，一般就会直接想到URL是有参数的。就可以找找参数的具体内容。

所以URL的构成也很清楚了，就是 https://tuchong.com/rest/search/post 然后加上自己定制的内容参数。

大概理清楚了，那就写代码吧。

## 代码走你

先上代码。

```python
def get_html_json_data(keyword, page):
    url = 'https://tuchong.com/rest/search/posts'
    params = {
                "query": keyword,
                "count": 20,
                "page": page
             }

    headers = {"user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36"}
    response = requests.get(url, params=params, headers=headers)

    return response.json()
```

思路很简单。就是动态接收keyword，page两个参数，然后赋值给Url参数就可以了。

至于headers其实加不加都无所谓，请求这种形式的数据，一般服务器都不会管user-agent的。毕竟没有人会无聊到在浏览器输入这样的URL去访问吧。

**这里需要注意两个地方。第一个是URL的末尾是没有带上问号的。第二个是返回值，通常我们会返回respons.text里的文本数据。但ajax加载的数据一般都是json数据，所以直接让response.json()解析出来就好了。也省去了调用标准库去解析的麻烦。**

写完了函数之后就测试一下。

```python
print(get_html_json_data("猫", 1))
```

打印出来的数据一大串是不是。Em..一大串就没错了。其实就是字典里嵌套字典的数据结构而已，耐心点慢慢解析出来吧，这个过程我就不详细说明了。

```python
json_data = get_html_json_data("猫", 1)

for each_images in json_data['data']['post_list']:
    for each_image in each_images['images']:
        url = each_image['source']['ft640']
        print(url)
```

这样就可以看到每张图片对应的URL了。

解析循环过程我就不叙述了，可以自己去摸索摸索。我就关于 ft640 这个键说明一下。

<img src="https://raw.githubusercontent.com/LawyZheng/shared-document/master/tuchong/7.png" width="100%">

事实上每张照片都有这几个数据，这些都是代表不同的尺寸而已。观察就可以发现图片的URL都是由 https://photo.tuchong.com/uers&#95;id/尺寸/img&#95;id.webp(jpg) 构成的。不过这之中还缺了一个全尺寸，就是f。所以要是想要全尺寸图片的话，把ft640换成f就好了。当然，想要别的尺寸的话，就提取对应的URL就可以了。

好。那我们大概重新整理一下代码，然后把爬取多个页面的代码也加上去。

```python
for page in range(1, 6):
    json_data = get_html_json_data("猫", page)

    # 遍历每个图集
    for each_images in json_data['data']['post_list']:
        # 遍历每张照片
        for each_image in each_images['images']:
            url = each_image['source']['ft640']
            url = url.replace('ft640', 'f')
            user_id = each_image['user_id']
            image_id = each_image['img_id']
            
            print(url, user_id, image_id)
```

不要太贪心了，5页就好了。太多了的话营养会跟不上的。这里我顺便提取了uers&#95;id和img&#95;id，为的是接下来方便保存图片的时候用的。

好，这样我们就获得了所有图片的URL了，那接下来只要用这些URL去下图片就可以了。

## 终于可以做最期待的事情了

图片要怎么下呢？其实和访问网站是一样的。老样子，先上代码。

```python
def download_image(url, user_id, image_id):
    headers = {"user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36"}
    response = requests.get(url, headers=headers)

    with open("tuchong/{}-{}.jpg".format(user_id, image_id), 'wb') as f:
        f.write(response.content)
```

代码就几行而已，如果把可有可无的headers也删掉的话，就更少了。下图片，包括视频其实都很简单。只要访问对应的URL,获取response对象，然后把里面的内容保存成对应文件就行了。

我为了方便查看，就都保存到了tuchong这个文件夹里，这里根据自己的需要写路径就好了。

但要注意**response对象中，我们需要获取的是content的属性，而不是text属性**。content的是内容数据的二进制形式，而text是文本形式，如果获取text属性是会乱码的。计算机中储存图片、视频都是二进制的形式储存的。

也正因为如此，**我们在写入文件的时候，记住要用二进制的形式 "wb" 写入文件**。

注意好这两点就行。最后把下载图片的函数写到主函数里就大功告成了！

```python
for page in range(1, 6):
    json_data = get_html_json_data("私房", page)

    # 遍历每个图集
    for each_images in json_data['data']['post_list']:
        # 遍历每张照片
        for each_image in each_images['images']:
            url = each_image['source']['ft640']
            url = url.replace('ft640', 'f')
            user_id = each_image['user_id']
            image_id = each_image['img_id']

            # 下载图片
            download_image(url, user_id, image_id)
```

我才不会告诉你把搜索关键字换成 "私房" ，就会有福利呢。

其实到这里这整个程序就算完成了，但但但但但但是呢。运行了之后发现速度有点慢是不是。慢就对了，因为这个执行速度很大程度取决于你的网速和电脑配置的。

所以接下来要进行一些改造，让这个爬虫可以飞起来。当然，如果Python水平还是入门的朋友的话，下面看不看都没关系。

## 飞起来！（optional）

为什么先来理解一下为什么上面的程序会慢。

这上面程序的执行顺序都是依次执行的，也就是说，要一页页的去访问页面，然后获取页面中图片的URL，再根据URL去下载图片，等一张下完了，才进行下一步操作。简单点说，就是单流水线、串联式的。

那我们自然而然就想到了，如果能让他多流水线访问，并联式下载，是不是速度就可以飞起来了。没错，答案就是这样。不过如果我们自己写多线程或者多进程的程序还是比较复杂的。

这时候，就轮到scrapy框架登场了。可以让你的下载速度提升20倍！（瞎说的，我并没有仔细计算过，反正能提高非常非常多的下载速度）

这里就简单的帖一下代码，具体scrapy库的用法可以去官方文档或者我之前的帖子里参照一下。  
[贪心学院Python训练营第六周-百度贴吧爬虫](https://zhuanlan.zhihu.com/p/81664441)

先安装scrapy库。

打开命令行，创建scrapy框架的工程文件夹。

> scrapy startproject tuchong_spider

再根据提示创建爬虫。

> cd tuchong_spider  
> scrapy genspider tuchong https://tuchong.com/search

后面的url随便写一个就行了，反正我们一会儿是要重写的。

然后打开spiders文件夹下的tuchong.py文件，写入下面的代码。

```python
# tuchong.py
from urllib.parse import urlencode
import json
import scrapy


class TuchongSpider(scrapy.Spider):
    name = 'tuchong'

    def start_requests(self):
        url = 'https://tuchong.com/rest/search/posts'

        for page in range(1, 6):
            params = {
                    "query": "私房",
                    "count": 20,
                    "page": page
                 }

            new_url = url + '?' + urlencode(params)
            yield scrapy.Request(new_url, self.parse)

    def parse(self, response):
        json_data = json.loads(response.text)

        # 遍历每个图集
        for each_images in json_data['data']['post_list']:
            # 提取出图集里每张图片的尺寸
            image_urls = list()
            for each_image in each_images['images']:
                url = each_image['source']['ft640']
                url = url.replace('ft640', 'f')
                image_urls.append(url)
                print(url)
```

思路和之前的程序是一模一样的。但因为我们没有用requests库，所以也就不能用他提供的一些方法了。只能使用urllib.pars和json两个标准库去代替了。

**这里我们通过urlencode函数重构URL的时候，别忘了自己给它加上问号！！**

这时候在命令行里可以试着运行一下爬虫看看。

> scrapy crawl tuchong

如果你在命令行里看到了很多图片的URL链接，那就说明成功了获取到URL了！接下来去下载文件就行了。

打开 items.py 文件，写入下面的代码。

```python
# item.py
import scrapy


class TuchongSpiderItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    image_urls = scrapy.Field()
    image = scrapy.Field()
```

然后回到 tuchong.py 文件，修改parse方法，引入item类。

```python
# tuchong.py
from tuchong_spider.items import TuchongSpiderItem

    def parse(self, response):
        json_data = json.loads(response.text)

        # 遍历每个图集
        for each_images in json_data['data']['post_list']:
            # 提取出图集里每张图片的尺寸
            image_urls = list()
            for each_image in each_images['images']:
                url = each_image['source']['ft640']
                url = url.replace('ft640', 'f')
                image_urls.append(url)

            item = TuchongSpiderItem()
            item['image_urls'] = image_urls

            yield item
```

再去到 settings.py 文件里进行配置，打开管道，设置保存文件路径等。

```python
# settings.py
ITEM_PIPELINES = {
   'scrapy.pipelines.images.ImagesPipeline': 1,
}
IMAGES_STORE = "tuchong_images"
MEDIA_ALLOW_REDIRECTS = True
IMAGES_EXPIRES = 30
```

解释一下这几个参数都是什么意思。

第一个ITEM&#95;PIPELINES表示打开图片的管道，这样scrapy框架才会自动为我们下载储存图片。

第二个IMAGES&#95;STORE指的是储存文件夹的路径，自定义一下就好。

第三个MEDIA&#95;ALLOW&#95;REDIRECTS表示允许图片重定向，如果不设置的话，会禁止图片的URL进行重定向，在这个程序里是会报错的。

第四个IMAGES&#95;EXPIRES表示的是爬取的数据有效期，单位是天。设置成30就代表，不重复爬取30天之内已经爬取过的数据。

OK，然后去你的命令行跑一下爬虫。你就会发现它飞起来了。(刚开始可能要耐心地等一段时间才会开始下载图片)

[github链接：requests版](https://github.com/LawyZheng/Public_Code/blob/master/tuchong.py)

[github链接：scrapy版](https://github.com/LawyZheng/Public_Code/tree/master/tuchong_spider/tuchong_spider)

[知乎链接](https://zhuanlan.zhihu.com/p/86647389)