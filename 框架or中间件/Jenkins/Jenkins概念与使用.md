## 1. 背景
以往打包部署过程：
1. 自己提交代码
2. 协调项目组人员提交代码
3. 拉取代码打包
4. 将代码包上传服务器
5. 查看应用是否运行，运行则关闭当前应用
6. 使用新的代码包启动应用
7. 观察日志是否启动成功

## 2. Jenkins自动化部署原理
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\自动化部署原理.png" style="width:700px;height:350px;" />

Jenkins环境条件：
1. JDK环境
2. Git/Svn客户端
3. Maven

## 3. Jenkins安装
1. 下载安装包`Jenkins-2.346.3.war`
2. 运行安装包`java -jar jenkins.war --httpPort=8010`
3. 浏览器访问`http://localhost:8010`
4. 查看初始密码`C:\Users\Administrator\.jenkins\secrets\initialAdminPassword`
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\初始密码.png" style="width:700px;height:250px;" />
5. 选择推荐的插件安装
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\自定义插件安装.png" style="width:700px;height:400px;" />
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\安装插件.png" style="width:700px;height:300px;" />
6. 设置初始用户和密码
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\注册初始用户.png" style="width:700px;height:300px;" />
7. 完成安装
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\完成安装.png" style="width:700px;height:200px;" />

## 4. Jenkins基本配置
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\基本配置.png" style="width:700px;height:450px;" />

### 1. Configure System
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\系统设置.png" style="width:700px;height:300px;" />

### 2. Configure Global Security
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\安全配置1.png" style="width:700px;height:500px;" />

<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\安全配置2.png" style="width:700px;height:80px;" />

### 3. Global Tool Configuration
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\全局工具配置-maven1.png" style="width:700px;height:250px;" />

<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\全局工具配置-jdk.png" style="width:700px;height:200px;" />

<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\全局工具配置-git.png" style="width:700px;height:150px;" />

<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\全局工具配置-maven2.png" style="width:700px;height:250px;" />

## 5. 自动化部署
### 1. 新建工程
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\新建工程.png" style="width:700px;height:400px;" />

### 2. General
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\General.png" style="width:700px;height:400px;" />

### 3. 源码管理
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\源码管理.png" style="width:700px;height:300px;" />

<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\添加凭依.png" style="width:700px;height:400px;" />

### 4. 构建触发器
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\构建触发器.png" style="width:700px;height:200px;" />

### 5. 构建
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\构建.png" style="width:700px;height:150px;" />

### 6. 构建后操作
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\构建后操作.png" style="width:700px;height:400px;" />

`stop.sh`
```shell
#!/bin/bash
echo "Stop Procedure : demo2-0.0.1-SNAPSHOT.jar"
pid=`ps -ef |grep java|grep demo2-0.0.1-SNAPSHOT.jar|awk '{print $2}'`
echo 'old Procedure pid:'$pid
if [ -n "$pid" ]
then
kill -9 $pid
fi
```

`start.sh`
```shell
#!/bin/bash
export JAVA_HOME=/usr/java/jdk1.8.0_131
echo ${JAVA_HOME}
echo 'Start the program : demo2-0.0.1-SNAPSHOT.jar'
chmod 777 /home/ldp/app/demo2-0.0.1-SNAPSHOT.jar
echo '-------Starting-------'
cd /home/ldp/app/
nohup ${JAVA_HOME}/bin/java -jar demo2-0.0.1-SNAPSHOT.jar &
echo 'start success'
```

### 7. Git回调
<img src="D:\Project\IT-notes\框架or中间件\Jenkins\img\Git回调.png" style="width:700px;height:500px;" />
