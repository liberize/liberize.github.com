---
layout: post
title: "Python 相对导入"
keywords: ["Python", "相对导入"]
description: "Python 相对导入简介"
category: "tech"
tags: ["python"]
---
{% include JB/setup %}

### 1、绝对导入和相对导入

绝对导入：按照sys.path顺序搜索，先主目录（sys.path中第一项），然后PYTHONPATH环境变量、标准库路径、pth指定路径等。

相对导入：在模块所在同一个包内搜索，注意该包目录与主目录的区别。

例1：有以下目录

```
app/
    __init__.py
    mod.py
    string.py
```

mod.py内容：`import string`

当在app/目录下执行python mod.py时为绝对导入，当在app上层目录执行python -m app.mod时为相对导入。

### 2、模块搜索顺序

在python 2.7及之前版本中默认是先“相对”后“绝对”的顺序搜索模块，也就是说先在模块所在同一个包内搜索然后在sys.path中搜索。

在上例中，在app上层目录执行python -m app.mod时，将导入app/string.py（可以在string.py中print或者在mod.py中加入print string.__file__来测试）。

使用以下语句将会只搜索绝对路径：

```python
from __future__ import absolute_import
```

在mod.py开头加上该语句，在app上层目录执行python -m app.mod时，将导入标准库中的string模块。

在python3.3中默认只搜索绝对路径，要使用相对导入，执行以下语句：

```python
from . import string
```

_注意_：开头点号只能用在from语句中，不能用在import语句中。

### 3、相对导入与`__name__`属性

相对导入使用模块的`__name__`属性来决定模块在包结构中的位置。当__name__属性不包含包信息（i.e. 没有用'.'表示的层次结构，比如'__main__'），则相对导入将模块解析为顶层模块，而不管模块在文件系统中的实际位置。

例2：

```
app/
    __init__.py
    sub1/
         __init__.py
         mod1.py
    sub2/
         __init__.py
         mod2.py
```

尝试在mod1.py导入mod2.py，加入`from ..sub2 import mod2`。

直接在sub1目录下执行python mod1.py或在app目录下执行`python sub1/mod1.py`将报错："`Attempted relative import in non-package`"。

在app目录下执行`python -m sub1.mod1`也将报错："`Attempted relative import beyond toplevel package`"。

_正确的做法_：

在app上层目录执行`python -m app.sub1.mod1`，或者不要使用`from ..sub2 import mod2`而改用其他方式（比如将sub2添加到sys.path）。

例3：

```
__init__.py
start.py
parent.py
sub/
    __init__.py
    relative.py
```

start.py中包含`import sub.relative`，relative.py中包含`from .. import parent`。

执行`python start.py`将报错："`Attempted relative import beyond toplevel package`"。

_解决办法_：

新建pkg目录，将parent.py、sub目录移到pkg目录中，start.py改为`import pkg.sub.relative`，其它不变。
