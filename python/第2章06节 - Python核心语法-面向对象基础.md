# Python 核心语法：面向对象基础与异常

> **学习目标：** 用类描述业务对象，区分实例与类属性，并以异常处理保证程序稳定。

## 类、对象与方法

面向过程关注步骤，面向对象关注对象的状态和行为。类是模板，对象是具体实例；实例方法的第一个参数通常是 `self`。

```python
class Student:
    school = "传智教育"

    def __init__(self, name, score):
        self.name = name
        self.score = score

    def passed(self):
        return self.score >= 60
```

`self.name` 是每个对象独有的实例属性；`school` 是所有实例共享的类属性。通过实例给同名属性赋值会创建实例属性，不会修改类属性。

> **重点：** `__init__` 用于初始化；`__str__`、`__len__` 等魔法方法只在能让对象更自然地使用时实现。

## 异常处理

异常是运行时错误。使用 `try...except` 在预期失败的位置捕获具体异常，`else` 放成功路径，`finally` 放必须执行的收尾逻辑。

```python
try:
    number = int(input("整数："))
except ValueError:
    print("输入不是整数")
else:
    print(number)
```

> **重点：** 不要无差别捕获并静默忽略 `Exception`。异常会沿调用栈向上传递，直到被处理或导致程序终止。
