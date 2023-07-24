## 1. Lang
<img src="D:\Project\IT-notes\技术要点\img\apache-commons-lang.png" style="width:700px;height:350px;" />

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

### 1. 日期时间
#### 1. 字符串转日期
```java
final String strDate = "2021-07-04 11:11:11";
final String pattern = "yyyy-MM-dd HH:mm:ss";
Date date2 = DateUtils.parseDate(strDate, pattern);
```

#### 2. 日期转字符串
```java
final Date date = new Date();
final String pattern = "yyyy年MM月dd日";
String strDate = DateFormatUtils.format(date, pattern);
```

#### 3. 日期计算
```java
final Date date = new Date();
Date newDate1 = DateUtils.addDays(date, 5); // 加5天
Date newDate2 = DateUtils.addHours(date, -5); // 减5小时
Date newDate3 = DateUtils.truncate(date, Calendar.DATE); // 过滤时分秒
boolean isSameDay = DateUtils.isSameDay(newDate1, newDate2); // 判断是否是同一天
```

### 2. 字符串
#### 1. 判空
```java
//判空
boolean isEmpty = StringUtils.isEmpty(str);
boolean isNotEmpty = StringUtils.isNotEmpty(str);
//去除空格后判空
boolean isBlank = StringUtils.isBlank(str);
boolean isNotBlank = StringUtils.isNotBlank(str);
//只要有一个为空则true
boolean isAnyEmpty = StringUtils.isAnyEmpty(str1, str2, str3);
//所有都为空则true
boolean isAllEmpty = StringUtils.isAllEmpty(str1, str2, str3);
```

#### 2. 去空格
```java
//去除两侧空格并返回新字符串
String newStr = StringUtils.trim();
//去除两侧空格后如果为null则返回空字符串
newStr = StringUtils.trimToEmpty(str);
//去除两侧空格后如果为空字符串则返回null
newStr = StringUtils.trimToNull(str);
//去除两侧指定的任意字符，如去除两侧的'a' 'b' 'c'字符
newStr = StringUtils.strip(str, "abc");
//去除左侧指定的任意字符
newStr = StringUtils.stripStart(str, "abc");
//去除右侧指定的任意字符
newStr = StringUtils.stripEnd(str, "abc");
```

#### 3. 分割
```java
//默认指定空格为分隔符
StringUtils.split(str);
//指定分隔符并分割字符串
StringUtils.split(str, ",");
```

#### 4. 取子字符串
```java
//获得"aa.bb.cc"字符串中最后一个'.'之前的字符串
StringUtils.substringBeforeLast("aa.bb.cc", "."); //aa.bb
//获得"aa.bb.cc"字符串中最后一个'.'之后的字符串
StringUtils.substringAfterLast("aa.bb.cc", "."); //cc
//获得"aa.bb.cc"字符串中第一个'.'之前的字符串
StringUtils.substringBefore("aa.bb.cc", "."); //aa
//获得"aa.bb.cc"字符串中第一个'.'之后的字符串
StringUtils.substringAfter("aa.bb.cc", "."); //cc
// 获取"ab.cc.txt"中.之间的字符串
StringUtils.substringBetween("ab.cc.txt", "."); //cc
StringUtils.substringBetween("a(bb)c", "(", ")"); //bb
```

#### 5. 其他
```java
// 首字母大写
StringUtils.capitalize("test"); // Test
// 字符串合并
StringUtils.join(new int[]{1,2,3}, ",");// 1,2,3
// 缩写
StringUtils.abbreviate("abcdefg", 6);// "abc..."
// 判断字符串是否是数字
StringUtils.isNumeric("abc123");// false
// 删除指定字符
StringUtils.remove("abbc", "b"); // ac

// 随机生成长度为5的字符串
RandomStringUtils.random(5);
// 随机生成长度为5的"只含大小写字母"字符串
RandomStringUtils.randomAlphabetic(5);
// 随机生成长度为5的"只含大小写字母和数字"字符串
RandomStringUtils.randomAlphanumeric(5);
// 随机生成长度为5的"只含数字"字符串
RandomStringUtils.randomNumeric(5);
```

### 3. 反射
```java
//读写属性
String value2 = (String) FieldUtils.readDeclaredField(reflectDemo, "abc", true); //123
public class ReflectDemo {
    private static String sAbc = "111";
    private String abc = "123";
    public void fieldRelated() throws Exception {
        ReflectDemo reflectDemo = new ReflectDemo();
        // 反射获取对象属性的值
        String value2 = (String) FieldUtils.readField(reflectDemo, "abc", true);//123
        // 反射获取类静态属性的值
        String value3 = (String) FieldUtils.readStaticField(ReflectDemo.class, "sAbc", true);//111
        // 反射设置对象属性值
        FieldUtils.writeField(reflectDemo, "abc", "newValue", true);
        // 反射设置类静态属性的值
        FieldUtils.writeStaticField(ReflectDemo.class, "sAbc", "newStaticValue", true);
    }
}

//获取被注解注释的方法
// 原生写法
List<Method> annotatedMethods = new ArrayList<Method>();
for (Method method : ReflectDemo.class.getMethods()) {
    if (method.getAnnotation(Test.class) != null) {
        annotatedMethods.add(method);
    }
}
// commons写法
Method[] methods = MethodUtils.getMethodsWithAnnotation(ReflectDemo.class, Test.class);


//调用方法
private static void testStaticMethod(String param1) {}
private void testMethod(String param1) {}
public void invokeDemo() throws Exception {
    // 调用函数"testMethod"
    ReflectDemo reflectDemo = new ReflectDemo();
    // 原生写法
    Method testMethod = reflectDemo.getClass().getDeclaredMethod("testMethod");
    testMethod.setAccessible(true); // 设置访问级别，如果private函数不设置则调用会报错
    testMethod.invoke(reflectDemo, "testParam");
    // commons写法
    MethodUtils.invokeExactMethod(reflectDemo, "testMethod", "testParam");
    
    // ---------- 类似方法 ----------
    // 调用static方法
    MethodUtils.invokeExactStaticMethod(ReflectDemo.class, "testStaticMethod", "testParam");
    // 调用方法(含继承过来的方法)
    MethodUtils.invokeMethod(reflectDemo, "testMethod", "testParam");
    // 调用static方法(当前不存在则向父类寻找匹配的静态方法)
    MethodUtils.invokeStaticMethod(ReflectDemo.class, "testStaticMethod", "testParam");
}
```

### 4. 系统
```java
// 判断操作系统类型
boolean isWin = SystemUtils.IS_OS_WINDOWS;
boolean isWin10 = SystemUtils.IS_OS_WINDOWS_10;
boolean isWin2012 = SystemUtils.IS_OS_WINDOWS_2012;
boolean isMac = SystemUtils.IS_OS_MAC;
boolean isLinux = SystemUtils.IS_OS_LINUX;
boolean isUnix = SystemUtils.IS_OS_UNIX;
boolean isSolaris = SystemUtils.IS_OS_SOLARIS;
// ... ...

// 判断java版本
boolean isJava6 = SystemUtils.IS_JAVA_1_6;
boolean isJava8 = SystemUtils.IS_JAVA_1_8;
boolean isJava11 = SystemUtils.IS_JAVA_11;
boolean isJava14 = SystemUtils.IS_JAVA_14;
// ... ...

// 获取java相关目录
File javaHome = SystemUtils.getJavaHome();
File userHome = SystemUtils.getUserHome();// 操作系统用户目录
File userDir = SystemUtils.getUserDir();// 项目所在路径
File tmpDir = SystemUtils.getJavaIoTmpDir();
```

## 2. IO
<img src="D:\Project\IT-notes\技术要点\img\apache-commons-io.png" style="width:700px;height:250px;" />

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

### 1. 输入输出流
```java
InputStream inputStream = new FileInputStream("test.txt");
OutputStream outputStream = new FileOutputStream("test.txt");
// 原生写法
if (inputStream != null) {
    try {
        inputStream.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
if (outputStream != null) {
    try {
        outputStream.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
// commons写法(可以传任意数量的流)
IOUtils.closeQuietly(inputStream, outputStream);


// ==== 输入流转换为byte数组 ====
// 原生写法
InputStream is = new FileInputStream("foo.txt");
byte[] buf = new byte[1024];
int len;
ByteArrayOutputStream out = new ByteArrayOutputStream();
while ((len = is.read(buf)) != -1) {
    out.write(buf, 0, len);
}
byte[] result = out.toByteArray();
// commons写法，输入流读出字节
byte[] result2 = IOUtils.toByteArray(is);
// commons写法，输入流读出字符串
String result2 = IOUtils.toString(is, "UTF-8");
// 将reader转换为字符串
String toString(Reader reader, String charset) throws IOException;
// 将url转换为字符串，也就是可以直接将网络上的内容下载为字符串
String toString(URL url, String charset) throws IOException;


// 按照行读取结果
InputStream is = new FileInputStream("test.txt");
List<String> lines = IOUtils.readLines(is, "UTF-8");

// 将行集合写入输出流
OutputStream os = new FileOutputStream("newTest.txt");
IOUtils.writeLines(lines, System.lineSeparator(), os, "UTF-8");

// 拷贝输入流到输出流
InputStream inputStream = new FileInputStream("src.txt");
OutputStream outputStream = new FileOutputStream("dest.txt");
IOUtils.copy(inputStream, outputStream);

//自动关闭输入流
InputStream is = new FileInputStream("test.txt");
AutoCloseInputStream acis = new AutoCloseInputStream(is);
IOUtils.toByteArray(acis); // 将流全部读完

// 从后向前按行读取
try (ReversedLinesFileReader reader = new ReversedLinesFileReader(new File("test.txt"), Charset.forName("UTF-8"))) {
    String lastLine = reader.readLine(); // 读取最后一行
    List<String> line5 = reader.readLines(5); // 从后再读5行
}

//计数流
InputStream is = new FileInputStream("test.txt");
try (CountingInputStream cis = new CountingInputStream(is)) {
	String txt = IOUtils.toString(cis, "UTF-8"); // 文件内容
	long size = cis.getByteCount(); // 读取的字节数
} catch (IOException e) {
	// 异常处理
}


//可观察的输入流（典型的观察者模式），可实现边读取边处理
private class MyObservableInputStream extends ObservableInputStream {
    class MyObserver extends Observer {
        @Override
        public void data(final int input) throws IOException {
            // 做自定义处理
        }
        @Override
        public void data(final byte[] input, final int offset, final int length) throws IOException {
            // 做自定义处理
        }
    }
    public MyObservableInputStream(InputStream inputStream) {
        super(inputStream);
    }
}


//BOMInputStream: 同时读取文本文件的bom头

//BoundedInputStream：有界的流，控制只允许读取前x个字节

//BrokenInputStream: 一个错误流，永远抛出IOException

//CharSequenceInputStream: 支持StringBuilder,StringBuffer等读取

//LockableFileWriter: 带锁的Writer，同一个文件同时只允许一个流写入，多余的写入操作会跑出IOException

//StringBuilderWriter: StringBuilder的Writer
```

### 2. 文件读写
**FileUtils、FilenameUtils、PathUtils**
```java
File readFile = new File("test.txt");
// 读取文件
String str = FileUtils.readFileToString(readFile, "UTF-8");
// 读取文件为字节数组
byte[] bytes = FileUtils.readFileToByteArray(readFile);
// 按行读取文件
List<String> lines =  FileUtils.readLines(readFile, "UTF-8");

File writeFile = new File("out.txt");
// 将字符串写入文件
FileUtils.writeStringToFile(writeFile, "测试文本", "UTF-8");
// 将字节数组写入文件
FileUtils.writeByteArrayToFile(writeFile, bytes);
// 将字符串列表一行一行写入文件
FileUtils.writeLines(writeFile, lines, "UTF-8");
```

### 3. 文件移动复制
```java
File srcFile = new File("src.txt");
File destFile = new File("dest.txt");
File srcDir = new File("/srcDir");
File destDir = new File("/destDir");
// 移动/拷贝文件
FileUtils.moveFile(srcFile, destFile);
FileUtils.copyFile(srcFile, destFile);
// 移动/拷贝文件到目录
FileUtils.moveFileToDirectory(srcFile, destDir, true);
FileUtils.copyFileToDirectory(srcFile, destDir);
// 移动/拷贝目录
FileUtils.moveDirectory(srcDir, destDir);
FileUtils.copyDirectory(srcDir, destDir);
// 拷贝网络资源到文件
FileUtils.copyURLToFile(new URL("http://xx"), destFile);
// 拷贝流到文件
FileUtils.copyInputStreamToFile(new FileInputStream("test.txt"), destFile);
```

### 4. 文件名
```java
// 获取名称，后缀等
String name = "/home/xxx/test.txt";
FilenameUtils.getName(name); // "test.txt"
FilenameUtils.getBaseName(name); // "test"
FilenameUtils.getExtension(name); // "txt"
FilenameUtils.getPath(name); // "/home/xxx/"
```

### 5. Path
```java
// 获取当前路径
Path path = PathUtils.current();
// 删除path
PathUtils.delete(path);
// 路径或文件是否为空
PathUtils.isEmpty(path);
// 设置只读
PathUtils.setReadOnly(path, true);
// 复制
PathUtils.copyFileToDirectory(Paths.get("test.txt"), path);
PathUtils.copyDirectory(Paths.get("/srcPath"), Paths.get("/destPath"));
// 统计目录内文件数量
Counters.PathCounters counters = PathUtils.countDirectory(path);
counters.getByteCounter(); // 字节大小
counters.getDirectoryCounter(); // 目录个数
counters.getFileCounter(); // 文件个数
```

### 6. 文件排序器
```java
List<File> files = Arrays.asList(new File[]{
        new File("/foo/def"),
        new File("/foo/test.txt"),
        new File("/foo/abc"),
        new File("/foo/hh.txt")});
// 排序目录在前
Collections.sort(files, DirectoryFileComparator.DIRECTORY_COMPARATOR); // ["/foo/def", "/foo/abc", "/foo/test.txt", "/foo/hh.txt"]
// 排序目录在后
Collections.sort(files, DirectoryFileComparator.DIRECTORY_REVERSE); // ["/foo/test.txt", "/foo/hh.txt", "/foo/def", "/foo/abc"]
// 组合排序，首先按目录在前排序，其次再按照名称排序
Comparator dirAndNameComp = new CompositeFileComparator(
            DirectoryFileComparator.DIRECTORY_COMPARATOR,
            NameFileComparator.NAME_COMPARATOR);
Collections.sort(files, dirAndNameComp); // ["/foo/abc", "/foo/def", "/foo/hh.txt", "/foo/test.txt"]
```

### 7. 文件监听器
```java
public static void main(String[] args) throws Exception {
    // 监听目录下文件变化。可通过参数控制监听某些文件，默认监听目录所有文件
    FileAlterationObserver observer = new FileAlterationObserver("/foo");
    observer.addListener(new myListener());
    FileAlterationMonitor monitor = new FileAlterationMonitor();
    monitor.addObserver(observer);
    monitor.start(); // 启动监视器
    Thread.currentThread().join(); // 避免主线程退出造成监视器退出
}

private class myListener extends FileAlterationListenerAdaptor {
    @Override
    public void onFileCreate(File file) {
        System.out.println("fileCreated:" + file.getAbsolutePath());
    }
    @Override
    public void onFileChange(File file) {
        System.out.println("fileChanged:" + file.getAbsolutePath());
    }
    @Override
    public void onFileDelete(File file) {
        System.out.println("fileDeleted:" + file.getAbsolutePath());
    }
}
```

### 8. 其他
```java
File file = new File("test.txt");
File dir = new File("/test");
// 删除文件
FileUtils.delete(file);
// 删除目录
FileUtils.deleteDirectory(dir);
// 文件大小，如果是目录则递归计算总大小
long s = FileUtils.sizeOf(file);
// 则递归计算目录总大小，参数不是目录会抛出异常
long sd = FileUtils.sizeOfDirectory(dir);
// 递归获取目录下的所有文件
Collection<File> files = FileUtils.listFiles(dir, null, true);
// 获取jvm中的io临时目录
FileUtils.getTempDirectory();
```

## 3. Codec
<img src="D:\Project\IT-notes\技术要点\img\apache-commons-codec.png" style="width:700px;height:200px;" />

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
</dependency>
```

### 1. Hex
```java
// byte数组转为16进制字符串
String hex = Hex.encodeHexString("123".getBytes());
System.out.println(hex);
// 16进制字符串解码
byte[] src = Hex.decodeHex(hex);
System.out.println(new String(src));
```

### 2. Base64/Base32/Base16
```java
// base64编码
String base64 = Base64.encodeBase64String("测试".getBytes());
System.out.println(base64);
// base64解码
byte[] src = Base64.decodeBase64(base64);
System.out.println(new String(src));
// 字符串是否是base64
Base64.isBase64(base64);



// 以流方式提供Base64编码和解码
// 附："123"的base64编码为"MTIz"
// 对输入流做base64编码
InputStream is = new ByteArrayInputStream("123".getBytes());
Base64InputStream ebis = new Base64InputStream(is, true);
String enc = IOUtils.toString(ebis, "UTF-8"); // MTIz
// 对base64数据流做解码
is = new ByteArrayInputStream(enc.getBytes());
Base64InputStream dbis = new Base64InputStream(is, false);
String dec = IOUtils.toString(dbis, "UTF-8"); // 123

// 将数据做base64编码写入输出流
final String data = "123";
ByteArrayOutputStream baos = new ByteArrayOutputStream();
Base64OutputStream ebos = new Base64OutputStream(baos, true);
IOUtils.write(data, ebos, "UTF-8");
String enc2 = baos.toString(); // MTIz
// 将base64数据做解码写入输出流
baos = new ByteArrayOutputStream();
Base64OutputStream dbos = new Base64OutputStream(baos, false);
IOUtils.write(data, dbos, "UTF-8");
String dec2 = dbos.toString(); // 123
```

### 3. URL
```java
URLCodec urlCodec = new URLCodec();
// url编码
String encUrl = urlCodec.encode("http://x.com?f=哈");
System.out.println(encUrl);
// url解码
String decUrl = urlCodec.decode(encUrl);
System.out.println(decUrl);
```

### 4. MD/SHA/HMAC
```java
String md5 = DigestUtils.md5Hex("测试");

String sha1 = DigestUtils.sha1Hex("测试");
String sha256 = DigestUtils.sha256Hex("测试");
String sha384 = DigestUtils.sha384Hex("测试");
String sha512 = DigestUtils.sha512Hex("测试");
String sha3_256 = DigestUtils.sha3_256Hex("测试");
String sha3_384 = DigestUtils.sha3_384Hex("测试");
String sha3_512 = DigestUtils.sha3_512Hex("测试");

String key = "asdf3234asdf3234asdf3234asdf3234";
String valueToDigest = "测试数据"; // valueToDigest参数支持字节数据，流，文件等
// 做HMAC-MD5摘要
String hmacMd5 = new HmacUtils(HmacAlgorithms.HMAC_MD5, key).hmacHex(valueToDigest);
// 做HMAC-sha摘要
String hmacSha256 = new HmacUtils(HmacAlgorithms.HMAC_SHA_256, key).hmacHex(valueToDigest);
String hmacSha384 = new HmacUtils(HmacAlgorithms.HMAC_SHA_384, key).hmacHex(valueToDigest);
String hmacSha512 = new HmacUtils(HmacAlgorithms.HMAC_SHA_512, key).hmacHex(valueToDigest);
```

## 4. Compress
<img src="D:\Project\IT-notes\技术要点\img\apache-commons-compress.png" style="width:700px;height:200px;" />

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-compress</artifactId>
    <version>1.21</version>
</dependency>
```

### 1. 压缩
- `.gz`：`GzipCompressorOutputStream、GzipCompressorInputStream`
- `.bz2`：`BZip2CompressorOutputStream、BZip2CompressorInputStream`
- `.xz`：`XZCompressorOutputStream、XZCompressorInputStream`
- `.lz4`：`FramedLZ4CompressorOutputStream、FramedLZ4CompressorInputStream`
- `.block_lz4`：`BlockLZ4CompressorOutputStream、BlockLZ4CompressorInputStream`
- `.pack`：`Pack200CompressorOutputStream、Pack200CompressorInputStream`
- `.deflate`：`DeflateCompressorOutputStream、DeflateCompressorInputStream`
- `.lzma`：`LZMACompressorOutputStream、LZMACompressorInputStream`
- `.sz`：`FramedSnappyCompressorOutputStream、FramedSnappyCompressorInputStream`
- `.z`：`ZCompressorInputStream`

```java
// gzip压缩
String file = "/test.js";
GzipParameters parameters = new GzipParameters();
parameters.setCompressionLevel(Deflater.BEST_COMPRESSION);
parameters.setOperatingSystem(3);
parameters.setFilename(FilenameUtils.getName(file));
parameters.setComment("Test file");
parameters.setModificationTime(System.currentTimeMillis());
FileOutputStream fos = new FileOutputStream(file + ".gz");
try (GzipCompressorOutputStream gzos = new GzipCompressorOutputStream(fos, parameters);
    InputStream is = new FileInputStream(file)) {
    IOUtils.copy(is, gzos);
}
// gzip解压
String gzFile = "/test.js.gz";
FileInputStream is = new FileInputStream(gzFile);
try (GzipCompressorInputStream gis = new GzipCompressorInputStream(is)) {
    GzipParameters p = gis.getMetaData();
    File targetFile = new File("/test.js");
    FileUtils.copyToFile(gis, targetFile);
    targetFile.setLastModified(p.getModificationTime());
}


// 压缩bz2
String srcFile = "/test.tar";
String targetFile = "/test.tar.bz2";
FileOutputStream os = new FileOutputStream(targetFile);
try (BZip2CompressorOutputStream bzos = new BZip2CompressorOutputStream(os);
    InputStream is = new FileInputStream(srcFile)) {
    IOUtils.copy(is, bzos);
}
// 解压bz2
String bzFile = "/test.tar.bz2";
FileInputStream is = new FileInputStream(bzFile);
try (BZip2CompressorInputStream bzis = new BZip2CompressorInputStream(is)) {
    File targetFile = new File("test.tar");
    FileUtils.copyToFile(bzis, targetFile);
}
```

### 2. 归档
- `.tar`：`TarArchiveOutputStream、TarArchiveInputStream`
- `.zip`：`ZipArchiveOutputStream、ZipArchiveInputStream`
- `.jar`：`JarArchiveOutputStream、JarArchiveInputStream`
- `.dump`：`DumpArchiveOutputStream、DumpArchiveInputStream`
- `.cpio`：`CpioArchiveOutputStream、CpioArchiveInputStream`
- `.ar`：`ArArchiveOutputStream、ArArchiveInputStream`
- `.arj`：`ArjArchiveInputStream`
- `.7z`：`SevenZOutputFile、SevenZFile`

```java
// tar压缩
public void tar() throws IOException {
    File srcDir = new File("/test");
    String targetFile = "/test.tar";
    try (TarArchiveOutputStream tos = new TarArchiveOutputStream(
            new FileOutputStream(targetFile))) {
        tarRecursive(tos, srcDir, "");
    }
}
// 递归压缩目录下的文件和目录
private void tarRecursive(TarArchiveOutputStream tos, File srcFile, String basePath) 
	throws IOException {
    if (srcFile.isDirectory()) {
        File[] files = srcFile.listFiles();
        String nextBasePath = basePath + srcFile.getName() + "/";
        if (ArrayUtils.isEmpty(files)) {
            // 空目录
            TarArchiveEntry entry = new TarArchiveEntry(srcFile, nextBasePath);
            tos.putArchiveEntry(entry);
            tos.closeArchiveEntry();
        } else {
            for (File file : files) {
                tarRecursive(tos, file, nextBasePath);
            }
        }
    } else {
        TarArchiveEntry entry = new TarArchiveEntry(srcFile, basePath + srcFile.getName());
        tos.putArchiveEntry(entry);
        FileUtils.copyFile(srcFile, tos);
        tos.closeArchiveEntry();
    }
}
// tar解压
public void untar() throws IOException {
    InputStream is = new FileInputStream("/test.tar");
    String outPath = "/test";
    try (TarArchiveInputStream tis = new TarArchiveInputStream(is)) {
        TarArchiveEntry nextEntry;
        while ((nextEntry = tis.getNextTarEntry()) != null) {
            String name = nextEntry.getName();
            File file = new File(outPath, name);
            //如果是目录，创建目录
            if (nextEntry.isDirectory()) {
                file.mkdir();
            } else {
                //文件则写入具体的路径中
                FileUtils.copyToFile(tis, file);
                file.setLastModified(nextEntry.getLastModifiedDate().getTime());
            }
        }
    }
}



// 7z压缩
public void _7z() throws IOException {
    try (SevenZOutputFile outputFile = new SevenZOutputFile(new File("/test.7z"))) {
        File srcFile = new File("/test");
        _7zRecursive(outputFile, srcFile, "");
    }
}
// 递归压缩目录下的文件和目录
private void _7zRecursive(SevenZOutputFile _7zFile, File srcFile, String basePath) 
	throws IOException {
    if (srcFile.isDirectory()) {
        File[] files = srcFile.listFiles();
        String nextBasePath = basePath + srcFile.getName() + "/";
        // 空目录
        if (ArrayUtils.isEmpty(files)) {
            SevenZArchiveEntry entry = _7zFile.createArchiveEntry(srcFile, nextBasePath);
            _7zFile.putArchiveEntry(entry);
            _7zFile.closeArchiveEntry();
        } else {
            for (File file : files) {
                _7zRecursive(_7zFile, file, nextBasePath);
            }
        }
    } else {
        SevenZArchiveEntry entry = _7zFile.createArchiveEntry(srcFile, basePath + srcFile.getName());
        _7zFile.putArchiveEntry(entry);
        byte[] bs = FileUtils.readFileToByteArray(srcFile);
        _7zFile.write(bs);
        _7zFile.closeArchiveEntry();
    }
}
 // 7z解压
public void un7z() throws IOException {
    String outPath = "/test";
    try (SevenZFile archive = new SevenZFile(new File("test.7z"))) {
        SevenZArchiveEntry entry;
        while ((entry = archive.getNextEntry()) != null) {
            File file = new File(outPath, entry.getName());
            if (entry.isDirectory()) {
                file.mkdirs();
            }
            if (entry.hasStream()) {
                final byte [] buf = new byte [1024];
                final ByteArrayOutputStream baos = new ByteArrayOutputStream();
                for (int len = 0; (len = archive.read(buf)) > 0;) {
                    baos.write(buf, 0, len);
                }
                FileUtils.writeByteArrayToFile(file, baos.toByteArray());
            }
        }
    }
}
```

### 3. 修改归档
```java
String tarFile = "/test.tar";
InputStream is = new FileInputStream(tarFile);
// 替换后会覆盖原test.tar，如果是windows可能会由于文件被访问而覆盖报错
OutputStream os = new FileOutputStream(tarFile);
try (TarArchiveInputStream tais = new TarArchiveInputStream(is);
     TarArchiveOutputStream taos = new TarArchiveOutputStream(os)) {
    ChangeSet changes = new ChangeSet();
    // 删除"test.tar中"的"dir/1.txt"文件
    changes.delete("dir/1.txt");
    // 删除"test.tar"中的"t"目录
    changes.delete("t");
    // 添加文件，如果已存在则替换
    File addFile = new File("/a.txt");
    ArchiveEntry addEntry = taos.createArchiveEntry(addFile, addFile.getName());
    // add可传第三个参数：true: 已存在则替换(默认值)， false: 不替换
    changes.add(addEntry, new FileInputStream(addFile));
    // 执行修改
    ChangeSetPerformer performer = new ChangeSetPerformer(changes);
    ChangeSetResults result = performer.perform(tais, taos);
}
```

### 4. 其他
```java
//动态获取流
// 使用factory动态获取归档流
ArchiveStreamFactory factory = new ArchiveStreamFactory();
String archiveName = ArchiveStreamFactory.TAR;
InputStream is = new FileInputStream("/in.tar");
OutputStream os = new FileOutputStream("/out.tar");
// 动态获取实现类，此时ais实际上是TarArchiveOutPutStream
ArchiveInputStream ais = factory.createArchiveInputStream(archiveName, is);
ArchiveOutputStream aos = factory.createArchiveOutputStream(archiveName, os);

// 使用factory动态获取压缩流
CompressorStreamFactory factory = new CompressorStreamFactory();
String compressName = CompressorStreamFactory.GZIP;
InputStream is = new FileInputStream("/in.gz");
OutputStream os = new FileOutputStream("/out.gz");
// 动态获取实现类，此时ais实际上是TarArchiveOutPutStream
CompressorInputStream cis = factory.createCompressorInputStream(compressName, is);
CompressorOutputStream cos = factory.createCompressorOutputStream(compressName, os);

//同时解压解包压缩归档于一体的文件
// 解压 解包test.tar.gz文件
String outPath = "/test";
InputStream is = new FileInputStream("/test.tar.gz");
// 先解压，所以需要先用gzip流包装文件流
CompressorInputStream gis = new GzipCompressorInputStream(is);
// 在解包，用tar流包装gzip流
try (ArchiveInputStream tgis = new TarArchiveInputStream(gis)) {
    ArchiveEntry nextEntry;
    while ((nextEntry = tgis.getNextEntry()) != null) {
        String name = nextEntry.getName();
        File file = new File(outPath, name);
        // 如果是目录，创建目录
        if (nextEntry.isDirectory()) {
            file.mkdir();
        } else {
            // 文件则写入具体的路径中
            FileUtils.copyToFile(tgis, file);
            file.setLastModified(nextEntry.getLastModifiedDate().getTime());
        }
    }
}
```

## 5. Exec
```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-exec</artifactId>
    <version>1.3</version>
</dependency>
```

### 1. 基本调用
```java
//JDK原生写法
//不使用工具类的写法
Process process = Runtime.getRuntime().exec("cmd /c ping 192.168.1.10");
int exitCode = process.waitFor(); // 阻塞等待完成
if (exitCode == 0) { // 状态码0表示执行成功
    String result = IOUtils.toString(process.getInputStream());
    System.out.println(result);
} else {
    String errMsg = IOUtils.toString(process.getErrorStream());
    System.out.println(errMsg);
}

//process.waitFor会引起线程阻塞
//且如果命令行执行成功后的返回结果将缓冲区填满，无法写入数据则线程也会阻塞
//对外则表现为线程阻塞，命令不占资源也无反应
final Process process = Runtime.getRuntime().exec("cmd /c ping 192.168.1.10");
new Thread(() -> {
    try (BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream()))) {
        String line;
        while ((line = br.readLine()) != null) {
            try {
                process.exitValue();
                break; // exitValue没有异常表示进程执行完成，退出循环
            } catch (IllegalThreadStateException e) {
                // 异常代表进程没有执行完
            }
            //此处只做输出，对结果有其他需求可以在主线程使用其他容器收集此输出
            System.out.println(line);
        }
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}).start();
process.waitFor();
```

```java
//commons-exec则不需要考虑操作系统的平台（jdk Runtime.getRuntime().exec存在命令前缀，不同的操作系统有不同的命令前缀）
//且commons-exec能自定义输出流接受命令执行结果，能将输出流输入至字符串流、文件流、网络流等
String command = "ping 192.168.1.10";
//接收正常结果流
ByteArrayOutputStream susStream = new ByteArrayOutputStream();
//接收异常结果流
ByteArrayOutputStream errStream = new ByteArrayOutputStream();
//转换
CommandLine commandLine = CommandLine.parse(command);
DefaultExecutor exec = new DefaultExecutor();
PumpStreamHandler streamHandler = new PumpStreamHandler(susStream, errStream);
exec.setStreamHandler(streamHandler);
int code = exec.execute(commandLine);
System.out.println("result code: " + code);
// 不同操作系统注意编码，否则结果乱码
String suc = susStream.toString("GBK");
String err = errStream.toString("GBK");
System.out.println(suc);
System.out.println(err);
```

### 2. 异步调用
```java
//JDK原生命令行异步调用写法
public class RuntimeAsyncDemo {

    public static void main(String[] args) throws Exception {
        System.out.println("1. 开始执行");
        String cmd = "cmd /c ping 192.168.1.11"; // 假设是一个耗时的操作
        execAsync(cmd, processResult -> {
            System.out.println("3. 异步执行完成，success=" + processResult.success + "; msg=" + processResult.result);
            System.exit(0);
        });
        // 做其他操作 ... ...
        System.out.println("2. 做其他操作");
        // 避免主线程退出导致程序退出
        Thread.currentThread().join();
    }
    private static void execAsync(String command, Consumer<ProcessResult> callback) throws IOException {
        final Process process = Runtime.getRuntime().exec(command);
        new Thread(() -> {
            StringBuilder successMsg = new StringBuilder();
            try (BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream(), "GBK"))) {
                // 存放临时结果
                String line;
                while ((line = br.readLine()) != null) {
                    try {
                        successMsg.append(line).append("\r\n");
                        int exitCode = process.exitValue();
                        ProcessResult pr = new ProcessResult();
                        if (exitCode == 0) {
                            pr.success = true;
                            pr.result = successMsg.toString();
                        } else {
                            pr.success = false;
                            pr.result = IOUtils.toString(process.getErrorStream());
                        }
                        callback.accept(pr); // 回调主线程注册的函数
                        break; // exitValue没有异常表示进程执行完成，退出循环
                    } catch (IllegalThreadStateException e) {
                        // 异常代表进程没有执行完
                    }
                    try {
                        // 等待100毫秒在检查是否完成
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }).start();
    }

    private static class ProcessResult {
        boolean success;
        String result;
    }
}
```

```java
//commons异步调用写法，commons-exec原生支持异步调用
public void execAsync() throws IOException, InterruptedException {
    String command = "ping 192.168.1.10";
    //接收正常结果流
    ByteArrayOutputStream susStream = new ByteArrayOutputStream();
    //接收异常结果流
    ByteArrayOutputStream errStream = new ByteArrayOutputStream();
    CommandLine commandLine = CommandLine.parse(command);
    DefaultExecutor exec = new DefaultExecutor();

    PumpStreamHandler streamHandler = new PumpStreamHandler(susStream, errStream);
    exec.setStreamHandler(streamHandler);
    ExecuteResultHandler erh = new ExecuteResultHandler() {
        @Override
        public void onProcessComplete(int exitValue) {
            try {
                String suc = susStream.toString("GBK");
                System.out.println(suc);
                System.out.println("3. 异步执行完成");
            } catch (UnsupportedEncodingException uee) {
                uee.printStackTrace();
            }
        }
        @Override
        public void onProcessFailed(ExecuteException e) {
            try {
                String err = errStream.toString("GBK");
                System.out.println(err);
                System.out.println("3. 异步执行出错");
            } catch (UnsupportedEncodingException uee) {
                uee.printStackTrace();
            }
        }
    };
    System.out.println("1. 开始执行");
    exec.execute(commandLine, erh);
    System.out.println("2. 做其他操作");
    // 避免主线程退出导致程序退出
    Thread.currentThread().join();
}
```

### 3. 看门狗
```java
String command = "ping 192.168.1.10";
ByteArrayOutputStream susStream = new ByteArrayOutputStream();
ByteArrayOutputStream errStream = new ByteArrayOutputStream();
CommandLine commandLine = CommandLine.parse(command);
DefaultExecutor exec = new DefaultExecutor();
//设置一分钟超时
ExecuteWatchdog watchdog = new ExecuteWatchdog(60*1000);
exec.setWatchdog(watchdog);
PumpStreamHandler streamHandler = new PumpStreamHandler(susStream, errStream);
exec.setStreamHandler(streamHandler);
try {
    int code = exec.execute(commandLine);
    System.out.println("result code: " + code);
    // 不同操作系统注意编码，否则结果乱码
    String suc = susStream.toString("GBK");
    String err = errStream.toString("GBK");
    System.out.println(suc+err);
} catch (ExecuteException e) {
    if (watchdog.killedProcess()) {
        // 被watchdog故意杀死
        System.err.println("超时了");
    }
}
//ExecuteWatchdog可以通过destroyProcess()删除监控的进程
//可以通过killedProcess()查看监控的进程是否被杀死
```

## 6. Net
```xml
<dependency>
    <groupId>commons-net</groupId>
    <artifactId>commons-net</artifactId>
    <version>3.9.0</version>
</dependency>
```

### 1. commons-net支持协议
**FTP/FTPS**：FTP（File Transfer Protocol，文件传输协议）是 TCP/IP 协议组中的协议之一。FTP协议包括两个组成部分，其一为FTP服务器，其二为FTP客户端。其中FTP服务器用来存储文件，用户可以使用FTP客户端通过FTP协议访问位于FTP服务器上的资源。在开发网站的时候，通常利用FTP协议把网页或程序传到Web服务器上。此外，由于FTP传输效率非常高，在网络上传输大的文件时，一般也采用该协议。实现该协议的类是**FTPClient/FTPSClient**

**FTP over HTTP** (experimental)：以http协议实现FTP的功能。由于FTP工作在被动模式时不仅需要将21作为FTP的控制（命令）端口，还要将20作为FTP的数据端口，因此在配置防火墙时比较麻烦，不如用http协议传输文件。因此可以利用原有的网站结合Alias的方法加目录访问控制来实现。实现该协议的类是**FTPHTTPClient**

**NNTP**：网络新闻组传输协议（Network News Transfer Protocol）是一个主要用于阅读和张贴新闻文章到Usenet上的Internet应用协议，也负责新闻在服务器间的传送。实现该协议的类是**NNTPClient**

**SMTP(S)**：SMTP（Simple Mail Transfer Protocol）是一种提供可靠且有效的电子邮件传输的协议。SMTP是建立在FTP文件传输服务上的一种邮件服务，主要用于系统之间的邮件信息传递，并提供有关来信的通知。实现该协议的类是**SMTPClient/SMTPSClient**

**POP3(S)**：POP3，全名为“Post Office Protocol - Version 3”，即“邮局协议版本3”。POP3协议允许电子邮件客户端下载服务器上的邮件，但是在客户端的操作（如移动邮件、标记已读等），不会反馈到服务器上。实现该协议的类是**POP3Client/POP3SClient**

**IMAP(S)**：IMAP（Internet Message Access Protocol）以前称作交互邮件访问协议（Interactive Mail Access Protocol），是一个应用层协议。它的主要作用是邮件客户端可以通过这种协议从邮件服务器上获取邮件的信息，下载邮件等。IMAP提供webmail 与电子邮件客户端之间的双向通信，客户端的操作都会反馈到服务器上，对邮件进行的操作，服务器上的邮件也会做相应的动作。实现该协议的类是**IMAPClient/IMAPSClient**

**Telnet**：Telnet协议是TCP/IP协议族中的一员，是Internet远程登录服务的标准协议和主要方式。它为用户提供了在本地计算机上完成远程主机工作的能力。在终端使用者的电脑上使用telnet程序，用它连接到服务器。终端使用者可以在telnet程序中输入命令，这些命令会在服务器上运行，就像直接在服务器的控制台上输入一样。Telnet是常用的远程控制Web服务器的方法。实现该协议的类是**TelnetClient**

**TFTP**：TFTP（Trivial File Transfer Protocol,简单文件传输协议）是TCP/IP协议族中的一个用来在客户机与服务器之间进行简单文件传输的协议，提供不复杂、开销不大的文件传输服务。端口号为69。实现该协议的类是**TFTPClient**

**Finger**：显示有关运行 Finger 服务或 Daemon 的指定远程计算机（通常是运行 UNIX 的计算机）上用户的信息。该远程计算机指定显示用户信息的格式和输出。实现该协议的类是**FingerClient**

**Whois**：whois 是用来查询域名的IP以及所有者等信息的传输协议。简单说，whois就是一个用来查询域名是否已经被注册，以及注册域名的详细信息的数据库（如域名所有人、域名注册商）。早期的whois查询多以命令行接口存在，但是现在出现了一些网页的线上查询工具，其仍依赖whois协议。命令行工具仍然被系统管理员广泛使用。whois通常使用TCP协议43端口。每个域名/IP的whois信息由对应的管理机构保存。实现该协议的类是**WhoisClient**

**rexec/rcmd/rlogin**：是一组Unix命令，远程执行，远程登录，起源于BSD系统。实现该协议的类是**RExecClient/RCommandClient/RLoginClient**

**Time** (rdate) / **Daytime**：**DAYTIME**协议是基于TCP的应用，是一种有用的调试工具，它的作用是返回当前时间和日期，格式是字符串格式。**Time**时间协议（英语：TIME protocol）是一个在RFC 868内定义的网络传输协议。它用作提供机器可读的日期时间信息。实现该协议的类是**TimeTCPClient/TimeUDPClient，DaytimeTCPClient/DaytimeUDPClient**

**Echo**：echo是一个计算机命令，它可以基于TCP协议，也可以基于UDP协议，服务器在端口7检测有无消息。是路由也是网络中最常用的数据包，可以通过发送echo包知道当前的连接节点有那些路径，并且通过往返时间能得出路径长度。实现该协议的类是**EchoTCPClient/EchoUDPClient**

**Discard**：抛弃协议，作用就是接收到什么抛弃什么，它对调试网络状态的一定的用处。实现该协议的类是**DiscardTCPClient/DiscardUDPClient**

**NTP/SNTP**：NTP服务器【Network Time Protocol（NTP）】是用来使计算机时间同步化的一种协议，它可以使计算机对其服务器或时钟源（如石英钟，GPS等等）做同步化。实现该协议的类是**NTPUDPClient**

## 7. Collections
```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

### 1. 工具类
```java
//CollectionUtils拓展
String str = null;
List list1 = Arrays.asList(new String[]{"1", "2", "3"});
List list2 = Arrays.asList(new String[]{"1", "2", "4"});
// 判断是否为空（null或空list都为true）
CollectionUtils.isEmpty(list1);
// 添加元素（忽略null元素）
CollectionUtils.addIgnoreNull(list1, str);
// list是否包含subList中的所有元素
CollectionUtils.containsAll(list1, list2); // false
// list是否包含subList中的任意一个元素
CollectionUtils.containsAny(list1, list2); // true
// list1 减去 list2
CollectionUtils.subtract(list1, list2); // ["3"]
// 合并两个list并去重
CollectionUtils.union(list1, list2); //["1", "2", "3", "4"]
// 取两个list同时存在的元素
CollectionUtils.intersection(list1, list2); // [1", "2"]


//ListUtils拓展
List list1 = Arrays.asList(new String[]{"1", "2", "3"});
List list2 = Arrays.asList(new String[]{"1", "2", "4"});
// 同CollectionUtils，求差集，返回结果为List
ListUtils.subtract(list1, list2); // ["3"]
//求并集且不去重
ListUtils.union(list1, list2); //["1", "2", "3", "4"]
//求交集
ListUtils.intersection(list1, list2); // [1", "2"]
// 判断两个集合中的内容是否完全相同（顺序也一致）
ListUtils.isEqualList(list1, list2); // false
// list1如果为null则转换为空List
ListUtils.emptyIfNull(list1);
// list1中所有元素做Hash
ListUtils.hashCodeForList(list1);
```

### 2. List拓展
```java
//FixedSizeList 用于装饰另一个 List 以阻止修改其大小。不支持添加、删除、清除等操作。set 方法是允许的（因为它不会改变列表大小）
List<String> sourceList = new ArrayList<>();
sourceList.add("1");
// 装饰一下原list
List<String> list = FixedSizeList.fixedSizeList(sourceList);
list.set(0, "11");
println(list); // [11]
// 以下改变容器size的操作会抛出异常
list.add("4"); // UnsupportedOperationException("List is fixed size")
list.remove("5"); // UnsupportedOperationException("List is fixed size")
list.clear(); // UnsupportedOperationException("List is fixed size")


//SetUniqueList 用来装饰另一个 List 以确保不存在重复元素，内部使用了 Set 来判断重复问题
List<String> sourceList = new ArrayList<>();
sourceList.add("1");
sourceList.add("2");
// 元素不重复的list
SetUniqueList<String> list = SetUniqueList.setUniqueList(sourceList);
// 存在则不处理，不会影响原来顺序
list.add("2");
println(list); // [1,2]


//TransformedList 装饰另一个 List 以转换添加的对象。add 和 set 方法受此类影响
List<String> sourceList = new ArrayList<>();
sourceList.add("1");
sourceList.add("2");
// 转换list,在添加元素的时候会通过第二个参数Transformer转换一下
// （Transformer接口只有一个抽象方法可以使用lambda表达式）
// transformingList不会对原list的已有元素做转换
TransformedList<String> list = TransformedList.transformingList(sourceList, e -> e.concat("_"));
list.add("a");
println(list); // [1, 2, a_]
// transformedList会对原list的已有元素做转换
list = TransformedList.transformedList(sourceList, e -> e.concat("_"));
list.add("a");
println(list); // [1_, 2_, a_]


//PredicatedList 装饰另一个 List ，装饰后的 List 在添加元素的时候会调用 Predicate 接口来判断元素，匹配通过才会被添加到集合中
List<String> sourceList = new ArrayList<>();
// 在添加元素的时候会通过第二个参数Predicate判断一下是否符合要求，符合要求才添加进来
PredicatedList<String> list = PredicatedList.predicatedList(new ArrayList<>(), e -> e.startsWith("_"));
list.add("_4");
println(list); // [_4]
// 以下会抛异常：java.lang.IllegalArgumentException: Cannot add Object '4'
list.add("4");


// 有序的set，按照插入顺序排序
Set<String> set = new ListOrderedSet<>();
set.add("aa");
set.add("11");
set.add("哈哈");
println(set); // [aa,11,哈哈]
```

### 3. Map拓展
```java
//MultiValuedMap 和正常的 Map 有点区别，同一个 key 允许存放多个 value，这些 value 会放到一个 List 中
//这个功能如果用 Java 的 Map 我们需要构造一个 Map<String, List<String>> 加个各种操作来实现
// list实现，允许value重复
ListValuedMap<String, String> map = new ArrayListValuedHashMap<>(); 
map.put("user", "张三");
map.put("user", "李四");
map.put("user", "张三");
map.put("age", "12");
// 注意：value的泛型是String, 但是get方法返回的是List<String>
List<String> users2 = map.get("user"); // [张三,李四,张三]
// multiMap的其他方法
map.containsKey("user"); // true
map.containsValue("张三"); // true
map.containsMapping("user", "张三"); // true
int size = map.size(); // 4
Collection<String> ss = map.values();// [张三,李四,张三,12]
map.remove("user"); // 清空user的所有value
// 转换为原生map
Map<String, Collection<String>> jMap = map.asMap();


// key大小写不敏感
Map<String, Integer> map = new CaseInsensitiveMap<>();
map.put("one", 1);
map.put("two", 2);
Integer o = map.get("ONE");
println(o); // 1


////有顺序的 Map，按照插入顺序排序。如果使用 hashMap 的话 key 会按照 hash 值排序，可能和插入顺序一样，也可能不一样
// key有序：按照插入顺序
OrderedMap<String, String> map = new ListOrderedMap<>();
map.put("哈哈", "1");
map.put("此处", "2");
map.put("cc", "3");
map.put("dd", "4");
// 得到的keySet有序
Set<String> set = map.keySet(); // 哈哈,此处,cc,dd
String nk = map.nextKey("此处"); // cc
String pk = map.previousKey("此处"); // 哈哈


//LRU（Least recently used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据
//其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”
LRUMap<String, String> map = new LRUMap<>(2);
map.put("aa", "1");
map.put("bb", "2");
map.put("cc", "3");
// 最早没有被使用的aa将被移出
println(map); // [bb:2, cc:3]
// 访问一次bb，此时在put的话将会移出最早没有被访问的cc
map.get("bb");
map.put("dd", "4");
println(map); // [bb:2, dd:4]


//装饰一个 Map 以在达到过期时间时删除过期条目
//当在 Map 中放置键值对时，此装饰器使用 ExpirationPolicy 来确定条目应保持多长时间，由到期时间值定义
//当对 Map 做操作的时候才会检查元素是否过期并触发删除操作
// 存活一秒钟
int ttlMillis = 1000;
PassiveExpiringMap.ExpirationPolicy<String, String> ep = new PassiveExpiringMap.ConstantTimeToLiveExpirationPolicy<>(ttlMillis);
PassiveExpiringMap<String, String> map = new PassiveExpiringMap<>(ep);
map.put("a", "1");
map.put("b", "2");
map.put("c", "3");
// 等待一秒后在获取
Thread.sleep(1000);
String vc = map.get("c");
println(vc); // null


//BidiMap 允许在 key 和 value 之间进行双向查找。其中一个键可以查找一个值，一个值可以同样轻松地查找一个键
// 双向map, 可通过value获取key
// value也不允许重复，如果重复将会覆盖旧值
BidiMap<String, String> map = new TreeBidiMap<>();
map.put("dog", "狗");
map.put("cat", "猫");
// value重复的话key也会被覆盖，相当于"cat2:猫"会覆盖掉"cat:猫"
// map.put("cat2", "猫");
println(map); // {cat=猫, dog=狗}
String key = map.getKey("狗");
println(key); // dog

// 反向，value变为key，key变为value
BidiMap<String, String> iMap = map.inverseBidiMap();
println(iMap); // {狗=dog, 猫=cat}
println(iMap.get("狗")); // dog

// 对反向map操作同时影响原map
iMap.put("鱼", "fish");
println(iMap); // {狗=dog, 猫=cat, 鱼=fish}
println(map); // {cat=猫, dog=狗, fish=鱼}
```

## 8. HTTPClient
```xml
<dependency>
    <groupId>commons-httpclient</groupId>
    <artifactId>commons-httpclient</artifactId>
    <version>3.1</version>
</dependency>
```

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.13</version>
</dependency>
```

```xml
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
    <version>5.2.1</version>
</dependency>
```

### 1. HttpClient3
```java
// httpClient对象是线程安全的，可以单例使用，提升性能
HttpClient httpClient = new HttpClient();
// 设置连接超时 和 socket超时
httpClient.getHttpConnectionManager().getParams().setConnectionTimeout(2000);
httpClient.getHttpConnectionManager().getParams().setSoTimeout(5000); // 响应超时
HttpMethod getM = new GetMethod("http://test.com/");
// 设置请求头
getM.setRequestHeader("Content-Type", "application/json");
NameValuePair p1 = new NameValuePair("name", "zs");
NameValuePair p2 = new NameValuePair("age", "11");
// 设置查询参数，相当于 ?name=zs&age=11
getM.setQueryString(new NameValuePair[]{p1, p2});
try {
    int code = httpClient.executeMethod(getM);
    if (code == HttpStatus.SC_OK) {
        // 获取结果字符串
        String res = getM.getResponseBodyAsString();
        // InputStream res = getM.getResponseBodyAsStream(); // 也可以转换为流
        System.out.println(res);
    } else {
        System.err.println("请求失败，状态码：" + code);
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    // 释放连接资源
    getM.releaseConnection();
}
```

### 2. HttpClient4
```java
// httpClient对象是线程安全的，可以单例使用，提升性能
CloseableHttpClient httpClient = HttpClientBuilder.create().build();
HttpGet httpGet = new HttpGet("http://test.com/?name=zs&age=11");
httpGet.setHeader("Content-Type", "application/json");
RequestConfig requestConfig = RequestConfig.custom()
        .setConnectTimeout(2000) // 连接超时
        .setConnectionRequestTimeout(2000) // 请求超时
        .setSocketTimeout(2000) // 响应超时
        .build();
httpGet.setConfig(requestConfig);
try (CloseableHttpResponse res = httpClient.execute(httpGet)) {
    int code = res.getStatusLine().getStatusCode();
    if (code == HttpStatus.SC_OK) {
        HttpEntity entity = res.getEntity();
        println(EntityUtils.toString(entity, "UTF-8"));
    } else {
        System.err.println("请求失败，状态码：" + code);
    }
} catch (IOException e) {
    System.err.println("请求异常");
} finally {
    httpGet.reset();
    // httpGet.releaseConnection(); // 等价于reset()
}
```

### 3. HttpClient5
```java
// httpClient对象是线程安全的，可以单例使用，提升性能
CloseableHttpClient httpClient = HttpClients.createDefault();
HttpPost httpPost = new HttpPost("https://test.com/");
RequestConfig requestConfig = RequestConfig.custom()
        .setConnectTimeout(Timeout.ofSeconds(2))
        .setConnectionRequestTimeout(Timeout.ofSeconds(2))
        .setResponseTimeout(Timeout.ofSeconds(2))
        .build();
httpPost.setConfig(requestConfig);
httpPost.setVersion(HttpVersion.HTTP_2); // 支持http2
httpPost.setHeader("Content-Type", "application/json");
// 支持多种entity参数，字节数组，流，文件等等
// 此处使用restful的"application/json"，所以传递json字符串
httpPost.setEntity(new StringEntity(JSON.toJSONString(paramsObj)));
try (CloseableHttpResponse res = httpClient.execute(httpPost)) {
    if (res.getCode() == HttpStatus.SC_OK) {
        HttpEntity entity = res.getEntity();
        println(EntityUtils.toString(entity));
    } else {
        System.err.println("请求失败，状态码：" + res.getCode());
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    // 释放连接资源
    httpPost.reset();
}


//重定向
CloseableHttpClient httpClient = HttpClients.custom()
        .disableAutomaticRetries() //关闭自动重试
        .setRedirectStrategy(DefaultRedirectStrategy.INSTANCE)
        .build();
// ... ...


//异步支持
CloseableHttpAsyncClient httpClient = HttpAsyncClients.createDefault();
SimpleHttpRequest request = SimpleHttpRequest.create(Method.GET.name(), "https://www.baidu.com/");
httpClient.start();
httpClient.execute(request, new FutureCallback<SimpleHttpResponse>() {
    @Override
    public void completed(SimpleHttpResponse result) {
        // 响应成功
        println(result.getBodyText());
        IOUtils.closeQuietly(httpClient);
    }

    @Override
    public void failed(Exception ex) {
        // 响应出错
        ex.printStackTrace();
        IOUtils.closeQuietly(httpClient);
    }

    @Override
    public void cancelled() {
        // 响应取消
        println("cancelled");
        IOUtils.closeQuietly(httpClient);
    }
});
// ... ... 做其他业务处理
```

## 9. Pool
```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.11.1</version>
</dependency>
```
