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
## 8. Playwright
`Playwright`存在同步异步两种调用浏览器的方式：
- 同步
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("http://playwright.dev")
    page.wait_for_timeout(5000)
    page.screenshot(path="example.png")
    print(page.title())
    browser.close()
```
- 异步
```python
import asyncio
from playwright.async_api import async_playwright

async def main():
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=False, slow_mo=50)
        page = await browser.new_page()
        await page.goto("http://playwright.dev")
        print(await page.title())
        await browser.close()

asyncio.run(main())
```
- 提供对浏览器标签页的截图：`page.screenshot(path="example.png")`
- 提供使用有头浏览器，减慢执行速度：`p.chrominum.launch(headless=False, slow_mo=50)`
- `Playwright`默认为无头浏览器，可以设置`browser = p.chromium.launch(headless=False)`实现有头浏览器
- `Playwright`会自动等待页面加载完后才执行操作，1. 会让页面元素加载到`Dom`，2. 元素可见，3. 元素稳定无动画加载，4. 元素对其他元素可见，5. 元素加载成功
### 1. 交互
同步语法与异步语法的区别只在于语句前面是否存在`await`

定位元素：
- `page.get_by_role`根据元素类型查找，`textbot`、`button`、`checkbox`、`radio`
- `page.get_by_label`根据标签查找
- `page.get_by_text`根据文字内容查找
- `page.locator`自定义条件查找
#### 1. 文字输入
```python
# Text input
page.get_by_role("textbox").fill("Peter")

# Date input
page.get_by_label("Birth date").fill("2020-02-02")

# Time input
page.get_by_label("Appointment time").fill("13:15")

# Local datetime input
page.get_by_label("Local time").fill("2020-03-02T05:15")
```
#### 2. 单选多选
```python
# Check the checkbox
page.get_by_label('I agree to the terms above').check()

# Assert the checked state
assert page.get_by_label('Subscribe to newsletter').is_checked() is True

# Select the radio button
page.get_by_label('XL').check()
```

```python
# Single selection matching the value
page.get_by_label('Choose a color').select_option('blue')

# Single selection matching the label
page.get_by_label('Choose a color').select_option(label='Blue')

# Multiple selected items
page.get_by_label('Choose multiple colors').select_option(['red', 'green', 'blue'])
```
#### 3. 鼠标点击
```python
# Generic click
page.get_by_role("button").click()

# Double click
page.get_by_text("Item").dblclick()

# Right click
page.get_by_text("Item").click(button="right")

# Shift + click
page.get_by_text("Item").click(modifiers=["Shift"])

# Hover over element
page.get_by_text("Item").hover()

# Click the top left corner
page.get_by_text("Item").click(position={ "x": 0, "y": 0})

# 是否绕过可操作性检查，如元素是否已加载入dom，元素是否可视化，元素是否加载完动画，元素是否可编辑，元素是否接收事件等
page.get_by_role("button").click(force=True)
```
#### 4. 打字
```python
# Type character by character
page.locator('#area').type('Hello World!')

page.locator('div').type('Hello World!')

page.locator('.class_name').type('Hello World!')

page.locator('div.more-choice-one>div:has-text("I see"), div.more-choice-one>div:has-text("我看")')
```
#### 5. 键盘特殊键位输入
```python
# Hit Enter
page.get_by_text("Submit").press("Enter")

# Dispatch Control+Right
page.get_by_role("textbox").press("Control+ArrowRight")

# Press $ sign on keyboard
page.get_by_role("textbox").press("$")
```

```text
Backquote, Minus, Equal, Backslash, Backspace, Tab, Delete, Escape,
ArrowDown, End, Enter, Home, Insert, PageDown, PageUp, ArrowRight,
ArrowUp, F1 - F12, Digit0 - Digit9, KeyA - KeyZ, etc.
```
#### 6. 上传文件
```python
# Select one file
page.get_by_label("Upload file").set_input_files('myfile.pdf')

# Select multiple files
page.get_by_label("Upload files").set_input_files(['file1.txt', 'file2.txt'])

# Remove all the selected files
page.get_by_label("Upload file").set_input_files([])

# Upload buffer from memory
page.get_by_label("Upload file").set_input_files(
    files=[
        {"name": "test.txt", "mimeType": "text/plain", "buffer": b"this is a test"}
    ],
)
```
#### 7. 聚焦
```python
page.get_by_label('password').focus()
```
#### 8. 拖拽
```python
page.locator("#item-to-be-dragged").hover()
page.mouse.down()
page.locator("#item-to-drop-at").hover()
page.mouse.up()
```
#### 9. 处理弹窗
处理`alert`、`confirm`、`prompt`
```python
page.on("dialog", lambda dialog: dialog.accept())
page.get_by_role("button").click()

def handle_dialog(dialog):
    assert dialog.type == 'beforeunload'
    dialog.dismiss()

page.on('dialog', lambda: handle_dialog)
page.close(run_before_unload=True)
```
#### 10. 下载
```python
# 点击按钮监听下载
# Start waiting for the download
with page.expect_download() as download_info:
    # Perform the action that initiates download
    page.get_by_text("Download file").click()
# Wait for the download to start
download = download_info.value
# Wait for the download process to complete
print(download.path())
# Save downloaded file somewhere
download.save_as("/path/to/save/download/at.txt")

# 监听页面的所有下载事件
page.on("download", lambda download: print(download.path()))
```
### 2. 仿真
仿真可包括：`userAgent`、`screenSize`、`viewport`、`hasTouch`、`geolocation`、`locale`、`timezone`、`permissions`、`colorScheme`
#### 1. 设备
```python
from playwright.sync_api import sync_playwright

def run(playwright):
    iphone_13 = playwright.devices['iPhone 13']
    browser = playwright.webkit.launch(headless=False)
    context = browser.new_context(
        **iphone_13,
    )

with sync_playwright() as playwright:
    run(playwright)
```
#### 2. 窗口
```python
# Create context with given viewport
context = browser.new_context(
	viewport={ 'width': 1280, 'height': 1024 }
)

# Resize viewport for individual page
page.set_viewport_size({"width": 1600, "height": 1200})

# Emulate high-DPI
context = browser.new_context(
	viewport={ 'width': 2560, 'height': 1440 },
	device_scale_factor=2,
)
```
#### 3. 触屏
```python
context = browser.new_context(
	isMobile=false
)
```
#### 4. 区域时区
```python
context = browser.new_context(
	locale='de-DE',
	timezone_id='Europe/Berlin',
)
```
#### 5. 权限
```python
context = browser.new_context(
	permissions=['notifications'],
)

context.grant_permissions(['notifications'], origin='https://skype.com')

context.clear_permissions()
```
#### 6. 地理位置
```python
context = browser.new_context(
	geolocation={"longitude": 41.890221, "latitude": 12.492348},
	permissions=["geolocation"]
)

context.set_geolocation({"longitude": 48.858455, "latitude": 2.294474})
```
#### 7. 颜色主题
```python
# Create context with dark mode
context = browser.new_context(
	color_scheme='dark' # or 'light'
)

# Create page with dark mode
page = browser.new_page(
	color_scheme='dark' # or 'light'
)

# Change color scheme for the page
page.emulate_media(color_scheme='dark')

# Change media for page
page.emulate_media(media='print')
```
#### 8. User-Agent
```python
context = browser.new_context(
	user_agent='My user agent'
)
```
#### 9. 网络
```python
context = browser.new_context(  
	offline=True
)
```
#### 10. JS
```python
context = browser.new_context(
	javaScript_enabled=False
)

href = page.evaluate('() => document.location.href')
status = page.evaluate("""async () => {
  response = await fetch(location.href)
  return response.status
}""")

# A primitive value.
page.evaluate('num => num', 42)

# An array.
page.evaluate('array => array.length', [1, 2, 3])

# An object.
page.evaluate('object => object.foo', { 'foo': 'bar' })

# A single handle.
button = page.evaluate('window.button')
page.evaluate('button => button.textContent', button)

# Alternative notation using elementHandle.evaluate.
button.evaluate('(button, from) => button.textContent.substring(from)', 5)

# Object with multiple handles.
button1 = page.evaluate('window.button1')
button2 = page.evaluate('.button2')
page.evaluate("""o => o.button1.textContent + o.button2.textContent""",
    { 'button1': button1, 'button2': button2 })

# Object destructuring works. Note that property names must match
# between the destructured object and the argument.
# Also note the required parenthesis.
page.evaluate("""
    ({ button1, button2 }) => button1.textContent + button2.textContent""",
    { 'button1': button1, 'button2': button2 })

# Array works as well. Arbitrary names can be used for destructuring.
# Note the required parenthesis.
page.evaluate("""
    ([b1, b2]) => b1.textContent + b2.textContent""",
    [button1, button2])

# Any non-cyclic mix of serializables and handles works.
page.evaluate("""
    x => x.button1.textContent + x.list[0].textContent + String(x.foo)""",
    { 'button1': button1, 'list': [button2], 'foo': None })

data = { 'text': 'some data', 'value': 1 }
# Pass |data| as a parameter.
result = page.evaluate("""data => {
  window.myApp.use(data)
}""", data)
```
### 3. 监听事件
```python
# 每个事件起一个独立的监听器
def print_request_sent(request):
  print("Request sent: " + request.url)

def print_request_finished(request):
  print("Request finished: " + request.url)

page.on("request", print_request_sent)
page.on("requestfinished", print_request_finished)
page.goto("https://wikipedia.org")

page.remove_listener("requestfinished", print_request_finished)
page.goto("https://www.openstreetmap.org/")


# 若干个事件共用同一个监听器
page.once("dialog", lambda dialog: dialog.accept("2021"))
page.evaluate("prompt('Enter a number:')")
```
### 4. Locate定位元素
```python
# locate by role
<h3>Sign up</h3>
<label>
  <input type="checkbox" /> Subscribe
</label>
<br/>
<button>Submit</button>
expect(page.get_by_role("heading", name="Sign up")).to_be_visible()
page.get_by_role("checkbox", name="Subscribe").check()
page.get_by_role("button", name=re.compile("submit", re.IGNORECASE)).click()


<label>Password <input type="password" /></label>
# locate by label
page.get_by_label("Password").fill("secret")


<input type="email" placeholder="name@example.com" />
# locate by placeholder
page.get_by_placeholder("name@example.com").fill("playwright@microsoft.com")


<span>Welcome, John</span>
# locate by text
expect(page.get_by_text("Welcome, John")).to_be_visible()
expect(page.get_by_text("Welcome, John", exact=True)).to_be_visible()
expect(page.get_by_text(re.compile("welcome, john", re.IGNORECASE))).to_be_visible()


<img alt="playwright logo" src="/img/playwright-logo.svg" width="100" />
# locate by alt text
page.get_by_alt_text("playwright logo").click()
expect(page.get_by_title("Issues count")).to_have_text("25 issues")


<span title='Issues count'>25 issues</span>
# locate by title
expect(page.get_by_title("Issues count")).to_have_text("25 issues")
```

**重点：Locate by CSS or XPath**
```python
page.locator("css=button").click()
page.locator("xpath=//button").click()

page.locator("button").click()
page.locator("//button").click()

page.locator(
    "#tsf > div:nth-child(2) > div.A8SBwf > div.RNNXgb > div > div.a4bIc > input"
).click()
page.locator('//*[@id="tsf"]/div[2]/div[1]/div[1]/div/div[2]/input').click()
```
### 5. Filtering Locators过滤
```python
<ul>
  <li>
    <h3>Product 1</h3>
    <button>Add to cart</button>
  </li>
  <li>
    <h3>Product 2</h3>
    <button>Add to cart</button>
  </li>
</ul>
# filter by text
page.get_by_role("listitem").filter(has_text="Product 2").get_by_role(
    "button", name="Add to cart"
).click()
page.get_by_role("listitem").filter(has_text=re.compile("Product 2")).get_by_role(
    "button", name="Add to cart"
).click()


<ul>
  <li>
    <h3>Product 1</h3>
    <button>Add to cart</button>
  </li>
  <li>
    <h3>Product 2</h3>
    <button>Add to cart</button>
  </li>
</ul>
# filter by not having text
expect(page.get_by_role("listitem").filter(has_not_text="Out of stock")).to_have_count(5)


<ul>
  <li>
    <h3>Product 1</h3>
    <button>Add to cart</button>
  </li>
  <li>
    <h3>Product 2</h3>
    <button>Add to cart</button>
  </li>
</ul>
# filter by child/descendant
page.get_by_role("listitem").filter(
    has=page.get_by_role("heading", name="Product 2")
).get_by_role("button", name="Add to cart").click()
# filter by not having child/descendant
expect(
    page.get_by_role("listitem").filter(
        has_not=page.get_by_role("heading", name="Product 2")
    )
).to_have_count(1)
```
### 6. Match匹配
```python
# locator链
save_button = page.get_by_role("button", name="Save")
dialog = page.get_by_test_id("settings-dialog")
dialog.locator(save_button).click()


# 并
button = page.get_by_role("button").and_(page.getByTitle("Subscribe"))
# 或
new_email = page.get_by_role("button", name="New")
dialog = page.get_by_text("Confirm security settings")
expect(new_email.or_(dialog)).to_be_visible()
if (dialog.is_visible()):
  page.get_by_role("button", name="Dismiss").click()
new_email.click()


<button style='display: none'>Invisible</button>
<button>Visible</button>
# 找不可见的
page.locator("button").locator("visible=true").click()
```
### 7. HTTP Proxy
```python
browser = chromium.launch(proxy={
	"server": "http://myproxy.com:3128",
	"username": "usr",
	"password": "pwd"
})
```
## 9. Scrapy
<img src="D:\Project\IT-notes\Python\img\Scrapy整体架构.png" style="width:700px;height:400px;" />

### 1. 起步
```shell
scrapy startproject tutorial # 创建一个名为tutorial的爬虫项目
```

目录内容
```text
tutorial/
    scrapy.cfg            # deploy configuration file

    tutorial/             # project's Python module, you'll import your code from here
        __init__.py

        items.py          # project items definition file

        middlewares.py    # project middlewares file

        pipelines.py      # project pipelines file

        settings.py       # project settings file

        spiders/          # a directory where you'll later put your spiders
            __init__.py
```

在`spiders`目录中添加自定义`spider quotes_spider.py`
```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes" # 自定义爬虫名称，作为spider的标识

	# 自定义请求方法，返回request，爬虫从这些初始请求中生成后续请求
    def start_requests(self):
	    # 定义请求地址
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        # 发起请求并定义回调，回调处理响应
		# 不写该段代码只定义urls也可行，scrapy会默认访问urls中的地址，并把名为parse的方法作为默认回调方法
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

	# 自定义回调，解析响应
    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = f'quotes-{page}.html'
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log(f'Saved file {filename}')
```

运行爬虫
```shell
scrapy crawl quotes
```

### 2. Scrapy Shell
使用`scrapy shell`直接访问给出的`url`后通过代码访问网页中的`html`元素

```shell
scrapy shell "http://quotes.toscrape.com/page/1"
```

#### 1. css选择器
```shell
# css选择器
>>> response.css('title')
[<Selector query='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]
>>> response.css('title::text').getall()
['Quotes to Scrape']
>>> response.css('title').getall()
['<title>Quotes to Scrape</title>']
>>> response.css('title::text').get()
'Quotes to Scrape'
>>> response.css('title::text')[0].get()
'Quotes to Scrape'

# 选择class为quote的所有div节点
>>> response.css("div.quote")
[<Selector xpath="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype...'>,
 ...]

>>> quote = response.css("div.quote")[0] # 选择class为quote的第一个div节点
>>> text = quote.css("span.text::text").get() # 选择class为text的span节点文本中的第一个
>>> tags = quote.css("div.tags a.tag::text").getall() # 选择class为tags的div节点文本中的所有class为tag的a节点中的文本
```
#### 2. re选择器
```shell
# re选择器
>>> response.css('title::text').re(r'Quotes.*')
['Quotes to Scrape']
>>> response.css('title::text').re(r'Q\w+')
['Quotes']
>>> response.css('title::text').re(r'(\w+) to (\w+)')
['Quotes', 'Scrape']
```
#### 3. xpath选择器
```shell
# xpath选择器
>>> response.xpath('//title')
[<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
>>> response.xpath('//title/text()').get()
'Quotes to Scrape'
```

### 3. 导出存储
```shell
scrapy crawl quotes -O quotes.json # -O覆盖现有任何文件
scrapy crawl quotes -o quotes.jl # -o追加内容至文件中
```

### 4. 获取属性
```python
response.css('li.next a::attr(href)').get()
response.css('li.next a').attrib['href']
```

### 5. 递归请求
```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }

        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
	        # response.urljoin(next_page_url) 相当于 把start_urls中的url与next_page_url中的url进行拼接
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)
```

```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('span small::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }

        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
	        # 将残缺的url进行拼接，并生成新的scrapy.Request
            yield response.follow(next_page, callback=self.parse)
```

```python
for href in response.css('ul.pager a::attr(href)'):
    yield response.follow(href, callback=self.parse)

for a in response.css('ul.pager a'):
    yield response.follow(a, callback=self.parse)

# 循环遍历所有ul.pager a中的url并生成scrapy.Request
yield from response.follow_all(css='ul.pager a', callback=self.parse)
```

例
```python
import scrapy

class AuthorSpider(scrapy.Spider):
    name = 'author'

    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        author_page_links = response.css('.author + a')
        yield from response.follow_all(author_page_links, self.parse_author)

        pagination_links = response.css('li.next a')
        yield from response.follow_all(pagination_links, self.parse)

    def parse_author(self, response):
        def extract_with_css(query):
            return response.css(query).get(default='').strip()

        yield {
            'name': extract_with_css('h3.author-title::text'),
            'birthdate': extract_with_css('.author-born-date::text'),
            'bio': extract_with_css('.author-description::text'),
        }
```

### 6. 入参
使用`-a`传参
```shell
scrapy crawl quotes -O quotes-humor.json -a tag=humor
```

```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        url = 'http://quotes.toscrape.com/'
        tag = getattr(self, 'tag', None)
        if tag is not None:
            url = url + 'tag/' + tag
        yield scrapy.Request(url, self.parse)

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
            }

        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
            yield response.follow(next_page, self.parse)
```

### 7. 命令行
```shell
scrapy --help
Scrapy 2.10.0 - active project: tutorial

Usage:
  scrapy <command> [options] [args]

Available commands:
  bench         Run quick benchmark test
  check         Check spider contracts
  crawl         Run a spider
  edit          Edit spider
  fetch         Fetch a URL using the Scrapy downloader
  genspider     Generate new spider using pre-defined templates
  list          List available spiders
  parse         Parse URL (using its spider) and print the results
  runspider     Run a self-contained spider (without creating a project)
  settings      Get settings values
  shell         Interactive scraping console
  startproject  Create new project
  version       Print Scrapy version
  view          Open URL in browser, as seen by Scrapy
```

- `scrapy startproject <projectName> [projectDir]`：创建项目，`scrapy startproject --help`查看
- `scrapy settings`：获取或写设置，`scrapy settings --help`查看
- `scrapy runspider <file.py>`：不创建项目的情况下直接运行单个`python`文件的`spider`，`scrapy runspider --help`查看
- `scrapy shell <url>`：`url`可为空、本地文件路径、网络资源文件，获取`url`对应的`html`对象，`scrapy shell --help`查看
- `scrapy fetch <url>`：访问`url`并将结果输出，`scrapy fetch --help`查看
- `scrapy crawl <spiderName>`：运行名称为`spiderName`的爬虫项目

### 8. Spider父类常用类属性
- `name`：标识了每一个`spider`的名字，必须定义且唯一
- `start_url`：包含初始请求页面url的列表，必须定义。`start_requests()`方法会引用该属性，发出初始的`Request`
- `custom_settings`：是一个字典，每一条键值对表示一个配置，可用于覆写`SETTINGS`（`Scrapy`的全局配置模块，位于`settings.py`文件中）
- `alowed_domains`：是一个字符串列表。规定了允许爬取的网站域名，非域名下的网页将被自动过滤
- `crawler`：可以通过它访问`Scrapy`的一些组件（例如：`extensions, middlewares, settings`）
- `settings`：包含运行中时的`Spider`的配置。这和我们使用`spider.crawler.settings`访问是一样的
- `logger`：是一个`Logger`对象。根据`Spider`的`name`创建的，它记录了事件日志

常用爬虫`Spider`父类
- `CrawlSpider`
- `XmlFeedSpider`
- `CSVFeedSpider`
- `SitemapSpider`

### 9. 选择器
选择器即为`css`与`xpath`两种选择器

### 10. Item
爬虫的主要目标就是需要从非结构化的数据源中提取出结构化的数据，那么Items在其中就起到关键的作用。一般情况下，我们提取到数据都是放在一个字典中，虽然字典好用，但是缺少结构性的东西，比如说容易打错字段名称，从而容易出错

比方说在这个`Scrapy`爬虫项目中，定义了一个`Item`类，这个`Item`里边包含了`name、age、year`等字段，这样可以把爬取过来的内容通过`Item`类进行实例化，这样就不容易出错了

- `dictionaries`：词典
- `item objects`：`Item`对象
```python
from scrapy.item import Item, Field

class UserItem(Item):
name = Field()
age = Field()
```
- `dataclass objects`：数据类对象
```python
from dataclasses import dataclass

@dataclass
class UserItem:
one_field: str
another_field: int
```
- `attrs objects`：属性对象
```python
import attr

@attr.sclass CustomItem:
one_field = attr.ib()
another_field = attr.ib()
```

例
```python
from logging import log
from scrapy.spiders import CSVFeedSpider
from ..items import ExampleItem

class CsvfeedspiderSpider(CSVFeedSpider):
    name = 'CSVFeedSpider'
    allowed_domains = ['7huolianmeng.com']
    start_urls = ['http://www.7huolianmeng.com/feed.csv']
    headers = ['序号', '文章标题', '创作时间', '来源']
    delimiter = ','


    def adapt_response(self, response):
        return response.body.decode('gb18030')


    def parse_row(self, response, row):     
        i = ExampleItem()
        i['sort'] = row['序号']
        i['title'] = row['文章标题']
        i['date'] = row['创作时间']
        i['source'] = row['来源']
        self.log(i)


import scrapy

class ExampleItem(scrapy.Item):
    title=scrapy.Field()
    date=scrapy.Field()
    source=scrapy.Field()
    sort=scrapy.Field()
```

```python
from logging import log
from scrapy.spiders import CSVFeedSpider
from ..items import ExampleItem

class CsvfeedspiderSpider(CSVFeedSpider):
    name = 'CSVFeedSpider'
    allowed_domains = ['7huolianmeng.com']
    start_urls = ['http://www.7huolianmeng.com/feed.csv']
    headers = ['序号', '文章标题', '创作时间', '来源']
    delimiter = ','


    def adapt_response(self, response):
        return response.body.decode('gb18030')

    def parse_row(self, response, row):
        i = ExampleItem(title=row['文章标题'],date= row['创作时间'],source=row['来源'],sort=row['序号'])
        self.log(i)


from dataclasses import dataclass

@dataclass
class ExampleItem:
    title:str
    date:str
    source:str
    sort:str
```

```python
from logging import log
from scrapy.spiders import CSVFeedSpider
from ..items import ExampleItem

class CsvfeedspiderSpider(CSVFeedSpider):
    name = 'CSVFeedSpider'
    allowed_domains = ['7huolianmeng.com']
    start_urls = ['http://www.7huolianmeng.com/feed.csv']
    headers = ['序号', '文章标题', '创作时间', '来源']
    delimiter = ','
    
    def adapt_response(self, response):
        return response.body.decode('gb18030')

    def parse_row(self, response, row):     
        i = ExampleItem(title=row['文章标题'],date=row['创作时间'],sort=row['序号'],source=row['来源'])
        self.log(i)


import attr

@attr.s
class ExampleItem():
    title=attr.ib()
    date=attr.ib()
    source=attr.ib()
    sort=attr.ib()
```

### 11. ItemLoader
简单来说，`Items`只负责填充`Item`，而具体是怎么填充的，使用哪种机制或者手段去填充的，是由`Item`加载器负责的，即`Item Loaders`
简单来说，`Items`负责数据装载的容器，而`Item Loaders`负责装载容器的机制或者是规则

```python
import scrapy
from ..items import Article
from scrapy.loader import ItemLoader

class mySpider(scrapy.Spider):
    name="mySpider"
    start_urls = [
        "http://www.zhihu.com/people/10xiansheng/posts?page=1"
    ]

    def parse(self, response):
	    # 加载Item，生成Item对应ItemLoader
        l = ItemLoader(item=Article(), response=response)
        # 向Item对应属性设置值，填充item
        l.add_css('title', '.ListShortcut .List-item .ArticleItem .ContentItem-title a::text')
        l.add_css('content','.ListShortcut .List-item .RichContent .RichContent-inner .RichText::text')
        # l.add_xpath('name', '//div[@class="product_name"]')
		# l.add_xpath('name', '//div[@class="product_title"]')
		# l.add_xpath('price', '//p[@id="price"]')
		# l.add_css('stock', 'p#stock]')
		# 添加值示例.可以直接设置值
		# l.add_value('last_updated', 'today')
        self.log(l.load_item())
```

### 12. Pipeline管道
`Item`管道的主要责任是负责处理有爬虫从网页中抽取的`Item`，他的主要任务是清洗、验证和存储数据。当页面被蜘蛛解析后，将被发送到`Item`管道，并经过几个特定的次序处理数据。每个`Item`管道的组件都是有一个简单的方法组成的`Python`类。获取了Item并执行方法，同时还需要确定是否需要在`Item`管道中继续执行下一步或是直接丢弃掉不处理
简而言之，就是通过`spider`爬取的数据都会通过这个`pipeline`处理，可以在`pipeline`中不进行操作或者执行相关对数据的操作

`pipeline`核心方法：
- `open_spider(self, spider)`：在`Spider`开启的时候被自动调用的。在这里我们可以做一些初始化操作，如开启数据库连接等。其中，参数spider就是被开启的`Spider`对象
- `close_spider(self, spider)`：在`Spider`关闭的时候自动调用的。在这里我们可以做一些收尾工作，如关闭数据库连接等。其中，参数`spider`就是被关闭的Spider对象
- `from_crawler(cls, crawler)`：用于获取`settings`配置文件中的信息，需要注意的这个是一个类方法
- `process_item(self, item, spider)`：必须返回一个具有数据的`dict`,或者`item`对象，或者抛出`DropItem`异常，被丢弃的`item`将不会被之后的`pipeline`组件所处理

```python
#配置MongoDB数据库的连接信息
MONGO_URL = '192.168.8.30'
MONGO_PORT = 27017
MONGO_DB = 'news'

ITEM_PIPELINES = {
    'myproject.pipelines.PricePipeline': 300,
    'myproject.pipelines.JsonWriterPipeline': 800,
}
```

```python
import pymongo
 
class MongoDBPipeline(object):
    """
    1、连接数据库操作
    """
    def __init__(self,mongourl,mongoport,mongodb):
        '''
        初始化mongodb数据的url、端口号、数据库名称
        :param mongourl:
        :param mongoport:
        :param mongodb:
        '''
        self.mongourl = mongourl
        self.mongoport = mongoport
        self.mongodb = mongodb
 
    @classmethod
    def from_crawler(cls,crawler):
        """
        1、读取settings里面的mongodb数据的url、port、DB。
        :param crawler:
        :return:
        """
        return cls(
            mongourl = crawler.settings.get("MONGO_URL"),
            mongoport = crawler.settings.get("MONGO_PORT"),
            mongodb = crawler.settings.get("MONGO_DB")
        )
 
    def open_spider(self,spider):
        '''
        1、连接mongodb数据
        :param spider:
        :return:
        '''
        self.client = pymongo.MongoClient(self.mongourl,self.mongoport)
        self.db = self.client[self.mongodb]
 
    def process_item(self,item,spider):
        '''
        1、将数据写入数据库
        :param item:
        :param spider:
        :return:
        '''
        name = item.__class__.__name__
        # self.db[name].insert(dict(item))
        self.db['user'].update({'url_token':item['url_token']},{'$set':item},True)
        return item
 
    def close_spider(self,spider):
        '''
        1、关闭数据库连接
        :param spider:
        :return:
        '''
        self.client.close()
```

### 13. Middleware
`Scrapy`默认下载器：
```json
{
    'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware': 100,
    'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware': 300,
    'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware': 350,
    'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware': 400,
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': 500,
    'scrapy.downloadermiddlewares.retry.RetryMiddleware': 550,
    'scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware': 560,
    'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware': 580,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 590,
    'scrapy.downloadermiddlewares.redirect.RedirectMiddleware': 600,
    'scrapy.downloadermiddlewares.cookies.CookiesMiddleware': 700,
    'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 750,
    'scrapy.downloadermiddlewares.stats.DownloaderStats': 850,
    'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware': 900,
}
```
#### 1. Downloader Middeware
`Download Middleware`用于处理`request`与`response`的钩子框架，可以全局修改比如：代理IP、header等参数

可以在`settings.py`中设置激活该中间件：
```python
DOWNLOADERmIDDLEWARES = {
    'myproject.middlewares.Custom_A_DownloaderMiddleware': 543,
    'myproject.middlewares.Custom_B_DownloaderMiddleware': 643,
    'myproject.middlewares.Custom_B_DownloaderMiddleware': None,
}
```

- 数字越小，越靠近`engine`，越优先处理`process_request()`，越后执行`process_response()`
- 数字越大，越靠近下载器，越优先处理`process_response()`，越后执行`process_request()`
- 设置为`None`则为不激活、关闭的状态

*有时我们需要编写自己的一些下载器中间件，如使用代理，更换user-agent等，对于请求的中间件实现 `process_request(_request_, _spider_)`；对于处理回复中间件实现`process_response(_request_, _response_, _spider_)`；以及异常处理实现 `process_exception(_request_, _exception_, _spider_)`*

**更改user-agent**
```python
from faker import Faker

class UserAgent_Middleware():
    def process_request(self, request, spider):
        f = Faker()
        agent = f.firefox()
        request.headers['User-Agent'] = agent
```

**代理ip**
```python
headers = {
    'user-agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36',
    # 'Connection': 'close'
}

class Proxy_Middleware():

    def __init__(self):
        self.s = requests.session()

    def process_request(self, request, spider):

        try:
            xdaili_url = spider.settings.get('XDAILI_URL')

            r = self.s.get(xdaili_url, headers= headers)
            proxy_ip_port = r.text
            request.meta['proxy'] = 'http://' + proxy_ip_port
        except requests.exceptions.RequestException:
            print('***get xdaili fail!')
            spider.logger.error('***get xdaili fail!')

    def process_response(self, request, response, spider):
        if response.status != 200:
            try:
                xdaili_url = spider.settings.get('XDAILI_URL')

                r = self.s.get(xdaili_url, headers= headers)
                proxy_ip_port = r.text
                request.meta['proxy'] = 'http://' + proxy_ip_port
            except requests.exceptions.RequestException:
                print('***get xdaili fail!')
                spider.logger.error('***get xdaili fail!')

            return request
        return response

    def process_exception(self, request, exception, spider):

        try:
            xdaili_url = spider.settings.get('XDAILI_URL')

            r = self.s.get(xdaili_url, headers= headers)
            proxy_ip_port = r.text
            request.meta['proxy'] = 'http://' + proxy_ip_port
        except requests.exceptions.RequestException:
            print('***get xdaili fail!')
            spider.logger.error('***get xdaili fail!')

        return request
```
#### 2. Spider Middleware
`Spider`中间件用于处理`response`及`spider`生成的`item`和`request`
- 数字越小越靠近引擎，`process_spider_input()`越优先处理，`process_spider_output()`越后处理
- 数字越大越靠近`spider`，`process_spider_output()`越优先处理，`process_spider_input()`越后处理
- 关闭、不激活则使用`None`

- `process_spider_input(_response_, _spider_)`当`response`通过`spider`中间件时，这个方法被调用，返回`None`
- `process_spider_output(_response_, _result_, _spider_)`当`spider`处理`response`后返回`result`时，这个方法被调用，必须返回`Request`或`Item`对象的可迭代对象，一般返回`result`
- `process_spider_exception(_response_, _exception_, _spider_)`当`spider`中间件抛出异常时，这个方法被调用，返回`None`或可迭代对象的`Request`、`dict`、`Item`
#### 3. Retry Middleware
```python
from scrapy.downloadermiddlewares.retry import RetryMiddleware
from scrapy.utils.response import response_status_message

class My_RetryMiddleware(RetryMiddleware):

    def process_response(self, request, response, spider):
        if request.meta.get('dont_retry', False):
            return response

        if response.status in self.retry_http_codes:
            reason = response_status_message(response.status)
            try:
                xdaili_url = spider.settings.get('XDAILI_URL')

                r = requests.get(xdaili_url)
                proxy_ip_port = r.text
                request.meta['proxy'] = 'https://' + proxy_ip_port
            except requests.exceptions.RequestException:
                print('获取讯代理ip失败！')
                spider.logger.error('获取讯代理ip失败！')

            return self._retry(request, reason, spider) or response
        return response


    def process_exception(self, request, exception, spider):
        if isinstance(exception, self.EXCEPTIONS_TO_RETRY) and not request.meta.get('dont_retry', False):
            try:
                xdaili_url = spider.settings.get('XDAILI_URL')

                r = requests.get(xdaili_url)
                proxy_ip_port = r.text
                request.meta['proxy'] = 'https://' + proxy_ip_port
            except requests.exceptions.RequestException:
                print('获取讯代理ip失败！')
                spider.logger.error('获取讯代理ip失败！')

            return self._retry(request, exception, spider)
```

### 14. 导出文件
- `FEED_URI`：指定输出文件
- `FEED_FORMAT`：指定数据格式，`JsonItemExporter`读取整块`Item`对象进入内存、`JsonLinesItemExport`分片读取整块`Item`对象进入内存、`CsvItemExporter`输出`CSV`格式、`XmlItemExporter`输出`XML`格式
- `FEED_STORAGES`：额外存储方式，即指定对应存储形式的实现类
- `FEED_STORAGES_BASE`：基础存储方式，即指定对应存储形式的实现类
```json
{
    '': 'scrapy.extensions.feedexport.FileFeedStorage',
    'file': 'scrapy.extensions.feedexport.FileFeedStorage',
    'stdout': 'scrapy.extensions.feedexport.StdoutFeedStorage',
    's3': 'scrapy.extensions.feedexport.S3FeedStorage',
    'ftp': 'scrapy.extensions.feedexport.FTPFeedStorage',
}
```
- `FEED_EXPORTERS`：额外输出方式，即指定对应文件输出格式的实现类
- `FEED_EXPORTERS_BASE`：基础输出方式，即指定对应文件输出格式的实现类
```json
{
    'json': 'scrapy.exporters.JsonItemExporter',
    'jsonlines': 'scrapy.exporters.JsonLinesItemExporter',
    'jl': 'scrapy.exporters.JsonLinesItemExporter',
    'csv': 'scrapy.exporters.CsvItemExporter',
    'xml': 'scrapy.exporters.XmlItemExporter',
    'marshal': 'scrapy.exporters.MarshalItemExporter',
    'pickle': 'scrapy.exporters.PickleItemExporter',
}
```
- `FEED_EXPORT_ENCODING`：文件编码格式，默认`None`，一般设置为`utf-8`
- `FEED_EXPORT_FIELDS`：指定数据输出项及顺序，例`FEED_EXPORT_FIELDS = ["foo", "bar", "baz"]`
- `FEED_EXPORT_INDENT`：默认值为0，单值为0或负数时将在新一行输出数据，设置大于0则为每一级的数据添加等量倍的空格缩进

例
```python
# -*- coding: utf-8 -*-
import scrapy


class QuotesItem(scrapy.Item):
    text = scrapy.Field()
    author = scrapy.Field()


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    allowed_domains = ['toscrape.com']
    custom_settings = {
        'FEED_EXPORT_ENCODING': 'utf-8',
        'FEED_URI': 'quotes.jsonlines',
    }

    def __init__(self, category=None, *args, **kwargs):
        super(QuotesSpider, self).__init__(*args, **kwargs)
        self.start_urls = ['http://quotes.toscrape.com/tag/%s/' % category, ]

    def parse(self, response):
        quote_block = response.css('div.quote')
        for quote in quote_block:
            text = quote.css('span.text::text').extract_first()
            author = quote.xpath('span/small/text()').extract_first()
            # item = dict(text=text, author=author)
            item = QuotesItem()
            item['text'] = text
            item['author'] = author
            yield item

        next_page = response.css('li.next a::attr("href")').extract_first()
        if next_page is not None:
            yield response.follow(next_page, self.parse)
```

