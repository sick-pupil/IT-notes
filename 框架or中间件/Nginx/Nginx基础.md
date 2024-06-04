## 1. 安装配置
**Linux安装步骤**
```shell
cd /opt # 进入根目录下的 opt 目录

wget http://nginx.org/download/nginx-1.16.1.tar.gz # 下载tar包

tar -zxvf nginx-1.16.1.tar.gz -C /usr/local/java # 解压到 /usr/local/java 目录下（java目录需要自行创建） 解压完成后，你会在 /usr/lcoal/java 目录下会多出一个目录 nginx-1.16.1

cd /usr/local/java/nginx-1.16.1 # 进入 nginx-1.16.1 目录

./configure # 执行 ./configure 命令

make && make install # 编译并安装

# 编译安装完后，在 /usr/local/ 目录下会自动生成一个 nginx 目录，代表安装成功

cd /usr/local/nginx/sbin/ # 进入 sbin 目录
./nginx # 启动 Nginx
```

**Windows安装步骤**
下载压缩包后直接解压，即可使用

**常用命令**
```shell
cd /usr/local/nginx/sbin # 首先进入 sbin 目录

./nginx # 启动 Nginx 
./nginx -s stop # 停止 Nginx 
./nginx -s reload # 重新加载 Nginx 
./nginx -v # 查看 Nginx 版本

nginx -t             # 检查配置文件是否有语法错误
nginx -s reload       # 热加载，重新加载配置文件
nginx -s stop         # 快速关闭
nginx -s quit         # 等待工作进程处理完成后关闭
```

## 2. 目录结构
```shell
├── client_body_temp                 # POST 大文件暂存目录
├── conf                             # Nginx所有配置文件的目录
│   ├── fastcgi.conf                 # fastcgi相关参数的配置文件
│   ├── fastcgi.conf.default         # fastcgi.conf的原始备份文件
│   ├── fastcgi_params               # fastcgi的参数文件
│   ├── fastcgi_params.default       
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types                   # 媒体类型
│   ├── mime.types.default
│   ├── nginx.conf                   #这是Nginx默认的主配置文件，日常使用和修改的文件
│   ├── nginx.conf.default
│   ├── scgi_params                  # scgi相关参数文件
│   ├── scgi_params.default
│   ├── uwsgi_params                 # uwsgi相关参数文件
│   ├── uwsgi_params.default
│   └── win-utf
├── fastcgi_temp                     # fastcgi临时数据目录
├── html                             # Nginx默认站点目录
│   ├── 50x.html                     # 错误页面优雅替代显示文件，例如出现502错误时会调用此页面
│   └── index.html                   # 默认的首页文件
├── logs                             # Nginx日志目录
│   ├── access.log                   # 访问日志文件
│   ├── error.log                    # 错误日志文件
│   └── nginx.pid                    # pid文件，Nginx进程启动后，会把所有进程的ID号写到此文件
├── proxy_temp                       # 临时目录
├── sbin                             # Nginx 可执行文件目录
│   └── nginx                        # Nginx 二进制可执行程序
├── scgi_temp                        # 临时目录
└── uwsgi_temp                       # 临时目录
```

## 3. 配置文件
### 1. 全局配置
```shell
#user nobody; 					#运行用户，若编译时未指定则默认为 nobody
worker_processes 1; 			#工作进程数量，可配置成服务器内核数 * 2
#error_log logs/error.log; 		#错误日志文件的位置
#pid logs/nginx.pid; 			#PID 文件的位置
```

### 2. IO事件配置
```shell
events {
    use epoll; 					#使用 epoll 模型，2.6及以上版本的系统内核，建议使用epoll模型以提高性能
    worker_connections 4096; 	#每个进程处理 4096 个连接
}
#如提高每个进程的连接数还需执行“ulimit -n 65535”命令临时修改本地每个进程可以同时打开的最大文件数。
#在Linux平台上，在进行高并发TCP连接处理时，最高的并发数量都要受到系统对用户单一进程同时可打开文件数量的限制(这是因为系统为每个TCP连接都要创建一个socket句柄，每个socket句柄同时也是一个文件句柄)。
#可使用ulimit -a命令查看系统允许当前用户进程打开的文件数限制.
```

### 3. HTTP配置
```shell
http {
	##文件扩展名与文件类型映射表
    include       mime.types;

	##默认文件类型
    default_type  application/octet-stream;

	##日志格式设定
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

	##访问日志位置
    #access_log  logs/access.log  main;

	##支持文件发送(下载)
    sendfile        on;
 	##此选项允许或禁止使用socke的TCP_CORK的选项（发送数据包前先缓存数据），此选项仅在使用sendfile的时候使用
    #tcp_nopush     on;

	##连接保持超时时间，单位是秒
    #keepalive_timeout  0;
    keepalive_timeout  65;

	##gzip模块设置，设置是否开启gzip压缩输出
    #gzip  on;

	##Web 服务的监听配置
	server {
		##监听地址及端口
		listen 80; 
		##站点域名，可以有多个，用空格隔开
		server_name www.lic.com;
	
		##网页的默认字符集
		charset utf-8;
	
		##根目录配置
		location / {
		
			##网站根目录的位置/usr/local/nginx/html
			root html;
		
			##默认首页文件名
			index index.html index.htm;
		}
	
		##内部错误的反馈页面
		error_page 500 502 503 504 /50x.html;
		##错误页面配置
		location = /50x.html {
			root html;
		}
	}
}
```

1. **`location`匹配规则**：
- `location`常见配置指令，`root`、`alias`、`proxy_pass`
- `root`（根路径配置）：请求`www.lic.com/test`，会返回文件`/usr/local/nginx/html/test/index.html`
- `alias`（别名配置）：请求`www.lic.com/test`，会返回文件`/usr/local/nginx/html/index.html`

```shell
location /i/ {
    root /data/w3;
}
# /i/top.gif`请求会返回`/data/w3/i/top.gif

location /i/ {
    alias /data/w3/images/;
}
# /i/top.gif`请求，返回`/data/w3/images/top.gif
```

2. **日志格式**
- `$remote_addr`与`$http_x_forwarded_for`用以记录客户端的ip地址
- `$remote_user`：用来记录客户端用户名称
- `$time_local`： 用来记录访问时间与时区
- `$request`： 用来记录请求的url与http协议
- `$status`： 用来记录请求状态；成功是200
- `$body_bytes_sent`：记录发送给客户端文件主体内容大小
- `$http_referer`：用来记录从那个页面链接访问过来的
- `$http_user_agent`：记录客户浏览器的相关信息

*通常web服务器放在反向代理的后面，这样就不能获取到客户的IP地址了，通过`$remote_addr`拿到的`IP`地址是反向代理服务器的IP地址。反向代理服务器在转发请求的http头信息中，可以增加`x_forwarded_for`信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址*

3. 基于域名的虚拟主机
```shell
http {
......
	server {
		listen 80;
		# server_name可以使用通配符或者正则表达式以此匹配多级域名
		server_name www.lic.com;					#设置域名www.lic.com
		charset utf-8;
		access_log logs/www.lic.access.log; 
		location / {
			root /var/www/html/lic;					#设置www.lic.com 的工作目录
			index index.html index.php;
		}
		error_page 500 502 503 504 /50x.html;
		location = 50x.html{
			root html;
		}
	}
	
	server {
		listen 80;
		server_name www.accp.com;					#设置域名www.accp.com
		charset utf-8;
		access_log logs/www.accp.access.log; 
		location / {
			root /var/www/html/accp;
			index index.html index.php;
		}
		error_page 500 502 503 504 /50x.html;
		location = 50x.html{
			root html;
		}
	}	
}
```

4. 基于IP的虚拟主机
```shell
listen 192.168.184.30:80; #设置监听地址
server_name www.lic.com;

listen 192.168.184.31:80; #设置监听地址
server_name www.accp.com;
```

5. 基于端口的虚拟主机
```shell
listen 192.168.184.30:8080; #设置监听 8080 端口
server_name www.lic.com;

listen 192.168.184.30:8888; #设置监听 8888 端口
server_name www.accp.com;
```

## 4. 反向代理
```shell
server {
    listen       80;
    server_name  www.zhengqing520.com;# 服务器地址或绑定域名
 
    location ^~ /api {  # ^~/api 表示匹配前缀为api的请求
        proxy_pass http://www.zhengqing520.com:9528/api/;
        # 注：proxy_pass的结尾有/， -> 效果：会在请求时将/api/*后面的路径直接拼接到后面
  
        # proxy_set_header作用：设置发送到后端服务器(上面proxy_pass)的请求头值  
            # 【当Host设置为 $http_host 时，则不改变请求头的值;
            #   当Host设置为 $proxy_host 时，则会重新设置请求头中的Host信息;
            #   当为$host变量时，它的值在请求包含Host请求头时为Host字段的值，在请求未携带Host请求头时为虚拟主机的主域名;
            #   当为$host:$proxy_port时，即携带端口发送 ex: $host:8080 】
        proxy_set_header Host $host; 
  
        proxy_set_header X-Real-IP $remote_addr;
        # 在web服务器端获得用户的真实ip 需配置条件 【 $remote_addr值 = 用户ip 】
        
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # 在web服务器端获得用户的真实ip 需配置条件
        
        proxy_set_header REMOTE-HOST $remote_addr;
        # proxy_set_header X-Forwarded-For $http_x_forwarded_for;
        # $http_x_forwarded_for变量 = X-Forwarded-For变量
    }

    location ^~ /blog/ { # ^~/blog/ 表示匹配前缀为blog/后的请求
        proxy_pass  http://zhengqingya.gitee.io/blog/; 
  
        proxy_set_header Host $proxy_host; # 改变请求头值 -> 转发到码云才会成功
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-NginX-Proxy true;
    }
}

```

**`location`修饰符**：
1. `=`使用精确匹配并且终止搜索。即一旦匹配上使用=修饰的`location`，下面的其它`location`配置就无效了
2. `~`区分大小写的正则表达式匹配
3. `~*`不区分大小写的正则表达式匹配
4. `^~`如果该`location`是最佳的匹配，那么对于匹配这个`location`的字符串不再进行正则表达式检测

**`proxy_pass url`是否带斜杠场景**：
客户端请求`URL`：`https://172.16.1.1/hello/world.html`
1. `url`后面加斜杠
```shell
location /hello/ {
	proxy_pass http://127.0.0.1/;
}
```
**代理到`URL`：`http://127.0.0.1/world.html`**

2. `url`后面没有斜杠
```shell
location /hello/ {
	proxy_pass http://127.0.0.1;
}
```
**代理到`URL`：`http://127.0.0.1/hello/world.html`**

3. `url`后面存在其他路由，但最好还是添加了斜杠
```shell
location /hello/ {
	proxy_pass http://127.0.0.1/test/;
}
```
**代理到`URL`：`http://127.0.0.1/test/world.html`**

4. `url`后面存在其他路由，但最好没有添加了斜杠
```shell
location /hello/ {
	proxy_pass http://127.0.0.1/test;
}
```
**代理到`URL`：`http://127.0.0.1/testworld.html`**

## 5. 负载均衡
**实际为反向代理添加负载均衡功能**
```shell
	# 负载均衡配置
	upstream server_list {
	   # 这个是tomcat的访问路径
	   server localhost:8080;
	   server localhost:9999;
	}
	
	server {
		listen       80;
		server_name  localhost;

		location / {
			root   html;
			# 反向代理配置
			proxy_pass http://server_list;
			index  index.html index.htm;
		}

		error_page   500 502 503 504  /50x.html;
		location = /50x.html {
			root   html;
		}
	}
```

1. **为负载均衡设置权重，指定轮询几率，`weight`和访问比率成正比**
```shell
# 负载均衡配置
upstream server_list{
	# 这个是tomcat的访问路径
	server localhost:8080 weight=5;
	server localhost:9999 weight=1;
}
```

2. **为负载均衡设置`ip_hash`，每个请求按访问`ip`的`hash`值分配，这样每个访问客户端会固定访问一个后端服务器，可以解决会话`Session`丢失的问题**
```shell
upstream backserver { 
	ip_hash;
	server 127.0.0.1:8080;
	server 127.0.0.1:9090;
}
```

3. **为负载均衡设置最少连接，`web`请求会被转发到连接数最少的服务器上**
```shell
upstream backserver { 
	least_conn;
	server 127.0.0.1:8080; 
	server 127.0.0.1:9090; 
}
```

4. **不设置则为平均轮询**

## 6. 动静分离
`Nginx`动静分离，简单来说，就是动态请求和静态请求分开，也可以理解成使用`Nginx`处理静态页面，`Tomcat`处理动态页面，实际即为使用`location-root`配置静态资源（前端页面资源）访问，使用`location-proxy_pass`配置动态资源（后端接口资源）访问

*也可以使用`alias`配置静态资源访问*
```shell
location /css {
    alias /usr/local/nginx/static/css;
    index index.html index.htm;
}

location /img {
    alias /usr/local/nginx/static/img;
}
```