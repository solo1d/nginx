> 系统版本 :
>
> 树莓派4B ：Linux raspberrypi 6.1.0-rpi4-rpi-v8 #1 SMP PREEMPT Debian 1:6.1.54-1+rpt2 (2023-10-05) aarch64 GNU/Linux
>
> - os-release :
>
>   - PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
>
>   - NAME="Debian GNU/Linux"
>
>   - VERSION_ID="12"
>
>   - VERSION="12 (bookworm)"
>
>   - VERSION_CODENAME=bookworm
>
>   - ID=debian

## 目录

- [安装所需要的库](#安装所需要的库)
- [编译选项和安装目录确认](#编译选项和安装目录确认)
- [运行](#运行)
- [添加开机启动](#添加开机启动)



## 安装所需要的库

### 切换国内源

教程连接  `https://mirror.tuna.tsinghua.edu.cn/help/raspbian/` ,`https://mirror.tuna.tsinghua.edu.cn/help/debian/` , `https://mirror.tuna.tsinghua.edu.cn/help/raspberrypi/`,

```bash
#可选步骤：  切换清华源
sudo apt install apt-transport-https ca-certificates
sudo cp -a /etc/apt/sources.list.d/raspi.list /etc/apt/sources.list.d/raspi.list.back
sudo echo 'deb https://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ bookworm main' > /etc/apt/sources.list.d/raspi.list
```

```ini
将如下内容写入 /etc/apt/sources.list 文件内

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
deb-src https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
```

```bash
#更新源
sudo apt clean 
sudo apt update 
```





### 安装库

```bash
# openssl 库
sudo apt install openssl libssl-dev
# PCRE 库
sudo apt install  libpcre3-dev
# libxml2/libxslt 库
sudo apt install  libui-gxmlcpp-dev 
# GD lib 库
sudo apt install libgd-dev
# GeoIP 库
sudo apt install libgeoip-dev
# google perftools 库
sudo apt  install google-perftools libgoogle-perftools-dev
```



## 编译选项和安装目录确认

**安装目录： `${HOME}/nginx`**

**源码目录：`${HOME}/nginx-src`**

- 安装后的位置
  - 可执行程序： `${HOME}/nginx/sbin`
  - 错误日志： `${HOME}/nginx/logs`
  - pid: `${HOME}/nginx/logs/nginx.pid`
  - 程序运行时的用户和组 `pi`

```bash
# 创建目录
mkdir ${HOME}/nginx/sbin
mkdir ${HOME}/nginx/logs
mkdir ${HOME}/nginx/conf
mkdir ${HOME}/nginx/httpProxyTemp
mkdir ${HOME}/nginx/httpFastcgiTemp
mkdir ${HOME}/nginx/httpUwsgiTemp
mkdir ${HOME}/nginx/httpScgiTemp

mkdir ${HOME}/nginx-src/builddir 
```

```bash
# 编译命令
# 来到源码目录： cd  ${HOME}/nginx-src/
./configure 	--prefix=${HOME}/nginx            \
  --sbin-path=${HOME}/nginx/sbin        \
  --conf-path=${HOME}/nginx/conf/nginx.conf       \
  --error-log-path=${HOME}/nginx/logs/error.log       \
  --pid-path=${HOME}/nginx/logs/nginx.pid        \
  --user=pi       \
  --group=pi       \
  --builddir=${HOME}/nginx-src/builddir       \
  --with-pcre       \
  --with-debug       \
  --with-http_ssl_module       \
  --with-stream								\
  --with-http_realip_module       \
  --with-http_addition_module       \
  --with-http_xslt_module         \
  --with-http_image_filter_module       \
  --with-http_geoip_module       \
  --with-http_sub_module       \
  --with-http_dav_module       \
  --with-http_flv_module       \
  --with-http_gzip_static_module       \
  --http-proxy-temp-path=${HOME}/nginx/httpProxyTemp       \
  --http-fastcgi-temp-path=${HOME}/nginx/httpFastcgiTemp       \
  --http-uwsgi-temp-path=${HOME}/nginx/httpUwsgiTemp       \
  --http-scgi-temp-path=${HOME}/nginx/httpScgiTemp       \
  --with-mail       \
  --with-mail_ssl_module       \
  --with-google_perftools_module       \
  --with-cpp_test_module        
```

```bash
# 输出一下内容代表成功
Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + using system zlib library

  nginx path prefix: "/home/pi/nginx"
  nginx binary file: "/home/pi/nginx/sbin"
  nginx modules path: "/home/pi/nginx/modules"
  nginx configuration prefix: "/home/pi/nginx/conf"
  nginx configuration file: "/home/pi/nginx/conf/nginx.conf"
  nginx pid file: "/home/pi/nginx/logs/nginx.pid"
  nginx error log file: "/home/pi/nginx/logs/error.log"
  nginx http access log file: "/home/pi/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "/home/pi/nginx/httpProxyTemp"
  nginx http fastcgi temporary files: "/home/pi/nginx/httpFastcgiTemp"
  nginx http uwsgi temporary files: "/home/pi/nginx/httpUwsgiTemp"
  nginx http scgi temporary files: "/home/pi/nginx/httpScgiTemp"
```

### 编译和安装

```bash
make

make install
```



## 运行

```bash
cd  ${HOME}/nginx/sbin

./nginx

# 重载配置文件
./nginx -s reload
```



## 添加开机启动

使用 `systemctl`  来管理开机启动

```ini
# 新建文件名为  nginx.service
touch nginx.service
# 将如下内容写入到该文件 nginx.service , 假设安装目录为  /home/pi/nginx
[Unit]
Description=nginx service
After=network.target 
   
[Service] 
Type=forking 
ExecStart=/home/pi/nginx/sbin/nginx
ExecReload=/home/pi/nginx/sbin/nginx -s reload
ExecStop=/home/pi/nginx/sbin/nginx -s quit
PrivateTmp=true 
   
[Install] 
WantedBy=multi-user.target
```

```bash
#将 nginx.service 文件拷贝到如下目录
sudo cp  nginx.service /lib/systemd/system
# 修复权限
sudo chmod 644 /lib/systemd/system/nginx.service
```

```bash
sudo systemctl start nginx.service			# 启动
sudo systemctl stop  nginx.service			# 停止
sudo systemctl enable nginx.service		  # 开机启动
sudo systemctl disable nginx.service		# 关闭开机启动
```



