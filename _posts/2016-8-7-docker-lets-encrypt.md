---
layout: post
title: 使用Docker无痛启用Let's Encrypt
---
最近搭了个服务器，部署了好几个服务，通过Docker容器[nginx-proxy](https://github.com/jwilder/nginx-proxy)自动实现了站点配置，非常舒服，就差最麻烦的 ssl 配置了。
早就听说了免费的[Let’s Encrypt](https://letsencrypt.org)，之前也看过几篇教程，看起来还是挺复杂的，用了Docker之后变得不想在操作系统上直接安装软件了（好吧，拖延症），今天正好有空想起这件事，上网搜了下，果然有人搞定了用Docker实现Let's Encrypt的配置，哈哈，那就上吧！

[Let’s Encrypt](https://letsencrypt.org) 就不多介绍了，自己看吧，[letsencrypt-nginx-proxy-companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion) 是用来自动创建，续约Let's Encrypt证书的Nginx伴侣，用到另外一个Nginx伴侣 [docker-gen](jwilder/docker-gen)，和我目前的架构正好一致，只是再起个Container，然后启动其他应用的Container的时候加些参数

附上 docker-compose.xml

    version: '2'
    services:
      nginx:
        image: nginx
        network_mode: bridge
        ports:
          - "80:80"
          - "443:443"
        container_name: nginx
        volumes:
          - /etc/nginx/vhost.d
          - /usr/share/nginx/html
          - /tmp/nginx:/etc/nginx/conf.d
          - ./certs:/etc/nginx/certs:ro
      docker-gen:
        image: jwilder/docker-gen
        network_mode: bridge
        container_name: docker-gen
        depends_on:
          - nginx
        volumes:
          - /var/run/docker.sock:/tmp/docker.sock:ro
          - .:/etc/docker-gen/templates
        volumes_from:
          - nginx
        command: -notify-sighup nginx -watch /etc/docker-gen/templates/nginx.tmpl /etc/nginx/conf.d/default.conf
      nginx-letsencrypt:
        image: jrcs/letsencrypt-nginx-proxy-companion
        network_mode: bridge
        volumes:
          - ./certs:/etc/nginx/certs:rw
          - /var/run/docker.sock:/var/run/docker.sock:ro
        volumes_from:
          - nginx
        environment:
          - NGINX_DOCKER_GEN_CONTAINER=docker-gen

应用 docker-compose 片段

    ...
    client:
      image: nginx:alpine
      container_name: client
      network_mode: bridge
      ports:
        - "127.0.0.1:8080:80"
      environment:
        - VIRTUAL_HOST=app.imag.ink,imag.ink #nginx-proxy的域名
        - LETSENCRYPT_HOST=app.imag.ink,imag.ink #证书对应的域名
        - LETSENCRYPT_EMAIL=foo@bar.com #发送证书到期的提醒邮件地址
      volumes:
        - ./client:/usr/share/nginx/html:ro
    ...

启动之后，稍等一会就可以用 https 访问了

![docker https](/images/docker-https.png)

Go Green, Yeah!

另外，好像起了这个Container之后，会自动把 http 重定向到 https 。 新加的域名有时候有问题，重启下Container就好了。

起个Container，稍微做点配置，就搞定 ssl，还免费！！还要什么收费证书服务啊？
这个例子也另外一个方面说明的Docker的灵活强大，主要归功与 [docker-gen](https://github.com/jwilder/docker-gen) 的创新，Docker还会更多想象空间。。
