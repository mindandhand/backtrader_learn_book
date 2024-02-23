类是一种模板，用于创建对象。对象是根据类创建的实例，它们拥有类定义的属性和方法。通过创建类的实例（即对象），可以在程序中实现抽象的概念，并使用这些对象进行交互，从而实现复杂的程序逻辑。

元类（metaclass）是一个高阶概念，它用于创建类。可以把元类看作是创建类的"类"。当创建一个类时，实际上是在创建一个元类的实例。

元类的主要用途是改变类的创建行为，或者在类创建之后对其进行修改。例如，你可以使用元类来自动添加方法、修改属性或执行其他任何在类创建时需要的操作。


##### class关键字创建类


```python
class Pig:  
    # 类变量
    species = "Animal"  
  
    # 初始化方法，在创建新实例时自动调用  
    def __init__(self, name, age):  
        # 实例变量，每个实例都有自己独立的值  
        self.name = name  
        self.age = age  
  
    # 一个实例方法
    def bark(self):  
        return f"{self.name} says, 'I am a Pig!'"  
    
# 创建Dog类的一个实例  
peppa_pig = Pig("Peppa", 4)  
  
# 访问实例变量  
print("name:",peppa_pig.name)  # 输出: Peppa  
print("age: ", peppa_pig.age)   # 输出: 4 
  
# 调用实例方法  
print(peppa_pig.bark())  # 输出:I am PeppaPig  
  
# 访问类变量  
print(Pig.species)  # 输出: Animal
```

    name: Peppa
    age:  4
    Peppa says, 'I am a Pig!'
    Animal



```python
#### Pig 是一个类，也是一个对象，可以进行拷贝
CopyPig = Pig
george_pig = CopyPig("George", 2)
# 打印copy 创建的对象
print(george_pig)
# 访问实例变量  
print("name:",george_pig.name)  
print("age: ", george_pig.age)  

#### 类可以作为参数
def call_bark(o):
    print(o.bark())
call_bark(george_pig)

# 内置方式 __class__ 保存了创建这个对象的类
# 查看 george_pig的__class__, 是Pig创建的
print(george_pig.__class__)
# Pig是谁创建的呢？
print(george_pig.__class__.__class__)
# 创建Pig的类又是谁创建的
print(george_pig.__class__.__class__.__class__)
```

    <__main__.Pig object at 0x7fe0287b2860>
    name: George
    age:  2
    George says, 'I am a Pig!'
    <class '__main__.Pig'>
    <class 'type'>
    <class 'type'>


从上面的结果可以看到，最终类的创建可以归结到type。
##### type关键字创建类
type 是 Python 中所有元类的基类,可以使用 type 函数来动态地创建类,type 函数接收三个参数：类名、基类元组和类体字典。
下面几个例子说明type 的用法：


```python

### 以下2种方式创建的类是一样的
# 方式 1 
class Dog1: 
    pass
print(Dog1())
print(Dog1().__class__)

# 方式 2
# 可以使用 type 函数来动态地创建类。type 函数接收三个参数：类名、基类元组和类体字典。
Dog2 = type("Dog2",(),{})
print(Dog2())
print(Dog2().__class__)
```

    <__main__.Dog1 object at 0x7fe0287b0d60>
    <class '__main__.Dog1'>
    <__main__.Dog2 object at 0x7fe029c4d900>
    <class '__main__.Dog2'>



```python
# 1 带有类属性的类
Dog3 = type("Dog3",(),{"specias":"animal"})
print("Dog3:", Dog3.specias)
print('---------------')


# 2 继承自Dog3的类，继承类写在第二个参数，是一个元组
ChildDog = type("ChildDog",(Dog3,),{})
print("ChildDog:", ChildDog.specias)
print('---------------')


# 3 带有实例方法的类
def bark(self):
    return ("Wo Wo Wo!")

Dog4 = type("Dog4",(),{"specias":"animal","bark":bark})
print("hasattr bark:",hasattr(Dog4,"bark"))
print("Dog4 func:", Dog4.bark)
print("Dog4 func():", Dog4().bark())
print('---------------')


# 4 带有静态方法的类

@staticmethod
def test_static_method():
    return ("static method ....")

Dog5 = type("Dog5",(),{"specias":"animal","bark":bark,"test_static_method":test_static_method})    
print("hasattr test_static_method:",hasattr(Dog5,"test_static_method"))
print("Dog5 func:", Dog5.test_static_method)
print("Dog5 func():", Dog5().test_static_method())
print('---------------')


# 5 带有类方法的类 
# 注意：类方法有参数，一般约定为cls，静态方法没有，静态方法不能调用类参数
@classmethod
def test_class_method(cls):
    return (f"class method ....",cls.specias)
Dog6 = type("Dog6",(),{"specias":"animal","bark":bark,"test_class_method":test_class_method})
print("hasattr test_class_method:",hasattr(Dog6,"test_class_method"))
print("Dog6 func:", Dog6.test_class_method)
print("Dog6:", Dog6().test_class_method())

```

    Dog3: animal
    ---------------
    ChildDog: animal
    ---------------
    hasattr bark: True
    Dog4 func: <function bark at 0x7fe0298269e0>
    Dog4 func(): Wo Wo Wo!
    ---------------
    hasattr test_static_method: True
    Dog5 func: <function test_static_method at 0x7fe029bd0040>
    Dog5 func(): static method ....
    ---------------
    hasattr test_class_method: True
    Dog6 func: <bound method test_class_method of <class '__main__.Dog6'>>
    Dog6: ('class method ....', 'animal')


##### 自定义元类
对于创建类，Python做了如下的操作：

- 创建的类中有__metaclass__这个属性吗？如果是，Python会通过__metaclass__创建类(对象)
- 如果Python没有找到__metaclass__，它会继续在父类中寻找__metaclass__属性，并尝试做和前面同样的操作。
- 如果Python在任何父类中都找不到__metaclass__，它就会在模块层次中去寻找__metaclass__，并尝试做同样的操作。
- 如果还是找不到__metaclass__,Python就会用内置的type来创建这个类对象。

所以，可以通过重新类的__metaclass__ 来自定义元类，以控制类的创建，实现自动地改变类，而无需通过class 再进行编码定义。
自定义元类通常需要继承type，并重写其__new__和__init__方法。



```python
## 下面是自定义元类的一个例子：
class MyMeta(type):  
    ## __new__ 方法需要返回一个对象，以完成对象的创建
    ## 在__init__之前被调用
    def __new__(cls, name, bases, attrs):  
        print(f"Creating class {name}")  
        # 在这里可以修改或添加属性  
        #print(attrs)
        attrs['additional_attribute'] = 'This is an additional attribute added by the metaclass.'  
        print("=======>print attrs ")
        for name,value in attrs.items():
            if not name.startswith("__"):
                print(f"{name}:{value}")
        print("=======>print attrs ")
        return super().__new__(cls, name, bases, attrs)  
  
    ## 执行具体的初始化操作，如：继承的基类，要添加的属性等
    def __init__(cls, name, bases, attrs):  
        print(f"Initializing class {name}")  
        # 在这里可以进行一些初始化操作  
        super().__init__(name, bases, attrs)  

class MyClass(metaclass=MyMeta):  
    MyClassAttr = "MyClassAttr"
    def __init__(self):  
        print("Initializing instance of MyClass")
        self.member = "member"
    def print_member(self):
        return (f"member is {self.member}")

print('MyClass.__class_:', MyClass.__class__)

print("-------------")
# 创建MyClass的实例  
instance1 = MyClass()
  
# 访问由元类添加的额外属性  
print(instance1.additional_attribute)
print(instance1.MyClassAttr)
print(instance1.print_member())

print("-------------")
instance2 = MyClass()  
  
# 访问由元类添加的额外属性  
print(instance2.additional_attribute)
print(instance2.MyClassAttr)
print(instance2.print_member())
```

    Creating class MyClass
    =======>print attrs 
    MyClassAttr:MyClassAttr
    print_member:<function MyClass.print_member at 0x7fe02bee12d0>
    additional_attribute:This is an additional attribute added by the metaclass.
    =======>print attrs 
    Initializing class MyClass
    MyClass.__class_: <class '__main__.MyMeta'>
    -------------
    Initializing instance of MyClass
    This is an additional attribute added by the metaclass.
    MyClassAttr
    member is member
    -------------
    Initializing instance of MyClass
    This is an additional attribute added by the metaclass.
    MyClassAttr
    member is member


从结果可以看到，类MyClass 指定了创建自身的元类为MyMeta， 定义class类(创建元类MyMeta的对象)的时候，先执行MyMeta的__new__方法创建类MyClass(元类对象)，并添加类属性additional_attribute，接着执行MyMeta的__init__方法进行初始化，这个过程只执行一次。
接着创建 MyClass对象， instance1， instance2 ，都使用MyClass的__init__函数进行初始化。

##### __bases__ 与 __call__
- __call__是一个特殊方法，也称为“魔术方法”或“双下划线方法”。当一个对象被当作函数调用时，Python会自动调用该对象的__call__方法。这允许我们为任何对象定义调用行为。

- __bases__是一个类属性，它表示一个类的所有直接基类的元组。这个属性是只读的，意味着你不能直接修改它。__bases__是Python的内建属性，用于提供关于类继承结构的信息。


```python
# 先看__bases___的例子
class Base1:  
    pass  
  
class Base2:  
    pass  
  
class Derived(Base1, Base2):  
    pass  
  
# 查看Derived类的所有直接基类  
print(Derived.__bases__)  # 输出: (<class '__main__.Base1'>, <class '__main__.Base2'>)  
  
# 查看Base1类的所有直接基类  
print(Base1.__bases__)  # 输出: (<class 'object'>,)  
  
# 查看没有继承其他类的类的所有直接基类  
class NoBase:  
    pass  
  
print(NoBase.__bases__)  # 输出: (<class 'object'>,)
```

    (<class '__main__.Base1'>, <class '__main__.Base2'>)
    (<class 'object'>,)
    (<class 'object'>,)


在这个例子中，Derived类继承自Base1和Base2类，所以Derived.__bases__返回了一个包含Base1和Base2的元组。Base1和Base2类都继承自object类，所以它们的__bases__属性都包含object。而NoBase类没有继承其他类，因此它的__bases__只包含object。

需要注意的是，__bases__属性提供的是直接基类的信息，不包括间接基类。


```python
# 再看__call__的例子
class CallableObject:  
    def __call__(self, *args, **kwargs):  
        print("Callable object was called with arguments:", args, kwargs)  
# 创建一个CallableObject的实例  
callable_instance = CallableObject()  
  
# 使用这个实例像函数一样进行调用  
callable_instance(1, x=2)
```

    Callable object was called with arguments: (1,) {'x': 2}


总结：
- 在Python中，__new__是一个静态方法，用于创建并返回实例对象。它通常是由类的元类来调用，并且是在__init__方法之前执行的。__new__方法负责分配内存给新创建的对象，并返回该对象的实例。
- 
- __call__是一个特殊方法，也称为“魔术方法”或“双下划线方法”。当一个对象被当作函数调用时，Python会自动调用该对象的__call__方法。这允许我们为任何对象定义调用行为。
- __bases__是一个类属性，它表示一个类的所有直接基类的元组。这个属性是只读的，意味着你不能直接修改它。__bases__是Python的内建属性，用于提供关于类继承结构的信息。

