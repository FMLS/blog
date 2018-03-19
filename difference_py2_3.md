# python 2 3主要区别

## print

Py2

```python
print 'hello world'
```

py3

```Python
print ('hello world')
```

---

##Ordering Comparisons

python3中` <, <=, >=, >`四个运算符在不同类型进行没有意义的比较时会抛出TypeError异常

py2

```python
>>> 1 < ''
True
>>> 0 > None
True
>>> len < len
False
```

py3

```python
>>> 1 < ''
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: '<' not supported between instances of 'int' and 'str'
```

但是注意,`== , !=` 两个运算符对上上述比较是有意义的

python3中cmp函数已经弃用, `__cmp__`方法也弃用了, 取而代之的是`__lt__`, `__hash__`等方法

builtin.sort和list.sort已经不再接收cmp参数, 取而代之的是key命名参数, 

---

##int long


Python 2中数字分为int, long两种类型, python3中只有int类型



---
## 整数除法

py2

```python
>>> 3 / 2
1
>>> 3 / 2.0
1.5
>>> 3 // 2
1
>>> 3 // 2.0
1.0
```

py3

```python
>>> 3 / 2
1.5
>>> 3 / 2.0
1.5
>>> 3 // 2
1
>>> 3 // 2.0
1.0
```

---

## unicode

在python3中用text和二进制 data的概念来代替python2的unicode 和8-bit string, 3中str对象都是unicode编码, bytes是二进制编码, 3中凡是尝试混合unicode和bytes的行为都会抛出TypeError

移除了basestring, str和bytes两个类没有共同继承关系

py2

```python
>>> print type(unicode('hello world'))
<type 'unicode'>
>>> print type(b'hello world')
<type 'str'>
>>> print 'hello ' + b'world'
hello world
```

py3

```python
>>> print('你好', type('你好'))
你好 <class 'str'>
>>> print ('hello ' + b'world')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: must be str, not bytes
```

---

## xrange

此函数在2中返回一个生成器, range则返回一个列表, xrange的惰性计算特性使之在很多一次性遍历大数据集的时候性能高, 如果需要多次遍历则不方便, 因为每次调用都会从头开始遍历. 在python3中, range函数实现了xrange的特性, xrange则不再使用. python3中的range类中新添加了\__contains__方法, 可以加速整数和布尔值的查找

py2

```python
>>> type(range)
<type 'builtin_function_or_method'>
>>> type(xrange)
<type 'type'>
```

py3

```python
>>> type(range)
<class 'type'>
>>> type(xrange)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'xrange' is not defined
```

---

## Raising exceptions

python2可以使用两种抛出异常的语法, python3只能把异常参数包裹在圆括号内, 并且在3中所有的异常必须继承自BaseException, 否则会抛出`TypeError: catching classes that do not inherit from BaseException is not allowed`

py2

```python
>>> raise IOError, 'test'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IOError: test
  
>>> raise IOError('test')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IOError: test
```

py3

```python
>>> raise IOError, 'test'
  File "<stdin>", line 1
    raise IOError, 'test'
                 ^
SyntaxError: invalid syntax
  
>>> raise IOError('test')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
OSError: test
```

---

## Handling exceptions

python2支持两种语法处理异常, python3只支持 `except Exception as e:`这样的写法

py2

```python
>>> try:
...     raise IOError, 'test'
... except IOError, e:
...     print e
...
test

>>> try:
...     raise IOError, 'test'
... except IOError as e:
...     print e
...
test
```

py3

```python
>>> try:
...     raise IOError('test')
... except IOError as e:
...     print(e)
...
test

>>> try:
...     raise IOError('test')
... except IOError, e:
  File "<stdin>", line 3
    except IOError, e:
                  ^
SyntaxError: invalid syntax
```

---

## input raw_input

python2中由于input会转换输入到不同类型对象, 所以大多使用raw_input, 在python3中input已经修复了这个问题, 其行为和raw_input相同, 通过input输入的内容全部是str类型, raw_input在python3中已经弃用

py2

```python
>>> res = input()
123
>>> print type(res)
<type 'int'>

>>> res = raw_input()
123
>>> print type(res)
<type 'str'>
```

py3

```python
>>> res = input()
123
>>> print(type(res))
<class 'str'>

#抛出异常
>>> res = raw_input()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'raw_input' is not defined
```

---

## 一些经常使用的函数返回迭代对象代替列表

1. range
2. zip
3. map
4. filter
5. {}.keys               {}.iterkeys()弃用
6. {}.values            {}.itervalues()弃用
7. {}.items             {}.itemitems() 弃用

## 弃用的函数

1. {}.has_key      用`in` 代替

---

## round half to even

python3这种规约方法目标是最大限度减少求和后的误差, python2里面使我们熟知的**四舍五入**

py2

```python
>>> round(1.5)
2.0
>>> round(2.5)
3.0
```

py3

```python
>>> round(1.5)
2
>>> round(2.5)
2
```

---

## OO

### super

python3中super方法可以不加参数直接调用

```python
class base(object):
        def __init__(self):
                print('I am base class')

class derived(base):
        def __init__(self):
                super().__init__()
                print('I am derived class')

obj = derived()

#print
#I am base class
#I am derived class
```

