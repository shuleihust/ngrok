## 前言

进行微信公众号的开发时，需要有公网IP的服务器才行，这样每次都得在本地开发再把代码提交到服务器，或直接在服务器上开发，不是很方便。
并且有时可能需要`把本机开发的网站等web项目给其他人演示`，以前都是上传到VPS上，也不是很方便。

通过google查找到`ngrok`这个东西，可以实现在本地开发即时调试，可以非常方便地实现内网穿透。

> ngrok 是一个反向代理，通过在公共的端点和本地运行的 Web 服务器之间建立一个安全的通道。ngrok 可捕获和分析所有通道上的流量，便于后期分析和重放。
> 详细介绍可以看百度百科的介绍：[ngrok介绍](http://baike.baidu.com/view/13085941.htm)。

国内有些人也贡献了自己的服务器，如[http://ngrok.cc/](http://ngrok.cc/)，如果不想自己搭建`ngrok`环境，可以直接去使用。

`ngrok`的[v1.x版本](https://github.com/inconshreveable/ngrok)是开源的，2.0就不是开源，而且两者的命令有些不同。

这里使用其1.x的开源代码进行布署，使用的编译环境为ubuntu 16.04。

## 编译安装

**以下教程中所有的example.com请替换为自己的域名**

```
#安装go环境
apt-get install golang
go version
#安装git
apt-get install git
git version
#安装mercurial包 （分布式版本控制系统）
apt-get install mercurial
hg version
#安装gcc
install gcc
#安装ngrok
cd /usr/local
git clone https://github.com/inconshreveable/ngrok.git ngrok
cd ngrok
#生成证书
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=example.com" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=example.com" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
#替换证书
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp device.crt assets/server/tls/snakeoil.crt
cp device.key assets/server/tls/snakeoil.key

#编译linux版本
GOOS=linux GOARCH=amd64 make release-server release-client
GOOS=linux GOARCH=386 make release-server release-client
#编译windows版本
GOOS=windows GOARCH=amd64 make release-server release-client
GOOS=windows GOARCH=386 make release-server release-client
#编译mac版本
GOOS=darwin GOARCH=amd64 make release-server release-client
GOOS=darwin GOARCH=386 make release-server release-client
```

## 使用方法

#### 服务端

```
#linux服务端后台启动方式
nohup ngrokd -domain="example.com" -httpAddr=":80" -httpsAddr=":443"   > /dev/null 2>&1 &
#windows服务端启动方式
ngrokd -domain="example.com" -httpAddr=":80" -httpsAddr=":443"
```

#### 客户端

ngrok.cfg文件内容

```
server_addr: "example.com:4443"
trust_host_root_certs: false
tunnels:
  test:
    subdomain: "test"
    proto:
      http: "8080"
  ssh:
    remote_port: 2201
    proto:
      tcp: "22"
```

将客户端程序拷贝到客户端电脑

```
#启动一个客户端
ngrok -config=ngrok.cfg start test
#另一种启动方式
ngrok -config=ngrok.cfg -subdomain test 8080
#同时启动多个客户端
ngrok -config=ngrok.cfg start test ssh
```