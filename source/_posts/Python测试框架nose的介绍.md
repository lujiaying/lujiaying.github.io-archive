title: Python测试框架nose的介绍
date: 2015-06-04 17:20:18
tags:
---
下面以一个例子说明[nose](https://nose.readthedocs.org/en/latest/)。
目录结构如下（foo模块以及foo的测试代码）：

```shell
test@local:~$ tree /tmp/foomodule/
/tmp/foomodule/
|-- foo
|   |-- a.py
|   |-- b.py
|   `-- __init__.py
`-- tests
    |-- test_a.py
    `-- test_b.py
```

模块代码如下：

```Python
# /tmp/foomodule/foo/a.py 

def add(a, b):
    return a + b
def double(a):
    return a * 2

# /tmp/foomodule/foo/b.py 

import memcache
class Cache:
    def __init__(self, server):
        self.cache = memcache.Client([server])
    def get(self, name):
        return self.cache.get(name)
    def set(self, name, value):
        return self.cache.set(name, value)
    def delete(self, name):
        return self.cache.delete(name)
    def close(self):
        self.cache.disconnect_all()
```


对应的测试代码：

```Python
# /tmp/foomodule/tests/test_a.py 
from foo.a import add, double
def test_add():
    v = add(10, 20)
    assert v == 30
def test_double():
    v = double(10)
    assert v == 20

# /tmp/foomodule/tests/test_b.py 
from foo.b import Cache
class TestCache:

    def setUp(self):
        self.cache = Cache("127.0.0.1:11211")
        self.key   = "name"
        self.value = "smallfish"

    def tearDown(self):
        self.cache.close()

    def test_00_get(self):
        v = self.cache.get(self.key)
        assert v == None

    def test_01_set(self):
        v = self.cache.set(self.key, self.value)
        assert v == True
        v = self.cache.get(self.key)
        assert v == self.value

    def test_02_delete(self):
        v = self.cache.delete(self.key)
        assert v == True
```

先不解释，跑一跑（-v -s选项，针对当前路径下tests目录测试)：

```shell
test@local:/tmp/foomodule$ nosetests -s -v
test_a.test_add ... ok
test_a.test_double ... ok
test_b.TestCache.test_00_get ... ok
test_b.TestCache.test_01_set ... ok
test_b.TestCache.test_02_delete ... ok
----------------------------------------------------------------------
Ran 5 tests in 0.024s
OK
```

nosetests会自动进行集成测试。nose默认是对当前目录下tests目录进行测试，默认规则：文件（含有test），函数（test_开头），类（Test开头）。

主要使用assert 进行断言。
