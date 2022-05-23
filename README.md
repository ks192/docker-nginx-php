# 以下是部署流程也可直接下载然后docker-compose up -d

---

# Docker compose 部署 nginx+php

## 拉取Docker镜像

```bash
docker pull nginx:1.21.6
docker pull php:7.4.28-fpm
```

## 创建docker-compose 目录

在home目录下创建docker-vps

```bash
mkdir /home/docker-vps  #创建目录
cd /home/docker-vps     #进入目录
```

## 拷贝配置文件到宿主机

### #拷贝 nginx的配置文

```bash
docker run -d --name nginx nginx
docker cp nginx:/etc/nginx ./
docker cp nginx:/var/log ./
docker rm -f nginx
```

### 拷贝 php的配置文件

```bash
docker run -d --name php php:7.4.28-fpm
docker cp php:/usr/local/etc/php ./
docker rm -f php
```

## 更改配置文件

> 配置内容参见：https://blog.csdn.net/qq_43534481/article/details/124916254?spm=1001.2014.3001.5501
>

### 配置php.ini

```bash
cd /home/docker-vps/php #进入php目录
mv php.ini-development php.ini  #重命名php.ini-development为php.ini
sed -i "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g" php.ini #替换字符串;cgi.fix_pathinfo=1 为 cgi.fix_pathinfo=0
```

### 配置default.conf

```bash
cd /home/docker-vps/nginx/conf.d  #进入目录
在default.conf的server块底部添加如下代码
    location ~ \.php$ {
        root           html;
        fastcgi_pass   php:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /home/www$fastcgi_script_name;
        include        fastcgi_params;
    }
```

示例如下注意缩进

![在这里插入图片描述](https://img-blog.csdnimg.cn/cb3745a3b4af4302a795f5886027e8ba.png#pic_center)


## 创建web根目录

```bash
mkdir /home/docker-vps/www  #创建目录
echo  '<script>alert('installation complete')</script>' > index.html  #创建index.html文件并写入内容
echo  '<?php phpinfo();?>' > index.php				      #创建index.php文件并写入内容
```


## 编写yaml文件


在 /home/docker-nginx目录下创建一个docker-compose.yaml 文件

```yaml
touch /home/docker-vps/docker-compose.yaml  #创建文件
```

将以下内容写入docker-compose.yaml

```yaml
version: "3"
services:
    nginx:
        image: nginx:1.21.6
        container_name: "vps-nginx"
        restart: always
        ports:
            - "80:80"
            - "443:443"
        environment:
           - TZ=Asia/Shanghai
        depends_on:
           - "php"
        volumes:
           - "/home/docker-vps/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf"
           - "/home/docker-vps/log:/var/log"
           - "/home/docker-vps/www:/home/www"
        networks:
           - net-app
    php:
        image: php:7.4.28-fpm
        container_name: "vps-php"
        restart: always
        ports:
            - "9000:9000"
        environment:
            - TZ=Asia/Shanghai
        volumes:
            - "/home/docker-vps/www:/home/www"
            - "/home/docker-vps/php:/usr/local/etc/php"
        networks:
           - net-app
networks:
    net-app: 
```

## 环境上线

```bash
docker-compose up -d
```

## 验证结果

访问host首页弹窗访问host/index.php显示phpinfo界面即为成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/075d2870c4764d93ba5cdabd4b02cdea.png#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/565f89ab9f7c4c238d4541ab10db0938.png#pic_center)