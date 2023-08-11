## 1. Requests请求响应
```python
import requests
# 六种请求
resp = requests.get('http://www.baidu.com', params={'key':'value'})
resp = requests.post('http://www.baidu.com', data = {'key':'value'})
resp = requests.put('http://www.baidu.com', data = {'key':'value'})
resp = requests.delete('http://www.baidu.com')
resp = requests.head('http://www.baidu.com')
resp = requests.options('http://www.baidu.com')

# url
resp.url
# 响应编码
resp.encoding
# 响应内容
resp.text
# 二进制响应内容
resp.content
# json响应内容
resp.json()
# 原始响应内容
resp.raw
# 响应状态码
resp.status_code
resp.status_code == requests.codes.ok
# 错误请求，抛出异常
bad_resp.raise_for_status()
# 响应头
resp.headers['Content-Type']
resp.headers.get('content-type')
# 响应cookies
resp.cookies['cookie_name']

# 自定义请求头
requests.get(url, headers={'user-agent': 'Fiddler/5.0.20204.45441'})

# 复杂post
requests.post(url, data=json.dumps(payload))
requests.post(url, json=payload)

# multipart上传文件
url = 'http://httpbin.org/post'
files = {'file': open('report.xls', 'rb')}
resp = requests.post(url, files = files)

files = {'file': ('report.xls', open('report.xls', 'rb'), 'application/vnd.ms-excel', {'Expires': '0'})}
# files = {'file': ('report.csv', 'some,data,to,send\nanother,row,to,send\n')}
requests.post(url, files=files)

url = 'http://httpbin.org/post'
multiple_files = [
	('images', ('foo.png', open('foo.png', 'rb'), 'image/png')),
	('images', ('bar.png', open('bar.png', 'rb'), 'image/png'))
]
requests.post(url, files=multiple_files)

# 发送cookie
cookies = dict(cookies_are='working')
requests.get(url, cookies=cookies)

jar = requests.cookies.RequestsCookieJar()
jar.set('tasty_cookie', 'yum', domain='httpbin.org', path='/cookies')
jar.set('gross_cookie', 'blech', domain='httpbin.org', path='/elsewhere')
url = 'http://httpbin.org/cookies'
requests.get(url, cookies=jar)

# 请求超时
requests.get('http://github.com', timeout=0.001)
```
## 2. Re正则表达式

| 元符号 | 含义 |
| ----- | ----- |
| . | 匹配除换⾏符以外的任意字符 |
| * | 重复零次或更多次 |
| + | 重复⼀次或更多次 |
| ? | 重复零次或⼀次 |
| {n} | 重复n次 |
| {n,} | 重复n次或更多次 |
| {n,m} | 重复n到m次 |
| .\* | 贪婪匹配, 尽可能多的去匹配结果 |
| .\*? | 惰性匹配, 尽可能少的去匹配结果 -> 回溯 |
| \\w | 匹配字⺟或数字或下划线 |
| \\s | 匹配任意的空⽩符 |
| \\d | 匹配数字 |
| \\n | 匹配⼀个换⾏符 |
| \\t | 匹配⼀个制表符 |
| ^ | 匹配字符串的开始 |
| $ | 匹配字符串的结尾 |
| \\W | 匹配⾮字⺟或数字或下划线 |
| \\D | 匹配⾮数字 |
| \\S | 匹配⾮空⽩符 |
| a\|b | 匹配字符a或字符b |
| () | 匹配括号内的表达式，也表示⼀个组 |
| \[...\] | 匹配字符组中的字符 |
| \[^...\] | 匹配除了字符组中字符的所有字符 |

|修饰符|描述|
|---|---|
|re.I|使匹配对大小写不敏感|
|re.L|做本地化识别（locale-aware）匹配|
|re.M|多行匹配，影响 ^ 和 $|
|re.S|使 . 匹配包括换行在内的所有字符|
|re.U|根据Unicode字符集解析字符。这个标志影响 \w, \W, \b, \B.|
|re.X|该标志通过给予你更灵活的格式以便你将正则表达式写得更易于理解。|

```python
# re.match只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None；而re.search匹配整个字符串，直到找到一个匹配
import re
print(re.match('www', 'www.runoob.com').span()) # 在起始位置匹配
print(re.match('com', 'www.runoob.com')) # 不在起始位置匹配

line = "Cats are smarter than dogs"
matchObj = re.match( r'(.*) are (.*?) .*', line, re.M|re.I)
if matchObj: 
	print("matchObj.group() : ", matchObj.group())
	print("matchObj.group(1) : ", matchObj.group(1))
	print("matchObj.group(2) : ", matchObj.group(2))
else: 
	print("No match!!")
	
print(re.search('www', 'www.runoob.com').span())  # 在起始位置匹配
print(re.search('com', 'www.runoob.com').span())  # 不在起始位置匹配

line = "Cats are smarter than dogs";
searchObj = re.search( r'(.*) are (.*?) .*', line, re.M|re.I)
if searchObj: 
	print("searchObj.group() : ", searchObj.group())
	print("searchObj.group(1) : ", searchObj.group(1))
	print("searchObj.group(2) : ", searchObj.group(2))
else: 
	print("Nothing found!!")
	
	
# 删除字符串中的 Python注释
num = re.sub(r'#.*$', "", phone)
print("电话号码是: ", num)
# 删除非数字(-)的字符串
num = re.sub(r'\D', "", phone)
print("电话号码是 : ", num)

pattern = re.compile(r'([a-z]+) ([a-z]+)', re.I)
# re.I 表示忽略大小写
m = pattern.match('Hello World Wide Web')
print m

pattern = re.compile(r'\d+')
# 查找数字 
result1 = pattern.findall('runoob 123 google 456') 
result2 = pattern.findall('run88oob123google456', 0, 10)

it = re.finditer(r"\d+","12a32bc43jf3") 
for match in it:
	print(match.group())
```

```python
import re
import requests
import  csv
 
page = input("请输入页码(0=1，25=2，50=3...)：")
header = {
"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36"
}
url = f"https://movie.douban.com/top250?start={page}&filter="
 
# 拿取页面源代码
res = requests.get(url=url,headers=header)
res_text = res.text
 
#解析数据
obj = re.compile(r'<li>.*?<span class="title">(?P<name>.*?)</span>'
                 r'.*?<br>(?P<year>.*?)&nbsp'
                 r'.*?<span class="rating_num" property="v:average">(?P<score>.*?)</span>'
                 r'.*?<span>(?P<views>.*?)人评价',re.S)
result = obj.finditer(res_text)
 
#打开csv文件
f = open("video.csv", mode="w")
#创建写入内容对象
csvwiter = csv.writer(f)
for i in result:
    dic = i.groupdict()
    dic['year'] = dic['year'].strip()#为year单独设置跳过空格
    csvwiter.writerow(dic.values())#写入内容为dic里的数据
    #print(i.group("name"))
    #print(i.group("year").strip())#跳过空格
    #print(i.group("score"))。
    #print(i.group("views"))
f.close()
print("done!")
```
## 3. Beautiful Soup

| 解析器 | 使用方法 | 优势 | 劣势 |
| ----- | ----- | ----- | ----- | ----- |
| Python标准库 | BeautifulSoup(markup, 'html.parser') | python内置的标准库，执行速度适中 | Python3.2.2之前的版本容错能力差 |
| lxml HTML解析器 | BeautifulSoup(markup, 'lxml') | 速度快、文档容错能力强 | 需要安装C语言库 |
| lxml XML解析器 | BeautifulSoup(markup 'xml') | 速度快，唯一支持XML的解析器 | 需要安装C语言库 |
| html5lib | BeautifulSoup(markup, 'html5lib') | 最好的容错性、以浏览器的方式解析文档、生成HTML5格式的文档 | 速度慢，不依赖外部拓展 |

```python
from bs4 import BeautifulSoup
# 选择解析器
soup = BeautifulSoup(html_doc, 'lxml')

# 获取html格式化输出
print(soup.prettify())
# 选择title节点
print(soup.title)
# 选择head节点
print(soup.head)
# 选择p节点
print(soup.p)
# 节点类型 <class 'bs4.element.Tag'>
print(type(soup.title) type(soup.head) type(soup.p))
# 节点名称
print(soup.title.name)
# 节点内容
print(soup.title.string)
print(soup.title.text)

# 节点属性
print(soup.p.attrs)
print(soup.p.attrs['name'])

# 直接子节点
soup.contents[0].name
soup.contents[1].name
# 多个子孙节点
for child in head_tag.descendants:
	print(child)
# 父节点
print(soup.a.parent)
# 父爷节点
print(soup.a.parents)
for i, parent in enumerate(soup.a.parents):
    print(i, parent)
# 兄弟节点
print(soup.a.next_sibling)
print(list(soup.a.next_siblings))
print(soup.a.previous_sibling)
print(list(soup.a.previous_siblings))

# 方法选择
# name 根据节点名称查找
# attrs 根据节点属性查找
find_all(name, attrs, recursive, text, **kwargs)
# 类似方法还有find查找单个节点
# find_parents查找多个父爷节点
# find_parent查找单个父节点
# find_next_siblings find_previous_siblings查找所有兄弟节点
# find_next_sibling find_previous_sibling查找单个兄弟节点

print(soup.find_all('a'))
for a in soup.find_all('a'):
    print(a.find_all('span'))
    print(a.string)

print(soup.find_all(attrs={'id': 'link1'}))
print(soup.find_all(attrs={'name': 'Dormouse'}))
print(soup.find_all(class_ = 'sister'))
print(soup.find_all(id = 'link2'))

print(soup.find(name='a'))

# css选择器
print(soup.select('.panel .panel-heading')) # 获取class为panel-heading的节点
print(soup.select('ul li')) # 获取ul下的li节点
print(soup.select('#list-2 li')) # 获取id为list-2下的li节点
print(soup.select('ul'))    # 获取所有的ul节点
for ul in soup.select('ul'):
    print(ul.select('li'))

# 获取文本
for li in soup.select('li'):
    print('String:', li.string)
    print('get text:', li.get_text())
```
## 4. Xpath
```python
from lxml import etree
doc='''
	<div>
		<ul>
			 <li class="item-0"><a href="link1.html">first item</a></li>
			 <li class="item-1"><a href="link2.html">second item</a></li>
			 <li class="item-inactive"><a href="link3.html">third item</a></li>
			 <li class="item-1"><a href="link4.html">fourth item</a></li>
			 <li class="item-0"><a href="link5.html">fifth item</a> # 注意，此处缺少一个 </li> 闭合标签
		 </ul>
	 </div>
	'''
html = etree.HTML(doc)
result = etree.tostring(html)
print(str(result,'utf-8'))

#pretty_print=True 会格式化输出
result = etree.tostring(html, pretty_print=True)
print(result)
```

### 1. 定位语法
路径表达式
```
    /   根节点，节点分隔符，
    //  任意位置
    .   当前节点
    ..  父级节点
    @   属性
```

通配符
```
    *   任意元素
    @*  任意属性
    node()  任意子节点（元素，属性，内容)
```

谓语
```
    //a[n] n为大于零的整数，代表子元素排在第n个位置的<a>元素
    //a[last()]   last()  代表子元素排在最后个位置的<a>元素
    //a[last()-]  和上面同理，代表倒数第二个
    //a[position()<3] 位置序号小于3，也就是前两个，这里我们可以看出xpath中的序列是从1开始
    //a[@href]    拥有href的<a>元素
    //a[@href='www.baidu.com']    href属性值为'www.baidu.com'的<a>元素
    //book[@price>2]   price值大于2的<book>元素
```

多个路径
```
//book/title | //book/price
```

语法案例
```python
from lxml import etree

if __name__ == '__main__':
    doc='''
        <div>
            <ul>
                 <li class="item-0"><a href="link1.html">first item</a></li>
                 <li class="item-1"><a href="link2.html">second item</a></li>
                 <li class="item-inactive"><a href="link3.html">third item</a></li>
                 <li class="item-1"><a href="link4.html">fourth item</a></li>
                 <li class="item-0"><a href="link5.html">fifth item</a> # 注意，此处缺少一个 </li> 闭合标签
             </ul>
         </div>
        '''

    html = etree.HTML(doc)
    print(html.xpath("//li"))
    print(html.xpath("//p")) 
	
	print(etree.tostring(html.xpath("//li[@class='item-inactive']")[0]))
	print(html.xpath("//li[@class='item-inactive']")[0].text)
	print(html.xpath("//li[@class='item-inactive']/a")[0].text)
	print(html.xpath("//li[@class='item-inactive']/a/text()"))
	print(html.xpath("//li[@class='item-inactive']/.."))
	print(html.xpath("//li[@class='item-inactive']/../li[@class='item-0']"))
```

### 2. 函数使用
contains
```python
from lxml import etree
if __name__ == '__main__':
    doc='''
        <div>
            <ul>
                 <p class="item-0 active"><a href="link1.html">first item</a></p>
                 <li class="item-1"><a href="link2.html">second item</a></li>
                 <li class="item-inactive"><a href="link3.html">third item</a></li>
                 <li class="item-1"><a href="link4.html">fourth item</a></li>
                 <li class="item-0"><a href="link5.html">fifth item</a> # 注意，此处缺少一个 </li> 闭合标签
             </ul>
         </div>
        '''
	
    html = etree.HTML(doc)
    print(html.xpath("//*[contains(@class,'item')]"))
```

starts-with
```python
from lxml import etree
if __name__ == '__main__':
    doc='''
        <div>
            <ul class='ul items'>
                 <p class="item-0 active"><a href="link1.html">first item</a></p>
                 <li class="item-1"><a href="link2.html">second item</a></li>
                 <li class="item-inactive"><a href="link3.html">third item</a></li>
                 <li class="item-1"><a href="link4.html">fourth item</a></li>
                 <li class="item-0"><a href="link5.html">fifth item</a> # 注意，此处缺少一个 </li> 闭合标签
             </ul>
         </div>
        '''
	
    html = etree.HTML(doc)
    print(html.xpath("//*[contains(@class,'item')]"))
    print(html.xpath("//*[starts-with(@class,'ul')]"))
```

text
```python
#最后一个li被限定了
print(html.xpath("//li[last()]/a/text()"))
#会得到所有的`<a>`元素的内容，因为每个<a>标签都是各自父元素的最后一个元素。
#本来每个li就只有一个<a>子元素，所以都是最后一个
print(html.xpath("//li/a[last()]/text()"))

print(html.xpath("//li/a[contains(text(),'third')]"))
```

last
```python
print(html.xpath("//li[position()=2]/a/text()"))
#结果为['third item']
```

node
```python
# 返回所有子节点
print(html.xpath("//ul/li[@class='item-inactive']/node()"))
print(html.xpath("//ul/node()"))
```

获取内容
```python
from lxml import etree
if __name__ == '__main__':
    doc='''
        <div>
            <ul class='ul items'>
                 <li class="item-0 active"><a href="link1.html">first item</a></li>
                 <li class="item-1"><a href="link2.html">second item</a></li>
                 <li class="item-inactive"><a href="link3.html">third item</a></li>
                 <li class="item-1"><a href="link4.html">fourth item</a></li>
                 <li class="item-0"><a href="link5.html">fifth item</a> # 注意，此处缺少一个 </li> 闭合标签
             </ul>
         </div>
        '''
    html = etree.XML(doc)
    print(html.xpath("//a/text()"))
    print(html.xpath("//a")[0].text)
    print(html.xpath("//ul")[0].text)
    print(len(html.xpath("//ul")[0].text))
    print(html.xpath("//ul/text()"))
```

获取属性
```python
print(html.xpath("//a/@href"))
print(html.xpath("//li/@class"))
```
## 5. Pyquery
url初始化
```python
html = '''
<div>
    <ul>
         <li class="item-0">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1 active"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
</div>
'''
from pyquery import PyQuery as pq
doc = pq(html)
print(doc)
print(type(doc))
print(doc('li'))
```

css选择器
```python
doc = pq(url="http://www.baidu.com",encoding='utf-8')
print(doc('head'))

from pyquery import PyQuery as pq
doc = pq(html)
print(doc('#container .list li'))
```

查找元素
```python
from pyquery import PyQuery as pq
doc = pq(html)
items = doc('.list')
print(type(items))
print(items)
lis = items.find('li')
print(type(lis))
print(lis)

li2 = items.children('.active')
print(li2)
```

父元素
```python
items = doc('.list')
container = items.parent()
parents = items.parents()
print(type(container))
print(container)
```

兄弟元素
```python
li = doc('.list .item-0.active')
print(li.siblings())
```

单个元素
```python
lis = doc('li').items()
print(type(lis))
for li in lis:
    print(type(li))
    print(li)
```

获取属性
```python
from pyquery import PyQuery as pq
doc = pq(html)
a = doc('.item-0.active a')
print(a)
print(a.attr('href'))
print(a.attr.href)
```

获取文本
```python
from pyquery import PyQuery as pq
doc = pq(html)
a = doc('.item-0.active a')
print(a)
print(a.text())
```
## 6. 多线程与协程


## 7. Selenium
主要为`selenium webDriver`驱动浏览器

### 1. 浏览器驱动
```python
from selenium import webdriver
from selenium.webdriver import ActionChains
import time
# webdriver为浏览器的驱动器，支持多种浏览器
browser = webdriver.Chrome()

# 浏览器访问网页页面并获取静态页面内容
browser.get('https://www.baidu.com')
print(browser.page_source)
```
### 2. 获取元素
```python
# 查找元素
input_first = browser.find_element_by_id('q')
input_second = browser.find_element_by_css_selector('#q')
input_third = browser.find_element_by_xpath('//*[@id="q"]')
# 查找元素常用方法，根据节点名称、xpath、类名、css属性查找
find_element_by_name
find_element_by_xpath
find_element_by_link_text
find_element_by_partial_link_text
find_element_by_tag_name
find_element_by_class_name
find_element_by_css_selector
# 查找单个元素通用方法
find_element(BY.ID, 'q')
input_first = browser.find_element(BY.ID, 'q')
# 查找多个元素通用方法
input_firsts = browser.find_elements_by_id('q')
```
### 3. 交互
```python
# 交互
# 填写输入框、清空输入框、点击按钮
input = browser.find_element_by_id('q')
input.send_keys('iPhone')
time.sleep(5)
input.clear()
input.send_keys('男士内裤')
button = browser.find_element_by_class_name('btn-search')
button.click()

# 交互动作，使用动作链
browser.switch_to.frame('iframeResult') # 切换到iframeResult框架
source = browser.find_element_by_css_selector('#draggable') # 找到被拖拽对象
target = browser.find_element_by_css_selector('#droppable') # 找到目标
actions = ActionChains(browser) # 动作链
actions.drag_and_drop(source, target) # 拖拉拽
actions.perform() # 执行动作
```
### 4. 执行JS
```python
# 代码执行js
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
browser.execute_script('alert("To Bottom")')

# 获取节点属性、文本、id、位置、标签名、大小
logo = browser.find_element_by_id('zh-top-link-logo')
print(logo.get_attribute('class'))

input = browser.find_element_by_class_name('zu-top-add-question')
print(input.text)
print(input.id)
print(input.location)
print(input.tag_name)
print(input.size)
```
### 5. Iframe
```python
# Frame
# 到子frame
browser.switch_to.frame('iframeResult')
# 到父frame
browser.switch_to.parent_frame()
```
### 6. 等待
```python
# 隐式等待，如果webDriver没有在dom找到元素，将继续等待，超出设定时间后才抛出异常
browser.implicitly_wait(10)#等待十秒加载不出来就会抛出异常，10秒内加载出来正常返回
browser.get('https://www.zhihu.com/explore')
input = browser.find_element_by_class_name('zu-top-add-question')

# 显式等待，指定等待条件与最长等待时间，在等待时间内满足等待条件则返回，如果在等待时间内一直不满足等待条件直至超时，则抛出异常
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
wait = WebDriverWait(browser, 10)
input = wait.until(EC.presence_of_element_located((By.ID, 'q')))
button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '.btn-search')))

#等待条件
title_is 标题是某内容
title_contains 标题包含某内容
presence_of_element_located 元素加载出，传入定位元组，如(By.ID, 'p')
visibility_of_element_located 元素可见，传入定位元组
visibility_of 可见，传入元素对象
presence_of_all_elements_located 所有元素加载出
text_to_be_present_in_element 某个元素文本包含某文字
text_to_be_present_in_element_value 某个元素值包含某文字
frame_to_be_available_and_switch_to_it frame加载并切换
invisibility_of_element_located 元素不可见
element_to_be_clickable 元素可点击
staleness_of 判断一个元素是否仍在DOM，可判断页面是否已经刷新
element_to_be_selected 元素可选择，传元素对象
element_located_to_be_selected 元素可选择，传入定位元组
element_selection_state_to_be 传入元素对象以及状态，相等返回True，否则返回False
element_located_selection_state_to_be 传入定位元组以及状态，相等返回True，否则返回False
alert_is_present 是否出现Alert
```
### 7. 浏览前进后退
```python
browser = webdriver.Chrome()
browser.get('https://www.baidu.com/')
browser.get('https://www.taobao.com/')
browser.get('https://www.python.org/')
browser.back()
time.sleep(1)
browser.forward()
browser.close()
```
### 8. Cookies
```python
browser = webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
print(browser.get_cookies())
browser.add_cookie({'name': 'name', 'domain': 'www.zhihu.com', 'value': 'germey'})
print(browser.get_cookies())
browser.delete_all_cookies()
print(browser.get_cookies())
```
### 9. 选项卡
```python
browser = webdriver.Chrome()
browser.get('https://www.baidu.com')
browser.execute_script('window.open()')
print(browser.window_handles)
browser.switch_to_window(browser.window_handles[1])
browser.get('https://www.taobao.com')
time.sleep(1)
browser.switch_to_window(browser.window_handles[0])
browser.get('http://www.fishc.com')
```
### 10. 异常处理
```python
from selenium import webdriver
from selenium.common.exceptions import TimeoutException, NoSuchElementException

browser = webdriver.Chrome()
try:
    browser.get('https://www.baidu.com')
except TimeoutException:
    print('Time Out')
try:
    browser.find_element_by_id('hello')
except NoSuchElementException:
    print('No Element')
finally:
    browser.close()
```
## 8. Scrapy