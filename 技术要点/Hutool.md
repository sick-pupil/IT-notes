## 1. 日期时间
### 1. DateUtil
```java
//Date、Long、Calendar类型互相转换
//当前时间
Date date = DateUtil.date();
//当前时间
Date date2 = DateUtil.date(Calendar.getInstance());
//当前时间
Date date3 = DateUtil.date(System.currentTimeMillis());
//当前时间字符串，格式：yyyy-MM-dd HH:mm:ss
String now = DateUtil.now();
//当前日期字符串，格式：yyyy-MM-dd
String today= DateUtil.today();


//格式化
String dateStr = "2017-03-01";
Date date = DateUtil.parse(dateStr);
//结果 2017/03/01
String format = DateUtil.format(date, "yyyy/MM/dd");
//常用格式的格式化，结果：2017-03-01
String formatDate = DateUtil.formatDate(date);
//结果：2017-03-01 00:00:00
String formatDateTime = DateUtil.formatDateTime(date);
//结果：00:00:00
String formatTime = DateUtil.formatTime(date);


//日期时间部分获取
Date date = DateUtil.date();
//获得年的部分
DateUtil.year(date);
//获得月份，从0开始计数
DateUtil.month(date);
//获得月份枚举
DateUtil.monthEnum(date);


//开始结束日期时间
String dateStr = "2017-03-01 22:33:23";
Date date = DateUtil.parse(dateStr);
//一天的开始，结果：2017-03-01 00:00:00
Date beginOfDay = DateUtil.beginOfDay(date);
//一天的结束，结果：2017-03-01 23:59:59
Date endOfDay = DateUtil.endOfDay(date);


//时间加减计算、偏移
String dateStr = "2017-03-01 22:33:23";
Date date = DateUtil.parse(dateStr);
//结果：2017-03-03 22:33:23
Date newDate = DateUtil.offset(date, DateField.DAY_OF_MONTH, 2);
//常用偏移，结果：2017-03-04 22:33:23
DateTime newDate2 = DateUtil.offsetDay(date, 3);
//常用偏移，结果：2017-03-01 19:33:23
DateTime newDate3 = DateUtil.offsetHour(date, -3);
//昨天
DateUtil.yesterday()
//明天
DateUtil.tomorrow()
//上周
DateUtil.lastWeek()
//下周
DateUtil.nextWeek()
//上个月
DateUtil.lastMonth()
//下个月
DateUtil.nextMonth()


//日期时间差
String dateStr1 = "2017-03-01 22:33:23";
Date date1 = DateUtil.parse(dateStr1);
String dateStr2 = "2017-04-01 23:33:23";
Date date2 = DateUtil.parse(dateStr2);
//相差一个月，31天
long betweenDay = DateUtil.between(date1, date2, DateUnit.DAY);
//Level.MINUTE表示精确到分
String formatBetween = DateUtil.formatBetween(between, Level.MINUTE);
//输出：31天1小时
Console.log(formatBetween);
```

### 2. DateTime
```java
Date date = new Date();
//new方式创建
DateTime time = new DateTime(date);
Console.log(time);
//of方式创建
DateTime now = DateTime.now();
DateTime dt = DateTime.of(date);


//日期时间部分获取
DateTime dateTime = new DateTime("2017-01-05 12:34:23", DatePattern.NORM_DATETIME_FORMAT);
//年，结果：2017
int year = dateTime.year();
//季度（非季节），结果：Season.SPRING
Season season = dateTime.seasonEnum();
//月份，结果：Month.JANUARY
Month month = dateTime.monthEnum();
//日，结果：5
int day = dateTime.dayOfMonth();


//DateTime为可变对象，可设置为不可变对象
DateTime dateTime = new DateTime("2017-01-05 12:34:23", DatePattern.NORM_DATETIME_FORMAT);
//默认情况下DateTime为可变对象，此时offset == dateTime
DateTime offset = dateTime.offset(DateField.YEAR, 0);
//设置为不可变对象后变动将返回新对象，此时offset != dateTime
dateTime.setMutable(false);
offset = dateTime.offset(DateField.YEAR, 0);
```

## 2. 加密解密
```java
//hex 16进制加密解密
String str = "我是一个字符串";
String hex = HexUtil.encodeHexStr(str, CharsetUtil.CHARSET_UTF_8);
//hex是：
//e68891e698afe4b880e4b8aae5ad97e7aca6e4b8b2
String decodedStr = HexUtil.decodeHexStr(hex);
//解码后与str相同


//base62
String a = "伦家是一个非常长的字符串66";
// 17vKU8W4JMG8dQF8lk9VNnkdMOeWn4rJMva6F0XsLrrT53iKBnqo
String encode = Base62.encode(a);
// 还原为a
String decodeStr = Base62.decodeStr(encode);


//base64
String a = "伦家是一个非常长的字符串";
//5Lym5a625piv5LiA5Liq6Z2e5bi46ZW/55qE5a2X56ym5Liy
String encode = Base64.encode(a);
// 还原为a
String decodeStr = Base64.decodeStr(encode);


//base32
String a = "伦家是一个非常长的字符串";
String encode = Base32.encode(a);
Assert.assertEquals("4S6KNZNOW3TJRL7EXCAOJOFK5GOZ5ZNYXDUZLP7HTKCOLLMX46WKNZFYWI", encode);
String decodeStr = Base32.decodeStr(encode);
Assert.assertEquals(a, decodeStr);


//AES
String content = "test中文";
//随机生成密钥
byte[] key = SecureUtil.generateKey(SymmetricAlgorithm.AES.getValue()).getEncoded();
//构建
SymmetricCrypto aes = new SymmetricCrypto(SymmetricAlgorithm.AES, key);
//加密
byte[] encrypt = aes.encrypt(content);
//解密
byte[] decrypt = aes.decrypt(encrypt);
//加密为16进制表示
String encryptHex = aes.encryptHex(content);
//解密为字符串
String decryptStr = aes.decryptStr(encryptHex, CharsetUtil.CHARSET_UTF_8);


//DESede
String content = "test中文";
byte[] key = SecureUtil.generateKey(SymmetricAlgorithm.DESede.getValue()).getEncoded();
SymmetricCrypto des = new SymmetricCrypto(SymmetricAlgorithm.DESede, key);
//加密
byte[] encrypt = des.encrypt(content);
//解密
byte[] decrypt = des.decrypt(encrypt);
//加密为16进制字符串（Hex表示）
String encryptHex = des.encryptHex(content);
//解密为字符串
String decryptStr = des.decryptStr(encryptHex);


//RSA
RSA rsa = new RSA();
//获得私钥
rsa.getPrivateKey();
rsa.getPrivateKeyBase64();
//获得公钥
rsa.getPublicKey();
rsa.getPublicKeyBase64();
//公钥加密，私钥解密
byte[] encrypt = rsa.encrypt(StrUtil.bytes("我是一段测试aaaa", CharsetUtil.CHARSET_UTF_8), KeyType.PublicKey);
byte[] decrypt = rsa.decrypt(encrypt, KeyType.PrivateKey);
//私钥加密，公钥解密
byte[] encrypt2 = rsa.encrypt(StrUtil.bytes("我是一段测试aaaa", CharsetUtil.CHARSET_UTF_8), KeyType.PrivateKey);
byte[] decrypt2 = rsa.decrypt(encrypt2, KeyType.PublicKey);
```

## 3. HTTP
```java
//GET
// 最简单的HTTP请求，可以自动通过header等信息判断编码，不区分HTTP和HTTPS
String result1= HttpUtil.get("https://www.baidu.com");
// 当无法识别页面编码的时候，可以自定义请求页面的编码
String result2= HttpUtil.get("https://www.baidu.com", CharsetUtil.CHARSET_UTF_8);
//可以单独传入http参数，这样参数会自动做URL编码，拼接在URL中
HashMap<String, Object> paramMap = new HashMap<>();
paramMap.put("city", "北京");
String result3= HttpUtil.get("https://www.baidu.com", paramMap);


//POST
HashMap<String, Object> paramMap = new HashMap<>();
paramMap.put("city", "北京");
String result= HttpUtil.post("https://www.baidu.com", paramMap);


//文件上传
HashMap<String, Object> paramMap = new HashMap<>();
//文件上传只需将参数中的键指定（默认file），值设为文件对象即可，对于使用者来说，文件上传与普通表单提交并无区别
paramMap.put("file", FileUtil.file("D:\\face.jpg"));
String result= HttpUtil.post("https://www.baidu.com", paramMap);


//文件下载
String fileUrl = "http://mirrors.sohu.com/centos/8.4.2105/isos/x86_64/CentOS-8.4.2105-x86_64-dvd1.iso";
//将文件下载后保存在E盘，返回结果为下载文件大小
long size = HttpUtil.downloadFile(fileUrl, FileUtil.file("e:/"));
System.out.println("Download size: " + size);


String result2 = HttpRequest.post(url)
    .header(Header.USER_AGENT, "Hutool http")//头信息，多个头信息多次调用此方法即可
    .form(paramMap)//表单内容
    .timeout(20000)//超时，毫秒
    .execute().body();
Console.log(result2);

String json = ...;
String result2 = HttpRequest.post(url)
    .body(json)
    .execute().body();

HttpResponse res = HttpRequest.post(url)..execute();
Console.log(res.getStatus());

HttpResponse res = HttpRequest.post(url)..execute();
//预定义的头信息
Console.log(res.header(Header.CONTENT_ENCODING));
//自定义头信息
Console.log(res.header("Content-Disposition"));


```

## 4. 命令行
```java
String str = RuntimeUtil.execForStr("ipconfig");
List<String> strList = RuntimeUtil.execForStr("ipconfig");
```

## 5. ID
```java
//UUID
//生成的UUID是带-的字符串，类似于：a5c8a5e8-df2b-4706-bea4-08d0939410e3
String uuid = IdUtil.randomUUID();
//生成的是不带-的字符串，类似于：b17f24ff026d40949c85a24f4f375d42
String simpleUUID = IdUtil.simpleUUID();


//ObjectId
//生成类似：5b9e306a4df4f8c54a39fb0c
String id = ObjectId.next();
//方法2：从Hutool-4.1.14开始提供
String id2 = IdUtil.objectId();


//Snowflake
//参数1为终端ID
//参数2为数据中心ID
Snowflake snowflake = IdUtil.getSnowflake(1, 1);
long id = snowflake.nextId();
//简单使用
long id = IdUtil.getSnowflakeNextId();
String id = snowflake.getSnowflakeNextIdStr();
```
