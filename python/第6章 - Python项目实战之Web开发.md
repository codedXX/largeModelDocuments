# Web开发

## 面向对象高级

~~~python
"""
封装: 将数据(属性)和操作数据的方法绑定在一起, 形成一个独立的单元(类), 保护数据不被外部访问，通过访问修饰符实现封装。
     1. 私有属性: 在属性名前加双下划线__
     2. 私有方法: 在方法名前加双下划线__
注意事项: Python中是没有真正的私有机制 ;
"""
class Car:
    def __init__(self, brand, model, color, owner):
        self.brand = brand          # 品牌(公有属性)
        self.model = model          # 型号(公有属性)
        self.color = color          # 颜色(公有属性)

        self.__owner = owner          # 拥有者(私有属性)

    def start(self): # 启动
        print(f'{self.brand} {self.model} 正在启动...')

    def run(self): # 行驶
        print(f'{self.__owner} : {self.brand} {self.model} 正在行驶...')
        self.__control_fuel()

    def stop(self): # 停止
        print(f'{self.brand} {self.model} 停止行驶...')

    def __control_fuel(self): # 私有方法
        print(f'{self.brand} {self.model} 正在控制油门...')

    def get_owner(self):
        return self.__owner[0:1] + "**"


if __name__ == '__main__':
    car = Car('Audi', 'A6', '黑色', '涛哥')
    print(car.brand)
    # print(car.model)
    # print(car.color)

    # Python中是没有真正的私有机制 ;
    # print(car._Car__owner)
    # car._Car__control_fuel()

    # car.start()
    # car.run()
    # car.stop()
    # print(car.get_owner())
~~~

>**【if __name__ == '__main__':】 这个代码是什么意思？**
>
>
>
>`if __name__ == '__main__':` 的意思是：
>
>**只有当这个 Python 文件被直接运行时，下面缩进的代码才会执行。**
>
>在该文件的第 32 行：
>
>```python
>if __name__ == '__main__':
>    car = Car('Audi', 'A6', '黑色', '涛哥')
>```
>
>当你直接运行：
>
>```python
>python 01.面向对象高级-封装.py
>```
>
>但如果另一个文件导入它：
>
>```python
>from 01.面向对象高级-封装 import Car
>```
>
>此时这个文件的 `__name__` 会是模块名，而不是 `'__main__'`，所以第 32 行下面的测试代码不会执行。这样就可以：
>
>- `Car` 类可以被其他文件复用；
>- 文件底部的示例、测试代码不会在导入时自动运行。
>
>这里的 `if __name__ == '__main__':` 可以理解为：“如果我是程序的直接入口，就运行以下演示代码。”



> ⭐⭐**“Python中并没有真正的私有机制” 这句话是什么意思？**
>
> 
>
> Python 中“没有真正的私有机制”是指：Python 没有像 Java 的 `private`、C++ 的 `private` 那样，在语言层面严格禁止类外访问成员的权限控制。
>
> ```python
> class User:
>     def __init__(self):
>         self._name = "Alice"
> ```
>
> `_name` 前面的单下划线只是一种**约定**：提示其他开发者“这是内部属性，请不要从类外直接使用”。 但仍可以直接访问：
>
> ```python
> user = User()
> print(user._name)  # Alice，完全可以访问
> ```
>
> 双下划线看起来更像“私有”：
>
> ```python
> class User:
>     def __init__(self):
>         self.__password = "secret"
> ```
>
> 下面的访问会失败：
>
> ```python
> user = User()
> print(user.__password)  # AttributeError
> ```
>
> 但这不是严格的权限限制，而是 Python 做了`名称改写`（name mangling）：
>
> ```python
> print(user._User__password)  # secret
> ```
>
> 也就是说：
>
> - `_field`：仅靠命名约定，表示“内部使用”。
> - `__field`：Python 会改写属性名，主要用于避免子类意外覆盖同名属性。
> - 两者都不能真正阻止外部代码读取或修改。
> - Python 倾向于“我们都是成年人”（consenting adults）原则：语言提供约定和工具，开发者自行遵守边界。 真正需要保护密码、密钥等敏感数据时，不能依赖 `__password`；应避免把敏感数据暴露在对象中，并使用权限控制、加密或安全存储。

### 封装

* 封装就是把数据(属性)和操作数据的函数(方法)捆绑在一起，形成一个独立的单元（类），并隐藏内部的实现细节，只对外暴露必要的功能（方法）。
  * ⭐`隐藏内部的实现细节`就是：`私有属性、私有方法`
  * ⭐`外暴露必要的功能（方法）`就是：`公共属性、公共方法`

~~~python
"""
封装: 将数据(属性)和操作数据的方法绑定在一起, 形成一个独立的单元(类), 保护数据不被外部访问，通过访问修饰符实现封装。
     1. 私有属性: 在属性名前加双下划线__
     2. 私有方法: 在方法名前加双下划线__
注意事项: Python中是没有真正的私有机制 ;
"""
class Car:
    def __init__(self, brand, model, color, owner):
        self.brand = brand          # 品牌(公有属性)
        self.model = model          # 型号(公有属性)
        self.color = color          # 颜色(公有属性)

        self.__owner = owner          # 拥有者(私有属性)

    def start(self): # 启动
        print(f'{self.brand} {self.model} 正在启动...')

    def run(self): # 行驶
        print(f'{self.__owner} : {self.brand} {self.model} 正在行驶...')
        self.__control_fuel()

    def stop(self): # 停止
        print(f'{self.brand} {self.model} 停止行驶...')

    def __control_fuel(self): # 私有方法
        print(f'{self.brand} {self.model} 正在控制油门...')

    def get_owner(self):
        return self.__owner[0:1] + "**"


if __name__ == '__main__':
    car = Car('Audi', 'A6', '黑色', '涛哥')
    print(car.brand)
    # print(car.model)
    # print(car.color)

    # Python中是没有真正的私有机制 ;
    # print(car._Car__owner)
    # car._Car__control_fuel()

    # car.start()
    # car.run()
    # car.stop()
    # print(car.get_owner())
~~~

> ⭐**私有：**私有的属性和方法只能在类的内部使用; Python中并`没有真正的私有机制`，`约定在私有属性名和方法名前加__(两个下划线)`。



#### 小结

1、**简单描述一下什么是封装？**

- 封装就是将数据（属性）及操作数据的方法绑定在一起，形成一个独立的类。

2、**私有属性、方法与公共的有什么区别呢？该怎么定义？**

- 私有的只能在类的内部访问，在外部是无法访问的；而公共的属性、方法在类的内部、外部都是可以访问的。

- 私有的属性和方法在命名的时候，均以 `__` 作为开头。（Python 中没有真正的私有机制）

3、**私有属性、方法的应用场景？**

- 如果某些属性和方法是不对使用者公开的，就可以将其设置为私有的。



### 继承

#### 介绍



* 继承描述的是两个类之间的关系，子类继承父类，就可以获取到父类的属性和方法。`(非私有)`

~~~python
"""
继承: 描述的是两个类之间的关系, 子类继承父类, 就可以获取到父类中的属性和方法 (非私有)
"""
class Car:
    def __init__(self, brand, model, color, owner):
        self.brand = brand          # 品牌(公有属性)
        self.model = model          # 型号(公有属性)
        self.color = color          # 颜色(公有属性)
        self.__owner = owner        # 拥有者(私有属性)

    def start(self): # 启动
        print(f'{self.brand} {self.model} 正在启动...')

    def run(self): # 行驶
        print(f'{self.__owner} : {self.brand} {self.model} 正在行驶...')
        self.__control_fuel()

    def stop(self): # 停止
        print(f'{self.brand} {self.model} 停止行驶...')

    def __control_fuel(self): # 私有方法
        print(f'{self.brand} {self.model} 正在控制油门...')

    def get_owner(self):
        return self.__owner[0:1] + "**"


# 燃油车
class FuelCar(Car):
    pass

# 电车
class ElectricCar(Car):
    pass

if __name__ == '__main__':
    c1 = FuelCar("BMW", "X5", "黑色", "张三")
    c1.start()
    c1.run()
    c1.stop()
    print(c1.brand)
    print(c1.get_owner())
    print(c1.model)
    print(c1.color)

~~~

##### 小结

1、继承的**语法格式**？

~~~python
class 子类(父类):
    pass
~~~

2、子类继承父类，是**将父类所有的属性和方法都继承下来了么**？

- 没有
- 子类继承父类，会将`父类公共的属性和方法继承下来（不含私有）`



#### 继承-重写

* `重写`是指子类继承父类后，如果父类中的方法不满足需求，可以在子类中重新定义父类中已有的方法（方法名相同），从而用子类的实现替换父类的实现。

~~~python
"""
继承: 描述的是两个类之间的关系, 子类继承父类, 就可以获取到父类中的属性和方法 (非私有)
重写: 是指子类继承父类后，如果父类中的方法不满足需求，可以在子类中重新定义父类中已有的方法（方法名相同），从而用子类的实现替换父类的实现。
"""
class Car:
    def __init__(self, brand, model, color, owner):
        self.brand = brand          # 品牌(公有属性)
        self.model = model          # 型号(公有属性)
        self.color = color          # 颜色(公有属性)
        self.__owner = owner        # 拥有者(私有属性)

    def start(self): # 启动
        print(f'{self.brand} {self.model} 正在启动...')

    def run(self): # 行驶
        print(f'{self.__owner} : {self.brand} {self.model} 正在行驶...')

    def stop(self): # 停止
        print(f'{self.brand} {self.model} 停止行驶...')

    def get_owner(self):
        return self.__owner[0:1] + "**"

    def charge(self):
        print(f'{self.brand} {self.model} 正在补充燃料...')

# 燃油车
class FuelCar(Car):
    def charge(self):
        # 方式一: super().方法名()
        # super().charge()

        # 方式二: 类名.方法名(self)
        Car.charge(self)
        print(f'{self.brand} {self.model} 正在加油...')

# 电车
class ElectricCar(Car):
    def charge(self):
        Car.charge(self)
        print(f'{self.brand} {self.model} 正在充电...')


if __name__ == '__main__':
    c1 = FuelCar("BMW", "X5", "黑色", "张三")
    c1.charge()
~~~

> **FuelCar里重写的charge方法里面为什么要加Car.charge(self)，不加的话也算重写吗？**
>
> * 算，是否写 `Car.charge(self)`，**不影响它是不是重写**。
> * `Car.charge(self)` 的作用是：在子类重写的方法里，额外执行一次父类原本的 `charge()` 逻辑。

##### 小结

1、什么是方法重写？重写的语法？

- 在继承体系中，如果父类的方法不能满足需求，就可以在子类中重新定义一个与父类同名的方法，从而用子类自己的实现来替换父类的实现，这个就称之为重写。 2、在子类中，如何调用父类中的方法？
- 方式一：`super().方法名()`
- 方式二：`父类名.方法名(self)`



#### 多继承

* 多继承指的是一个子类，同时继承了多个父类的情况`（会将多个父类中的非私有的属性和方法都继承下来）`。

* 语法：

  ~~~python
  class 子类(父类1, 父类2, 父类3, ...):
      代码
  ~~~

~~~python
"""
多继承: 一个子类继承了多个父类
"""
class Car:
    def __init__(self, brand, model, color, owner):
        self.brand = brand          # 品牌(公有属性)
        self.model = model          # 型号(公有属性)
        self.color = color          # 颜色(公有属性)
        self.__owner = owner        # 拥有者(私有属性)

    def start(self): # 启动
        print(f'{self.brand} {self.model} 正在启动...')

    def run(self): # 行驶
        print(f'{self.__owner} : {self.brand} {self.model} 正在行驶...')

    def stop(self): # 停止
        print(f'{self.brand} {self.model} 停止行驶...')

    def get_owner(self):
        return self.__owner[0:1] + "**"

    def charge(self):
        print(f'{self.brand} {self.model} 正在补充燃料...')


# 华为智驾
class HuaweiAiDriving:
    """华为AI智能驾驶"""
    def __init__(self, version="V1.0"):
        self.version = version

    def run(self):
        print(f'使用华为AI智能驾驶系统{self.version}正在行驶...')


# 问界汽车
class WenJieCar(Car, HuaweiAiDriving):
    def __init__(self, brand, model, color, owner, version = "V1.0"):
        Car.__init__(self, brand, model, color, owner)
        HuaweiAiDriving.__init__(self, version)

    def run(self):
        Car.run(self)
        HuaweiAiDriving.run(self)



# MRO: Method Resolution Order --> 方法解析顺序
if __name__ == '__main__':
    c = WenJieCar("BMW", "X5", "黑色", "张三", "V1.1")
    print(c.__dict__)

    # print(WenJieCar.__mro__)
    # print(WenJieCar.mro())

    c.run()
~~~





> **注意：** 当一个类继承了多个父类时，默认优先使用第一个父类中的同名属性或方法，可以使用 类名.__mro__ 属性 或 类名.mro() 方法查看调用顺序。
>
> * `MRO : Method Resolution Order(方法解析顺序)`

### 多态

#### 介绍

* 多态是指同一个方法，具有不同的形态、行为、表现。

~~~python
"""
多态: 是指同一个方法，具有不同的形态、行为、表现
"""
class Car:
    def __init__(self, brand, model, color, owner):
        self.brand = brand          # 品牌(公有属性)
        self.model = model          # 型号(公有属性)
        self.color = color          # 颜色(公有属性)
        self.__owner = owner        # 拥有者(私有属性)

    def start(self): # 启动
        print(f'{self.brand} {self.model} 正在启动...')

    def run(self): # 行驶
        print(f'{self.__owner} : {self.brand} {self.model} 正在行驶...')

    def stop(self): # 停止
        print(f'{self.brand} {self.model} 停止行驶...')

    def get_owner(self):
        return self.__owner[0:1] + "**"

    def charge(self):
        print(f'{self.brand} {self.model} 正在补充燃料...')

# 燃油车
class FuelCar(Car):
    def charge(self):
        print(f'{self.brand} {self.model} 正在加油...')

# 电车
class ElectricCar(Car):
    def charge(self):
        print(f'{self.brand} {self.model} 正在充电...')


# 补充燃料函数
def handle_charge(car: Car): # 函数参数类型声明 --- 指定的是父类型
    car.charge()


# 测试代码
if __name__ == '__main__':
    handle_charge(FuelCar("BMW", "X5", "黑色", "张三"))
    handle_charge(ElectricCar("BYD", "汉", "黑色", "李四"))

~~~



##### 小结

1、什么是多态？

- 多态是指同一个方法，具有不同的表现形态。

- 如：定义函数时，参数类型指定为父类类型，在执行的时候传入不同的子类对象，就具有不同的形态。

  ~~~python
  # 充燃料函数
  def handle_charge(car: Car):  # 函数参数类型声明，指定的是父类类型
      car.charge()
  # 测试代码
  if __name__ == '__main__':
      handle_charge(FuelCar("BMW", "X5", "黑色", "张三"))
      handle_charge(ElectricCar("BYD", "汉", "黑色", "李四"))
  ~~~



#### 多态-鸭子类型

* `鸭子类型(Duck Typing)`是指如果它走路像鸭子，叫起来叫鸭子，那么它就是鸭子。
* 在鸭子类型中，我们`关注`的是对象的`行为`（它有什么方法），而不是对象的类型（它是什么类）。

~~~python
"""
鸭子类型: 如果它走路像鸭子，叫起来像鸭子，那么它就是鸭子 。 -- 关注的是对象的行为（方法），而不是对象的类型
"""
class Duck:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def swimming(self):  # 启动
        print(f'Duck: {self.age} 岁的 {self.name} 正在游泳...')

class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def swimming(self): # 启动
        print(f'Dog: {self.age} 岁的 {self.name} 正在游泳...')


class Pig:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def swimming(self):  # 启动
        print(f'Pig: {self.age} 岁的 {self.name} 正在游泳...')


def go_swimming(duck):
    duck.swimming()


# 测试代码
if __name__ == '__main__':
    go_swimming(Dog("旺财", 4))
    go_swimming(Duck("唐老鸭", 2))
    go_swimming(Pig("佩奇", 1))

~~~

> 提示：鸭子类型的优势是不需要存在继承关系，只要对象有相应的方法就能使用。



#### 多态和鸭子类型的区别

| 对比点       | 经典的多态                      | 鸭子类型多态                               |
| ------------ | ------------------------------- | ------------------------------------------ |
| 类之间的关系 | FuelCar、ElectricCar 都继承 Car | Duck、Dog、Pig 彼此没有继承关系            |
| 相同方法     | 子类重写父类的 charge()         | 各个无关的类都自己定义 swimming()          |
| 函数参数     | def handle_charge(car: Car)     | def go_swimming(duck)                      |
| 对象要求     | 设计意图是传入 Car 或其子类     | 不管是什么类型，只要有 swimming() 方法即可 |
| 多态来源     | 继承 + 方法重写                 | 行为相同 + 鸭子类型                        |

### 案例



## FastAPI基础

## 汉字谜盒案例

