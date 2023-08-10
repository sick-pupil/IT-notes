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


## 4. Xpath


## 5. Pyquery


## 6. 多线程与协程


## 7. Selenium


## 8. Scrapy