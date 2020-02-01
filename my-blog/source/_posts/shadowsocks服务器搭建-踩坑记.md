---
title: shadowsocks服务器搭建-踩坑记
date: 2020-01-27 10:48:24
tags:
  - 踩坑
---

记录一下为了**科学上网**折腾了一上午的2020

- 阿里云新加坡服务器，购买！
- 服务端docker安装部署，成功！ [docker部署博文](https://blog.csdn.net/weixin_42890981/article/details/84134676)
- shadowSocks配置，成功！ [shadowSocks配置博文](https://www.kancloud.cn/kancloud/a-programmer-prepares/78264)
- 怎么不好使！(chrome浏览器报错)

```
500 Internal Privoxy Error
```

- 众里寻她千百度，解决！ [阿里云ss搭建报错](https://blog.csdn.net/weixin_30655569/article/details/99208840)

### 2020.2.1 坑

> 用了五天的小飞机突然上不了guge了～

排查原因： **端口封禁**（🍅🧱需谨慎啊！），[参考博文](https://www.flyzy2005.com/fan-qiang/tcp-blocked/)
```linux
ssh -v -p [端口号] 用户名@服务器地址
```

```docker
docker run -d -p 1984:1984 oddrationale/docker-shadowsocks -s 0.0.0.0 -p [端口号] -k [密码] -m aes-256-cfb
```