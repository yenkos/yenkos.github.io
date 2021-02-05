---
title: 私有npm仓库搭建与使用
categories: ['工程化']
tags: ['工程化', 'npm']
---




> NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：



允许用户从NPM服务器下载别人编写的第三方包到本地使用。允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。


### 一、为什么需要私有npm仓库


- 安全：内部源码不能发布到npm上开源，避免业务代码公开
- 免费：在npm上发布私有内容需要收费
- 效率：统一管理积累内部开发资源，提高开发效率



### 二、技术选型


- sinopia 不考虑，好几年没更新了
- cnpmjs 太笨重了，使用的方式基于作用域，略显繁琐
- verdaccio 轻量，使用简单，基于registry的概念，侵入少 👏👏👏👌



### 三、verdaccio搭建步骤-使用docker-安装在一组开发服务器上


#### 1.下载镜像


> docker pull verdaccio/verdaccio



![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612249875142-783a8928-fd0b-45f4-8dbb-65bfa0e406eb.png#align=left&display=inline&height=218&margin=%5Bobject%20Object%5D&name=image.png&originHeight=436&originWidth=1146&size=75828&status=done&style=none&width=573#align=left&display=inline&height=436&margin=%5Bobject%20Object%5D&originHeight=436&originWidth=1146&status=done&style=none&width=1146)


#### 2.容器启动配置


在 /alidata/www 目录下新建 npm 文件夹，并且创建 docker-compose.yml 配置文件


docker-compose是用来编排容器的，主要作用是将容器配置写在配置文件中，避免每次启动都需要手动运行 docker run 一堆东西。


> cd /alidata/www
mkdir npm && touch npm/docker-compose.yml



写入以下配置


```yaml
version: '2'

services:
  verdaccio:
    image: verdaccio/verdaccio
    container_name: verdaccio
    networks:
      - node-network
    environment:
      - VERDACCIO_PORT=4873
    ports:
      - 4873:4873
    volumes:
      - ./storage:/verdaccio/storage
      - ./conf:/verdaccio/conf
      - ./plugins:/verdaccio/plugins
networks:
  node-network:
  	driver: bridge
```


主要做了这些事情


- 将 npm/storage 目录挂载到容器的 /verdaccio/storage
- 将 npm/conf 目录挂载到容器的 /verdaccio/conf
- 将 npm/plugins 目录挂载到容器的 /verdaccio/plugins
- 容器和主机端口都设置为4873



/alidata/www/npm 文件夹下的内容可以定期git提交到云效，进行备份。主要包括用户资料，包代码。


#### 3.verdaccio配置


> 配置说明文档 [链接](https://verdaccio.org/docs/en/configuration)



在 /alidata/www/npm/conf 下新建一个文件


> touch npm/conf/config.yaml



```yaml
storage: /verdaccio/storage
auth:
  htpasswd:
    file: /verdaccio/conf/htpasswd
uplinks:
  npmjs:
    url: https://registry.npm.taobao.org/
packages:
	# scope packages
  '@scope/*':
    access: $all
    publish: $authenticated
    proxy: npmjs
  '**':
  	access: $all
    proxy: npmjs
logs:
  - {type: stdout, format: pretty, level: http}
```


- htpasswd 存放的是 npm 用户及密码信息的文件。
- uplinks 配置了npm替代地址，这里配置了淘宝源。如果在当前仓库找不相应软件包，就到uplinks配置的源获取。



#### 4.启动容器


> docker-compose up -d --build



![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612253057814-c2a65135-d72f-4b18-8dce-cac3bac1b9f0.png#align=left&display=inline&height=42&margin=%5Bobject%20Object%5D&name=image.png&originHeight=84&originWidth=1024&size=16770&status=done&style=none&width=512#align=left&display=inline&height=84&margin=%5Bobject%20Object%5D&originHeight=84&originWidth=1024&status=done&style=none&width=1024)




#### 5.nginx配置


先找到一组服务器的配置文件


> ps aux|grep nginx



![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612253220343-35c6b0e8-cc13-4043-8756-ae01b6338502.png#align=left&display=inline&height=136&margin=%5Bobject%20Object%5D&name=image.png&originHeight=272&originWidth=1942&size=76266&status=done&style=none&width=971#align=left&display=inline&height=272&margin=%5Bobject%20Object%5D&originHeight=272&originWidth=1942&status=done&style=none&width=1942)


在 /rrzuji/nginx/conf/vhosts 下新建配置


```
server {
  listen 80;
  server_name npm.rruzji.net;
  location / {
    proxy_pass              http://127.0.0.1:4873/;
    proxy_set_header        Host $host;
  }
}
```


#### 6.访问服务 npm.rrzuji.net


出现以下界面说明容器已经正常运行
![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612254830779-a21173ef-1d89-4b86-b2e2-9c4c26cc9666.png#align=left&display=inline&height=1055&margin=%5Bobject%20Object%5D&name=image.png&originHeight=2110&originWidth=2698&size=252901&status=done&style=none&width=1349#align=left&display=inline&height=2110&margin=%5Bobject%20Object%5D&originHeight=2110&originWidth=2698&status=done&style=none&width=2698)


#### 7.设置访问权限


尝试新建用户，发现出现500错误
![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612317778239-5a9f0130-035f-4c52-ac1b-8fcc391ec224.png#align=left&display=inline&height=173&margin=%5Bobject%20Object%5D&name=image.png&originHeight=346&originWidth=1130&size=68809&status=done&style=none&width=565#align=left&display=inline&height=346&margin=%5Bobject%20Object%5D&originHeight=346&originWidth=1130&status=done&style=none&width=1130)
查看docker日志，提示 permission denied 显然，这是一个文件权限问题，容器无法访问宿主路径。


> docker logs --tail 20 verdaccio



![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612317827950-d3595915-5a79-45e8-be8b-18f1f9fbfcf5.png#align=left&display=inline&height=283&margin=%5Bobject%20Object%5D&name=image.png&originHeight=566&originWidth=1680&size=168906&status=done&style=none&width=840#align=left&display=inline&height=566&margin=%5Bobject%20Object%5D&originHeight=566&originWidth=1680&status=done&style=none&width=1680)
查看文档
![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612337070620-875cd6e9-9e7e-416b-9ce9-26c6ee5c78f8.png#align=left&display=inline&height=533&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1066&originWidth=1704&size=234165&status=done&style=none&width=852#align=left&display=inline&height=1066&margin=%5Bobject%20Object%5D&originHeight=1066&originWidth=1704&status=done&style=none&width=1704)


执行下面代码，10001是容器内verdaccio使用的UID，65533是GID


> sudo chown -R 10001:65533 /alidata/www/npm/conf/htpasswd
sudo chown -R 10001:65533 /alidata/www/npm/storage



测试功能是否正常
![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612338078278-c3f84e47-a04d-4d32-b356-2c6edeea1be5.png#align=left&display=inline&height=91&margin=%5Bobject%20Object%5D&name=image.png&originHeight=216&originWidth=1126&size=30429&status=done&style=none&width=475#align=left&display=inline&height=216&margin=%5Bobject%20Object%5D&originHeight=216&originWidth=1126&status=done&style=none&width=1126)


到这里安装已经结束。接下来介绍一下如何使用以及注意事项。


### 四、使用方法


#### 1.安装nrm


没有安装nrm的可以先安装nrm，nrm是用来给npm快速换源的工具。安装好之后，新增一个名为 rnpm 的源。


> npm i nrm -g
nrm add rnpm [http://npm.rrzuji.net](http://npm.rrzuji.net)



![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612339456816-23617242-a05f-4515-a713-c40fc9c5cdbd.png#align=left&display=inline&height=255&margin=%5Bobject%20Object%5D&name=image.png&originHeight=510&originWidth=946&size=61349&status=done&style=none&width=473#align=left&display=inline&height=510&margin=%5Bobject%20Object%5D&originHeight=510&originWidth=946&status=done&style=none&width=946)


[http://npm.rrzuji.net](http://npm.rrzuji.net) 相当于私有源+淘宝源的组合，以下统称rnpm。


#### 2.用户注册


![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612339601767-02e84ba6-573c-4b1f-8715-f5c6784b47b8.png#align=left&display=inline&height=92&margin=%5Bobject%20Object%5D&name=image.png&originHeight=216&originWidth=1126&size=30429&status=done&style=none&width=478#align=left&display=inline&height=216&margin=%5Bobject%20Object%5D&originHeight=216&originWidth=1126&status=done&style=none&width=1126)


#### 3.用户登录


这个登录表示用户登录到rnpm，会在 userconfig 的文件里新增一条登录信息，不会影响到旧的npm用户授权认证记录。nrm use npm 切换后，就变成了npm用户，再 nrm use rnpm，就变成了 rnpm用户。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612339834033-a0cbc0d0-123f-4264-922c-75e65dfcb221.png#align=left&display=inline&height=94&margin=%5Bobject%20Object%5D&name=image.png&originHeight=188&originWidth=960&size=31997&status=done&style=none&width=480#align=left&display=inline&height=188&margin=%5Bobject%20Object%5D&originHeight=188&originWidth=960&status=done&style=none&width=960)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612340131756-ae928722-63d9-4280-8303-3335927e5099.png#align=left&display=inline&height=254&margin=%5Bobject%20Object%5D&name=image.png&originHeight=494&originWidth=938&size=63133&status=done&style=none&width=482#align=left&display=inline&height=494&margin=%5Bobject%20Object%5D&originHeight=494&originWidth=938&status=done&style=none&width=938)


#### 4.上传软件包


登录后，就可以上传软件包了，进入到软件包目录，执行


> npm publish



注意，提交的软件包名需要为私有库的格式(@scope/*)，同时也是为了和普通的包区别开来，否则会被拦截。


![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612363425630-e76f5284-dfbf-4a3a-86c7-d05f8f22ca4d.png#align=left&display=inline&height=472&margin=%5Bobject%20Object%5D&name=image.png&originHeight=822&originWidth=1300&size=721368&status=done&style=none&width=746)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612363459712-0dd5129a-7711-418f-b165-8cd2e41bede9.png#align=left&display=inline&height=1056&margin=%5Bobject%20Object%5D&name=image.png&originHeight=2112&originWidth=2322&size=233432&status=done&style=none&width=1161)




#### 5.项目安装使用


安装私有包


> npm install @scope/browser-version-tool-html



安装私有包和公共npm仓库的包，这是一个混用的例子


> npm install @scope/browser-version-tool-html axios



![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1612348513344-6146aab7-7f85-4159-94d6-c949b3b18cc7.png#align=left&display=inline&height=129&margin=%5Bobject%20Object%5D&name=image.png&originHeight=244&originWidth=1406&size=52683&status=done&style=none&width=746#align=left&display=inline&height=244&margin=%5Bobject%20Object%5D&originHeight=244&originWidth=1406&status=done&style=none&width=1406)


#### 6.删除软件包


> npm unpublish



#### 7.更新软件包


更新版本号后，重新发布进行更新


> npm publish

