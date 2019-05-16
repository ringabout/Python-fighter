### Python 标准库学习 --- string

想要代码写得好，除了参与开源项目、在大公司实习，最快捷高效的方法就是阅读 Python 标准库。学习 Python 标准库，不是背诵每一个标准库的用法，而是要过一遍留下印象，挑自己感兴趣的库重点研究。这样实际做项目的时候，我们就可以游刃有余地选择标准库。

作为这一系列的开始，第一个学习的是 string 模块。string 模块作为内置函数 str 的补充，提供了一些便利的函数。我会持续分享自己关于标准库的学习笔记与思考，争取一两周更新一篇标准库的内容。记得给公众号加个星标，不会错过精彩内容。还可以在 [github](https://github.com/xflywind/Python-fighter/blob/master/README.md) 上给我提 issue，我尽力回答。

#### 第一步

```python
# 导入 string 模块
import string
```
#### capwords

string 模块中提供了 capwords 函数，该函数使得字符串中的每个单词变为大写形式。我们来看看源码中是如何定义的:

```python
def capwords(s, sep=None):
    return (sep or ' ').join(x.capitalize() for x in s.split(sep))
```
capwords 接受一个位置参数：待处理的字符串，和一个可选关键字参数：字符串的分隔符。字符串默认使用空格分隔，比如 ‘my name is python ’，也可以指定 seq 分隔，比如传入 seq 为 ‘-’：‘my-name-is-python’。这个函数使得被分隔的单词首字母大写。

```python
>>> s = 'my name is python'
>>> capwords(s)
'My Name Is Python'

>>> s = 'my-name-is-python'
>>> capwords(s，'-')
'My-Name-Is-Python'
```

总结一下子：我们需要首先向 capwords 函数中传入字符串。capwords 函数通过 str.split 方法将字符串分割成单词，再通过生成器表达式和 str.capitalize 方法，使得每一个单词首字母大写，最后再通过 str.join 方法将单词拼装为字符串。

上面是 cpython 的实现。对于标准库中比较简单的函数，我们可以考虑，如果是自己的话，会用什么方法写这个函数，最后再使用 timeit 模块比较一下这两者的性能。

我举个例子，比如说，这个函数还可以使用 map 函数重写，下面这两种方法实质上和 cpython 的实现等价的。一个使用了 str 的 capitalize 方法，另一个通过 methodcaller 方法调用字符串的 capitalize 方法。

```python
def capwords1(s:str, seq:str=None)->str:
    return (seq or ' ').join(map(str.capitalize, s.split(seq)))


from operator import methodcaller

def capwords2(s:str, seq:str=None)->str:
    return (seq or ' ').join(map(methodcaller('capitalize'), s.split(seq)))
```
我们再和标准实现比较性能,我是在 ipython 上测试的:
```python
text = "your time is limted, so don't waste it living someone else's lives" * 10000
%timeit capwords(text)
24.9 ms ± 588 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
%timeit capwords1(text)
22.1 ms ± 721 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
%timeit capwords2(text)
28.4 ms ± 3.38 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```
通过测试，我们可以发现，我们实现的第一个版本的函数，性能可能好一些；而第二个版本的实现则要逊色不少。

### Template

下面是 Template 的基本用法，这是 string 模块提供给我们的字符串插值函数。该函数会将传进来的参数转化为字符串，然后进行插值，所以不支持格式化字符串 ，但是优点是更加安全。

首先建立一个模板接受 string 参数，string 的格式要求为：`$` + 标识符(首个字符必须为 字母或者下划线，之后的字符只能是 字母、下划线、数字)，使用 substitute 方法，我们就可以替换标识符。

匹配的样式：`$$`, %name, %{name}

```python
from string import Template
string = '姓名：$name 年龄：${age} 爱好：$hobby'
template = Template(string)
```

substitue 的参数可以是字典：

```python
>>> template.substitute({'name':'Python', 'age': 30, 'hobby':'all'})
'姓名：Python 年龄：30 爱好：all'
```

还可以是关键字参数：

```python
>>> template.substitute(name='Python', age=30, hobby='all')
'姓名：Python 年龄：30 爱好：all'
```

关键字错误，解释器会报 KeyError:

```python
>>> template.substitute(name='Python', age=20, hobb='all')
KeyError: 'hobby'
```

这时候，我们可以使用 template 提供的另外一个方法 safe_subsitute 来防止编译器报错。当 safe_substitute 方法没有找到相应的关键字，会原封不动地返回标识符。

```python
>>> template.safe_substitute(name='Python', age=30, hobb='all')
'姓名：Python 年龄：30 爱好：$hobby'
```

Template 有四个类属性，其中 delimiter 为分隔符，默认为 `$` ，后面接标识符。通过重写 delimiter，我们可以支持 % 等符号替换。类属性 idpattern 为标识符匹配规则，类属性 flags 表示忽略大小写。

```python
class Template(metaclass=_TemplateMetaclass):
    """A string class for supporting $-substitutions."""

    delimiter = '$'
    idpattern = r'(?a:[_a-z][_a-z0-9]*)'
    braceidpattern = None
    flags = _re.IGNORECASE
```

比如说，我们可以重写类属性 delimiter 和 idpattern。

```python
class MyTemplate(Template):
    delimiter = '%'
    idpattern = '[_][a-z]+_[a-z]+'
```

上面我们自定义了一个类，继承自 string.Template，并重写了 delimiter 和 idpattern 类属性。

```python
>>> s = '%_name_main %age'
>>> template = MyTemplate(s)
>>> template.substitute(_name_main='Python', age = 30)
ValueError: Invalid placeholder in string
>>> template.safe_substitute(_name_main='Python', age = 30)
'Python %age'
```

我们可以看到，分隔符已经换成了百分号，而标识符必须符合 `_字母_字母`的形式，否则会提示 valueError。

我们还可以从源码中学到一些技巧：

```python
from collections import ChainMap as _Chainmap

def substitute(*args, **kws):
    mapping = _ChainMap(kws, args[0])
```

*args 接受一个字典, kws 接受关键字参数，Chainmap 函数将多个映射连接起来，就可以查找 args 和 kws 中的关键字。

以上就是我学习 Python 标准库的思考，还请大家多多转发支持。







