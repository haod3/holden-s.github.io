---
layout: post
title: uWSGI下web应用获取POST请求参数的方法
---


本文介绍了使用uWSGI时，web应用获取POST请求参数的方法。

<!-- more -->

## uWSGI调用web应用

根据uWSGI的使用方法，用户需定义一个`application`，uWSGI调用这个application来处理web服务器发送的请求。例如在使用Python时，application被定义为

```py
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return ["Hello World"]
```

其中`env`包含了web服务器得到的请求信息，而这之中和参数相关的变量如下：

变量 | 含义
---|---
QUERY_STRING | HTTP请求中的查询字符串，URL中?后面的内容
CONTENT_LENGTH | 定义了request body的长度
uwsgi.input | 一个类文件的输入流，application可以通过这个获取HTTP的request body

当uWSGI处理GET请求时，直接从`QUERY_STRING`中获取即可，而POST请求并不直观的存储在env, 需要通过`wsgi.input`和`CONTENT_LENGTH`来联合获取

## 获取POST请求参数方法

```py
try:
    body_size = int(env.get('CONTENT_LENGTH', 0))
except ValueError:
    body_size = 0
    
query = env["wsgi.input"].read(body_size)
```

由于CONTENT_LENGTH可能不存在或者被忽略，因此需要用`try / catch`来处理一下，最后根据该长度获取POST请求参数。

## 例子

### 1. 编写application

```py
#!/usr/bin/python
# -*- coding: utf-8 -*-

# demo.py

def application(env, start_response):
    try:
        body_size = int(env.get('CONTENT_LENGTH', 0))
    except ValueError:
        body_size = 0

    query = env["wsgi.input"].read(body_size)

    print "POST请求参数： %s " % query

    start_response('200 OK', [('Content-Type','text/html')])
    return ["Hello World"]
```

### 2. 执行uWSGI

```sh
$ ./uwsgi uwsgi --http :9090 --wsgi-file demo.py
```

### 3. 解析POST请求参数

```sh
$ curl "host:9090" -d "query=hello word"
$ POST请求参数: query=hello world
```

## 参考资料
[1] [WSGI 简介](https://segmentfault.com/a/1190000003069785)   
[2] [Python/WSGI 应用快速入门](http://uwsgi-docs-cn.readthedocs.io/zh_CN/latest/WSGIquickstart.html)






