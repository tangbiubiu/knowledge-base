Enum是python的内置模块。作用是在枚举时提高代码的安全性和可读性。

看下列枚举场景：

```python
class Weekday():
    monday = 1
    tuesday = 2
    wednesday = 3
    thirsday = 4
    friday = 5
    saturday = 6
    sunday = 7
 
print(Weekday.wednesday)    #  3
```

它是可以运行的，但存在两个缺陷：

枚举类型可以被修改。
不同的成员可以被赋予相同的值
这两点都是我们不希望看到的，使用Enum就可以很好的管理枚举类。

要是用Enum，只需使枚举类继承Enum类即可，Enum的成员不允许轻易改写：

```python
from enum import Enum
class Weekday(Enum):
    monday = 1
    tuesday = 2
    wednesday = 3
    thirsday = 4
    friday = 5
    saturday = 6
    sunday = 7
 
print(Weekday.wednesday)         # Weekday.wednesday      
print(type(Weekday.wednesday))   # <enum 'Weekday'>
print(Weekday.wednesday.name)    # wednesday
print(Weekday.wednesday.value)   # 3
```

有时，我们不在意枚举的值是多少，可以使用enum.auto()自动赋值。

```python
from enum import Enum auto

class Weekday(Enum):
    monday = auto()
    tuesday = auto()
    wednesday = auto()
    thursday = auto()
    friday = auto()
    saturday = auto()
    sunday = auto()
```

在一些情况下，我们希望枚举的值是指定的数据类型，比如int或者str，我们就可以使用IntEnum或者StrEnum。

```python
from enum import StrEnum auto

class Color(StrEnum):
    RED = 'red'
    GREEN = 'green'
    BLUE = auto()  # 你也可以使用auto
```

一般我们不希望枚举类型出现重复，这时可以使用装饰器unique，它会检查。

```python
from enum import Enum unique

@unique
class Color():
    red = 1
    blue = 1  # 这里的出现重复的值，触发ValueError。
```