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
|   |   |
|---|---|
|^|匹配字符串的开头|
|$|匹配字符串的末尾。|
|.|匹配任意字符，除了换行符，当re.DOTALL标记被指定时，则可以匹配包括换行符的任意字符。|
|[...]|用来表示一组字符,单独列出：[amk] 匹配 'a'，'m'或'k'|
|[^...]|不在[]中的字符：[^abc] 匹配除了a,b,c之外的字符。|
|re*|匹配0个或多个的表达式。|
|re+|匹配1个或多个的表达式。|
|re?|匹配0个或1个由前面的正则表达式定义的片段，非贪婪方式|
|re{ n}|匹配n个前面表达式。例如，"o{2}"不能匹配"Bob"中的"o"，但是能匹配"food"中的两个o。|
|re{ n,}|精确匹配n个前面表达式。例如，"o{2,}"不能匹配"Bob"中的"o"，但能匹配"foooood"中的所有o。"o{1,}"等价于"o+"。"o{0,}"则等价于"o*"。|
|re{ n, m}|匹配 n 到 m 次由前面的正则表达式定义的片段，贪婪方式|
|a\|b|匹配a或b|
|(re)|匹配括号内的表达式，也表示一个组|
|(?imx)|正则表达式包含三种可选标志：i, m, 或 x 。只影响括号中的区域。|
|(?-imx)|正则表达式关闭 i, m, 或 x 可选标志。只影响括号中的区域。|
|(?: re)|类似 (...), 但是不表示一个组|
|(?imx: re)|在括号中使用i, m, 或 x 可选标志|
|(?-imx: re)|在括号中不使用i, m, 或 x 可选标志|
|(?#...)|注释.|
|(?= re)|前向肯定界定符。如果所含正则表达式，以 ... 表示，在当前位置成功匹配时成功，否则失败。但一旦所含表达式已经尝试，匹配引擎根本没有提高；模式的剩余部分还要尝试界定符的右边。|
|(?! re)|前向否定界定符。与肯定界定符相反；当所含表达式不能在字符串当前位置匹配时成功。|
|(?> re)|匹配的独立模式，省去回溯。|
|\w|匹配数字字母下划线|
|\W|匹配非数字字母下划线|
|\s|匹配任意空白字符，等价于 [\t\n\r\f]。|
|\S|匹配任意非空字符|
|\d|匹配任意数字，等价于 [0-9]。|
|\D|匹配任意非数字|
|\A|匹配字符串开始|
|\Z|匹配字符串结束，如果是存在换行，只匹配到换行前的结束字符串。|
|\z|匹配字符串结束|
|\G|匹配最后匹配完成的位置。|
|\b|匹配一个单词边界，也就是指单词和空格间的位置。例如， 'er\b' 可以匹配"never" 中的 'er'，但不能匹配 "verb" 中的 'er'。|
|\B|匹配非单词边界。'er\B' 能匹配 "verb" 中的 'er'，但不能匹配 "never" 中的 'er'。|
|\n, \t, 等。|匹配一个换行符。匹配一个制表符, 等|
|\1...\9|匹配第n个分组的内容。|
|\10|匹配第n个分组的内容，如果它经匹配。否则指的是八进制字符码的表达式。|

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
	print "matchObj.group() : ", matchObj.group() 
	print "matchObj.group(1) : ", matchObj.group(1) 
	print "matchObj.group(2) : ", matchObj.group(2) 
else: 
	print "No match!!"
	
print(re.search('www', 'www.runoob.com').span())  # 在起始位置匹配
print(re.search('com', 'www.runoob.com').span())  # 不在起始位置匹配

line = "Cats are smarter than dogs";
searchObj = re.search( r'(.*) are (.*?) .*', line, re.M|re.I)
if searchObj: 
	print "searchObj.group() : ", searchObj.group()
	print "searchObj.group(1) : ", searchObj.group(1)
	print "searchObj.group(2) : ", searchObj.group(2)
else: 
	print "Nothing found!!"
	
	
# 删除字符串中的 Python注释
num = re.sub(r'#.*$', "", phone)
print "电话号码是: ", num 
# 删除非数字(-)的字符串
num = re.sub(r'\D', "", phone)
print "电话号码是 : ", num

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
	print (match.group())
```
## 3. Beautiful Soup


## 4. Xpath


## 5. Pyquery


## 6. 多线程与协程


## 7. Selenium


## 8. Scrapy