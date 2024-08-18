## 1. JENV
使用`JENV`配置多`JDK`环境
1. 将`JENV`根目录配置到环境变量中
2. 使用命令`jenv add <name> <path>`添加`JDK`
```sh
jenv add 1.8 "D:\Admin\JEnv\JDK\jdk1.8.0_202"
jenv add 11  "D:\Admin\JEnv\JDK\jdk-11"
jenv add 17  "D:\Admin\JEnv\JDK\jdk-17.0.10"
jenv add 20  "D:\Admin\JEnv\JDK\jdk-20"
jenv add 21  "D:\Admin\JEnv\JDK\jdk-21.0.2"
jenv add 22  "D:\Admin\JEnv\JDK\jdk-22"
```
3. `jenv list`查看添加的`JDK`
4. `JENV`所有命令
```sh
"jenv list"                         #列出所有已注册的Java-env。
"jenv add <name> <path>"            #向JEnv添加一个新的Java版本，该版本可以通过给定的名称来引用。
"jenv remove <name>"                #从JEnv中删除指定的Java版本。
"jenv change <name>"                #将给定的Java版本全局应用于所有重新启动的外壳和这个外壳。
"jenv use <name>"                   #在本地为当前的外壳应用给定的Java版本。
"jenv local <name>"                 #在此文件夹中的任何时候都将使用给定的Java版本。还将为所有子文件夹设置Java版本。
"jenv link <executable>"            #在JAVA_HOME中创建可执行文件的快捷方式。例如“javac”
"jenv uninstall <name>"             #删除JEnv并将指定的Java版本恢复到系统。您可以保留配置文件
"jenv autoscan [--yes|-y] ?<path>?" #将扫描给定路径中的Java安装并请求将它们添加到JEnv。路径是可选的，“--yes|-y”接受缺省值。
```
## 2. HashMap底层
