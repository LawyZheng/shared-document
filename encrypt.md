##基础功能

为英文和中文进行转化加密，加密方式为ASCII码，分隔符为 "&#124;"。
注意：这种加密方法也可以为中英文以外的文字加密。

先写一个加密函数encrypt，参数为需要加密的字符串，返回值为加密的结果。

```python
def encrypt(string):
	result = ""
	for char in string:
		#如果是文字或者数字则转成ascii
		if char.isalnum():
			result += str(ord(char))
		#若不是文字则保持不变
		else:
			result += char

		#加上分隔符 |
		result += '|'

	return result
```

再写一个解密函数decrypt，参数为加密后的字符串，返回值为解密的结果。

```python
def decrypt(string):
	result = ""
	#分割字符串，并且去除末尾为空的元素
	temp_list = string.split("|")[:-1]

	for each in temp_list:
		#如果是数字则转成文字和阿拉伯数字
		if each.isdigit():
			result += chr(int(each))
		#若不是数字则保持不变
		else:
			result += each

	return result
```

## 拓展功能

到在线汉语字典网 在线新华字典 中搜索该汉字，获取该汉字搜索页面url中的数字编号作为汉字密码数字部分。

如果是汉字，则密码为 c + 数字密码。如 "好" 的密码为: c6348。

如果是应为字母或者数字，则为 e + ascii码。如"a"的密码为：e97。

分隔符为 "|" 。

注意：该加密方式，不支持中英文以外的其他语言。

导入所需要的模块，requests、re、BeautifulSoup。

```python
import requests
import re
from bs4 import BeautifulSoup
```

先写一个函数get&#95;cnumber，到在线新华字典网中搜索该汉字，并爬取URL中汉字的数字编号。参数为单个汉字字符，返回值为该汉字对应的数字。

```python
def get_cnumber(char):
	#获取网页的url
	headers = {'user-agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36'}
	url = 'http://xh.5156edu.com/index.php'
	data = data = {'f_key':char.encode('gbk'),'f_type':'zi'}
	resp = requests.post(url,data=data,headers=headers)

	#从url中获取数字
	reg = '/([0-9]+).html'
	return re.findall(reg, resp.url)[0]
```	
	
再写一个加密函数my&#95;encrypt，参数为需要加密的字符串，返回值为加密的结果。

```python
def my_encrypt(string):
	length = len(string)
	result = ""
	for i in range(length):
		#如果是标点符号，则保持不变
		if not string[i].isalnum():
			result += string[i]

		#如果是数字或者英文，则转为ascii码
		elif ord(string[i]) < 127 :
			result += 'e'
			result += str(ord(string[i]))

		#如果是中文，则去网站抓取数字
		else:
			result += 'c'
			result += get_cnumber(string[i])

		#加上分隔符 |
		result += '|'

		print('已完成%.2f%%' % ((i+1)*100/length), end='\r')

	print('恭喜你，加密成功！')
	return result
```

为了解密汉字，需要写一个函数get&#95;chinese&#95;char，到在线新华字典网中获取该数字对应的汉字。参数为数字字符串，返回值为汉字字符。

```python
def get_chinese_char(num):
	#获取html
	url = 'http://xh.5156edu.com/html3/%s.html' % num
	headers = {'user-agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36'}
	resp = requests.get(url,headers=headers)
	resp.encoding = 'gbk'

	#抓取汉字信息
	soup = BeautifulSoup(resp.text,'html.parser')
	target_char = soup.find('td',class_='font_22').text

	return target_char
```	

再写解密函数my&#95;decrypt，参数为加密后的字符串，返回值为解密后的结果。

```python
def my_decrypt(string):
	result = ""
	#分割字符串，并且去除末尾为空的元素
	temp_list = string.split('|')[:-1]
	length = len(temp_list)

	for i in range(length):
		#如果开头为e，则为数字或者英文，采用ascii解密
		if temp_list[i].startswith('e'):
			result += chr(int(temp_list[i][1:]))
		#如果开头为c，则为中文，则去网页抓取中文汉字
		elif temp_list[i].startswith('c'):
			result += get_chinese_char(temp_list[i][1:])
		#否则为标点符号，保持不变输出
		else:
			result += temp_list[i]

		print('恭喜你，已完成%.2f%%' % ((i+1)*100/length), end='\r')

	print('解密成功！')
	return result
```

[GitHub完整代码链接](https://github.com/LawyZheng/greedyai_learning/blob/master/greedyai_week1.py)

[知乎链接](https://zhuanlan.zhihu.com/p/76961461)