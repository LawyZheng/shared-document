这篇文章将分为上下两个部分，用100多行Python来看看今日（过去的24小时内）的世界上发生了什么。

上篇，会写到怎么获取数据，整理数据。

下篇，会写到怎么根据数据，选出最符合过去24小时的热点事件。

全篇都是新手向，不需要你是个专业码农，只需要你会一点Python基础就可以了。

上篇需要用到的第三方库：requests、BeautifulSoup、pandas、sqlalchemy（如果你想把数据储存到数据库的话）。

在你的开头先导入需要用到的库呗。

```python
import requests
from bs4 import BeautifulSoup
import time
import pandas
import re
from sqlalchemy import create_engine
```
身边很多朋友都对我说，让我多出去走走，多接触接触这个世界。嗯...鉴于我实在是不想迈出家门一步，也懒得和人聊天。所以为了不成为山林野人，我就想何不用代码来告诉我今天的世界都发生什么事了呢？

不过话说回来，想知道今天世界发生什么了，那你也总得有消息呀。这个东西计算机可凭空捏造不出来，所以呢，这里就提供给大家一个良心榜单中心 —— 今日热榜。这个网站概括了，微博、知乎、百度、天涯、哔哩哔哩、抖音等等等各大社交新闻APP的热门话题。而且它上面的内容都没有被推荐算法 "污染" 过，还是很客观的。

点开一看，令我瞬间起了邪念。如果你懂一点前端或者爬虫的知识，我相信你的反应也会和我一样。我x，这一块块一筐筐的排版方式不就是爬虫的天堂吗？

所以我果断鼠标右键 -> 查看网页源代码。往下拉一拉，马上乐开花。
![](https://github.com/LawyZheng/shared-document/blob/master/todayhot_spider_1/v2-78388311e80058fc0a92d1eed36fa950_b.png?raw=true)

这也太友好了，数据都直接写在源代码里了，连寻找Ajax动态API的步骤都省了。所以果断Python走你。

```python
def get_html(url):
    headers = {'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko)'}
    resp = requests.get(url, headers=headers)
    return resp.text

url = 'https://tophub.today'
html = get_html(url)
print(html)
```

print出来看看。perfect，一点防爬虫的措施都没有，完美的下载了网页源代码的所有内容。
好，接下来就是怎么寻找自己需要的数据了呗。回到网页上，鼠标右击 -> 检查。

鼠标点击这个按钮，让他变成蓝色（Chrome是蓝色的，其他浏览器不知道是不是蓝色）。然后移动你的鼠标到微博的空白区域，直到你看到微博这个区块都变成了蓝色。再看看下面的那个代码部分，把<div class="cc-cd" id="node-1">这个标签关上。

鼠标反复在下面四个标签上移动几下，可以看到这个蓝色的框框会在微博、知乎这些区块之间跳来跳去。

好的，到这我们已经不难看出规律了。这些<div class="cc-cd" id="xxxx">的标签就是用来管理一个个区块榜单的，那我们要获取的榜单数据自然都在这里面，所以用BeautifulSoup把这些都筛选出来就完事了呗。
def get_nodes(html):
    soup = BeautifulSoup(html, 'html.parser')
    nodes = soup.find_all('div', class_='cc-cd') # 注意别忘了class_参数后面的下划线
    return nodes

url = 'https://tophub.today'
html = get_html(url)
nodes = get_nodes(html)
print(len(nodes))
print结果太长了，我们就看看nodes变量的长度好了，是不是36！！！刚好是所有网页上所有区块的总数（如果你有兴趣数的话）。所以我们的nodes变量里已经储存了里面的所有数据了，接下来只用进行数据的整理，制作表格就好了。我们先初始化一个DataFrame类用于储存我们的数据。
df = pandas.DataFrame()
那接下来呢。在我们把所有的数据，写到这个df变量里之前，需要先思考一下怎么去构建这个数据表，我们需要的列有 "content(内容)"、"url(内容的地址连接，可要可不要)"、"source(该内容的来源，即微博、微信还是什么的)"、"start_time(该内容是何时开始上榜的)"、"end_time(该内容是何时出榜的)"。
用之前寻找榜单区块的方法，不难找到。榜单的名字是在<div class="cc-cd-lb">这个标签下的。

而里面一条条的榜单数据，都是在<div class="cc-cd-cd-l nano-content"> 标签里 <a href="xxxxxx">标签下的<span class="t">标签里的。


OK，那就得咯，代码走你。
def get_each_node_data(df, nodes):
    # 获取当前时间
    now = int(time.time())

    # 遍历每个榜
    for node in nodes:
        # 获得榜单的名字
        source = node.find('div', class_='cc-cd-lb').text.strip()

        # 获取榜单里的数据内容
        messages = node.find('div', class_='cc-cd-cb-l nano-content').find_all('a')
        for message in messages:
            content = message.find('span', class_='t').text.strip()

            # 如果不在数据库中，就添加新的数据
            if df.empty or df[df.content == content].empty:
                # 注意创建新的DataFrame的时候，即使只有一条数据，也需要用列表
                data = {
                    'content': [content],
                    'url': [message['href']],
                    'source': [source],
                    'start_time': [now],
                    'end_time': [now]
                }

                item = pandas.DataFrame(data)
                df = pandas.concat([df, item], ignore_index=True)

            # 如果已经在数据库中，则更新相关信息
            else:
                index = df[df.content == content].index[0]
                df.at[index, 'end_time'] = now

    return df


url = 'https://tophub.today'
html = get_html(url)
nodes = get_nodes(html)

df = pandas.DataFrame()
df = get_each_node_data(df, nodes)
Em...总的来说，已经基本竣工了。但是作为强迫症的我，看到微信区域里所有的消息都是「xx」开头，而这个xx的内容基本都是作者信息（我要的只是你的内容，谁写的和我有什么关系咧！），删掉，删掉！！而且什么淘宝、拼多多的榜单都是什么鬼（买买买什么的和我这个穷人又有什么关系呢？），走开！走开！！所以修改一下添加df数据表的过程。
def get_each_node_data(df, nodes):
    # 获取当前时间
    now = int(time.time())

    # 遍历每个榜
    for node in nodes:
        # 获得榜单的名字
        source = node.find('div', class_='cc-cd-lb').text.strip()

        # 获取榜单里的数据内容
        messages = node.find('div', class_='cc-cd-cb-l nano-content').find_all('a')
        # 若是不需要的榜单，则跳过
        if source in ['什么值得买', '淘宝', '拼多多', '武大珞珈山水', '复旦日月光华', '北大未名']:
            continue
        for message in messages:
            content = message.find('span', class_='t').text.strip()
            # 如果是来自微信的消息，就去掉 「」里面的内容
            if source == '微信':
                reg = '「.+?」(.+)'
                content = re.findall(reg, content)[0]

            # 如果不在数据库中，就添加新的数据
            if df.empty or df[df.content == content].empty:
                # 注意创建新的DataFrame的时候，即使只有一条数据，也需要用列表
                data = {
                    'content': [content],
                    'url': [message['href']],
                    'source': [source],
                    'start_time': [now],
                    'end_time': [now]
                }

                item = pandas.DataFrame(data)
                df = pandas.concat([df, item], ignore_index=True)

            # 如果已经在数据库中，则更新相关信息
            else:
                index = df[df.content == content].index[0]
                df.at[index, 'end_time'] = now

    return df
这里筛选微信的内容我用了正则表达式，简单点的话用字符串分割也没什么问题。纯属个人喜好问题。好了，那我们数据也获取到了，也整理好了，保存下来就大功告成了！
如果你想保存成excel（虽然我不推荐），那就和点击就送屠龙宝刀一样方便，DataFrame类提供了一键生成excel文件的功能。
df.to_excel('今日热榜.xlsx')
我比较推荐的是存成json文件，或者录入数据库。后者相较于前者要麻烦一丢丢，如果想了解详情，可以点击这里，看看我之前写的关于Python写入sqlite3的文章。
# 写成json文件
df.to_json('今日热榜.json')

# 写入数据库
engine = create_engine('sqlite:///' + path)  # path为数据库的路径(推荐写成绝对路径)
df.to_sql(table, con=engine, if_exists='replace') # table为保存数据库table的名字，可以随便写，DataFrame会为为什么自动创建
是不是大功告成了....呢？等等等等一下，戳都嘛跌，这里面还有一个小Bug，当你第一次运行的时候，他完美的执行了。但是当你每二次运行程序的时候，发现前一次的数据被覆盖掉了！！解决办法也很简单，只需在初始化DataFrame的时候，不要初始化一个空的df，直接从上一次运行产生的excel文件，或json文件，或数据库中读取数据是不是就可以了呗（以下为方便只写了excel的，其他方法可以自行查阅一下资料）。
所以就修改一下下代码就好了。
url = 'https://tophub.today'
html = get_html(url)
nodes = get_nodes(html)

df = pandas.read_excel('今日热榜.xlsx')
df = get_each_node_data(df, nodes)

df.to_excel('今日热榜.xlsx')
上篇到这就全部结束了，就70行代码还空了一大半，是不是esay peasy。但是呢，看一下结果，一千来条的数据，是不是头都看晕了？所以呢，下篇将会写如何筛选出最热门的那几条消息。
GitHub: https://github.com/LawyZheng/Public_Code/blob/master/today_hot_spider.py
附上完整代码。
import requests
from bs4 import BeautifulSoup
import time
import pandas
import re
from sqlalchemy import create_engine


def get_html(url):
    headers = {'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko)'}
    resp = requests.get(url, headers=headers)
    return resp.text


def get_nodes(html):
    soup = BeautifulSoup(html, 'html.parser')
    nodes = soup.find_all('div', class_='cc-cd')
    return nodes


def get_each_node_data(df, nodes):
    # 获取当前时间
    now = int(time.time())

    # 遍历每个榜
    for node in nodes:
        # 获得榜单的名字
        source = node.find('div', class_='cc-cd-lb').text.strip()

        # 获取榜单里的数据内容
        messages = node.find('div', class_='cc-cd-cb-l nano-content').find_all('a')
        # 若是不需要的榜单，则跳过
        if source in ['什么值得买', '淘宝', '拼多多', '武大珞珈山水', '复旦日月光华', '北大未名']:
            continue
        for message in messages:
            content = message.find('span', class_='t').text.strip()
            # 如果是来自微信的消息，就去掉 「」里面的内容
            if source == '微信':
                reg = '「.+?」(.+)'
                content = re.findall(reg, content)[0]

            # 如果不在数据库中，就添加新的数据
            if df.empty or df[df.content == content].empty:
                # 注意创建新的DataFrame的时候，即使只有一条数据，也需要用列表
                data = {
                    'content': [content],
                    'url': [message['href']],
                    'source': [source],
                    'start_time': [now],
                    'end_time': [now]
                }

                item = pandas.DataFrame(data)
                df = pandas.concat([df, item], ignore_index=True)

            # 如果已经在数据库中，则更新相关信息
            else:
                index = df[df.content == content].index[0]
                df.at[index, 'end_time'] = now

    return df


url = 'https://tophub.today'
html = get_html(url)
nodes = get_nodes(html)

df = pandas.read_excel('今日热榜.xlsx')
df = get_each_node_data(df, nodes)

df.to_excel('今日热榜.xlsx')
