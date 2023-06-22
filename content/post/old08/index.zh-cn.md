---
title: "在服务器上部署Jypyter并且为其套上https以便ios设备使用的方法"
slug: ""
description: ""
date: 2023-06-23T03:05:52+08:00
lastmod: 2023-06-23T03:05:52+08:00
draft: false
toc: true
weight: false
image: ""
categories: ["old"]
tags: ["old"]
---
萌萌开始给学弟学妹讲python了,而我还是个辣鸡,所以只能给萌萌填一点坑.就比如ios设备强制https访问导致的无法正常使用jypyter的问题.

在服务器上部署jupyter
推荐由annaconda安装. 先安装annaconda,之后
```
conda install jupyter
```

生成Jupyter的配置文件
在启动Jupyter的时候需要配置文件启动，所以首先要生成配置文件。使用命令
```
jupyter notebook --generate-config
```

自动生成配置文件，配置文件的目录应该是~/.jupyter。

生成密码
```
# 在python中运行
from notebook.auth import passwd
passwd()
```

生成ssl证书
```
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mykey.key -out mycert.pem
```

修改jupyter的配置文件
```
# 密码
c.NotebookApp.password = u'sha1:67c9e60bb8b6:9ffede0825894254b2e042ea597d771089e11aed'
# ssl证书绝对位置
c.NotebookApp.certfile = u'/absolute/path/to/your/certificate/mycert.pem'
c.NotebookApp.keyfile = u'/absolute/path/to/your/certificate/mykey.key'
# 其他设置
c.NotebookApp.ip = '*'
c.NotebookApp.allow_origin = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.port = 9999
```

运行
这个时候用jupyter命令应该就能运行了,并且通过https://ip:端口 的形式应该就能访问.

通过 nohup 命令应该就能够后台持久化运行.

但是现在依然是不安全的ssl证书所以无法使用ios设备访问.

添加https的步骤
其实很简单

1. 为服务器绑定域名
2. 给服务器加上nginx反向代理(443端口,带上刚才生成的ssl证书)
3. 使用cloud flare管理dns(添加cdn,https,防ddos等功能)

----------
给我来瓶冰阔落？

![Code](alipay.jpg)