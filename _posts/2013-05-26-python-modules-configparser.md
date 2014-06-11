---
layout: post
title: "Python 模块学习之 ConfigParser"
keywords: ["Python", "ConfigParser"]
description: "Python 模块学习之 ConfigParser"
category: "tech"
tags: ["python", "module"]
---
{% include JB/setup %}

在程序中使用配置文件来灵活的配置一些参数是一件很常见的事情，配置文件的解析并不复杂，在Python里更是如此，在官方发布的库中就包含有做这件事情的库，那就是ConfigParser，这里简单的做一些介绍。

Python ConfigParser模块解析的配置文件的格式比较象ini的配置文件格式，就是文件中由多个section构成，每个section下又有多个配置项，比如：

```ini
[db]
db_host = 127.0.0.1
db_port = 3306
db_user = root
db_pass = password
db_admin = True

[concurrent]
thread = 10
processor = 20
```

假设上面的配置文件的名字为test.conf。里面包含两个section,一个是db, 另一个是concurrent, db里面还包含有4项，concurrent里面有两项。

首先，读取配置文件：

```python
#-*- encoding: utf-8 -*-
import ConfigParser
cf = ConfigParser.ConfigParser()
cf.read("test.conf")
```

然后获取sections、options和items：

```python
print 'section:', cf.sections()
print 'options:', cf.options("db")
print 'db:', cf.items("db")
```

读取数据，使用`get`、`getfloat`、`getint`、`getboolean`等方法：

```python
print "db_host:", cf.get("db", "db_host")
print "db_port:", cf.getint("db", "db_port")
print "db_user:", cf.get("db", "db_user")
print "db_pass:", cf.get("db", "db_pass")
print "db_admin:", cf.getboolean("db", "db_admin")
```

修改数据，使用set方法：

```python
# 修改一个值
cf.set("db", "db_pass", "mypass")
cf.set("db", "db_admin", False)
```

修改section、option，使用`add_section`、`remove_section`、`remove_option`等方法：

```python
# 增加一个section和option
cf.add_section("test")
cf.set("test", "info", "hello")

# 删除option和section，返回True或False
cf.remove_option("concurrent", "thread")
cf.remove_section("concurrent")

# 测试section和option是否存在
if cf.has_option("test", "info"):
	print "section 'test' has option 'info'"
if cf.has_section("notexist"):
	print "config file has section 'notexist'"
```

最后写入文件：

```python
cf.write(open("test.conf", "w"))
```

以上就是对Python ConfigParser模块的相关应用方法的简单介绍。
