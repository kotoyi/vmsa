# static method

1. 与类绑定，与类对象无关，不依赖于对象的创建，或者对象的状态
1. 没有对象的信息，仅能处理输入参数
1. 可以被类直接调用，也可以被类对象调用
### 使用场景
1. 打包一些功能函数到一个类中，当某些功能函数并不需要使用类本身，但是放在一个类中是由意义的，则可以使用static method
1. 当该方法只有一个独立的实现，不会在子类中被覆盖的时候
# class method
1. 与类绑定，与类对象无关，不依赖于对象的创建，或者对象的状态
1. 参数就是类本身
1. 可以被类直接调用，也可以被类对象调用
## 使用场景
1. 工厂方法，返回不用使用场景的类对象的方法
```python
from date.datetime import date

# random Person
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    @classmethod
    def fromBirthYear(cls, name, birthYear):
        return cls(name, date.today().year - birthYear)
    
    def display(self):
        print(self.name + "'s age is: " + str(self.age))

person = Person('Adam', 19)
person.dispaly()

person1 = Person.fronBirthYear('John', 1985)
person1.display()
```
Output:
```
Adam's age is: 19
John's age is: 38
````
2. 在继承类中进行正确的实例创建

每当您通过将工厂方法实现为类方法来派生类时，它都会确保派生类的正确实例创建。

您可以为上面的示例创建一个静态方法，但它创建的对象将始终被硬编码为基类。

但是，当您使用类方法时，它会创建派生类的正确实例。
```python
from datetime import date

# random Person
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @staticmethod
    def fromFathersAge(name, fatherAge, fatherPersonAgeDiff):
        return Person(name, date.today().year - fatherAge + fatherPersonAgeDiff)

    @classmethod
    def fromBirthYear(cls, name, birthYear):
        return cls(name, date.today().year - birthYear)

    def display(self):
        print(self.name + "'s age is: " + str(self.age))

class Man(Person):
    sex = 'Male'

man = Man.fromBirthYear('John', 1985)
print(isinstance(man, Man))

man1 = Man.fromFathersAge('John', 1965, 20)
print(isinstance(man1, Man))
```
Output:
```
True
False
```
# 一等对象
1. 运行时创建
1. 能赋值给变量或数据结构中的元素
1. 能作为参数传递给函数
1. 能作为函数的返回结果
在python中，所有的函数都是一等对象

python函数是对象，__doc__用于生成对象的帮助文本，是函数的属性，函数对象本身是function类的实例

以下示例展示了一等函数的本性，赋值，传参。
map函数返回一个可迭代对象，里面的元素是把第一个参数应用到第二个参数中各个元素上得到的结果
```python
def factorial(n):
    #return n!
    return 1 if n < 2 else n * factorial(n-1)
```
Output:
```
[1, 1, 2, 6, 24， 120， 720， 5040， 40320， 362880， 3628800]
```
## 高阶函数
接受函数作为参数，或者把函数作为返回结果的函数是高阶函数
## 匿名函数
lambda函数的定义体中不能赋值，也不能使用while和try等python语句，可用作参数传递给高阶函数
```
sorted(fruits, key=lambda word: word[::-1])
```
## 可调用对象
1. 用户定义函数：def语句或lambda表达式创建
1. 内置函数：C实现的函数，如len或time.strftime
1. 内置方法：C实现的方法，如dict.get
1. 方法：在类的定义体中定义的函数
1. 类：调用类时会运行__new__方法创建一个实例，然后运行__init__方法，初始化实例，最后把实例返回给调用方
1. 类的实例：如果定义了__call__方法，那么类实例可以用作函数调用
1. 生成器函数：使用yield关键字的函数或方法。调用生成器函数返回的时生成器对象
# 用户定义的可调用类型
任何python对象都可以表现得像函数，只需实现实例方法__call__
```python
import random
class BingoCage():
    def __init__(self, item):
        ...

    def pick(self):
        try:
            return self._items.pop()
        except IndexError:
            raise LookupError('pick from empty BingoCage')
    def __call__(self):
        return self.pick()
```
演示
```python
bingo = BingoCage(range(3))
bingo.pick()
-->1
bingo()
-->0
```
函数类对象必须在内部维护一个状态，让它在调用之间可用，例如BingoCase中的剩余元素。

创建保有内部状态的函数，还可以使用闭包
# 执行
## eval
## getattr
## \_\_getattribute__