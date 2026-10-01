# 构造函数\_\_init\_\_

### `init` 到底是什么？

当你定义一个类：

```Python
class Student:
    pass
```

然后创建对象：

```Python
s1 = Student()
```

Python 会创建一个 `Student` 对象。

如果这个类里面写了：

```Python
class Student:
    def __init__(self):
        print("创建了一个学生对象")
```

那么执行：

```Python
s1 = Student()
```

Python 会自动调用：

```Python
__init__()
```

于是输出：

```Plain Text
创建了一个学生对象
```

也就是说，你不需要自己写：

```Python
s1.__init__()
```

创建对象的时候，Python 会自动帮你调用它。

---

## 为什么需要 `init`？

因为我们创建对象的时候，通常希望这个对象一出生就带有一些数据。

例如“学生”应该有：

```Plain Text
姓名
年龄
专业
```

我们就可以写：

```Python
class Student:
    def __init__(self, name, age, major):
        self.name = name
        self.age = age
        self.major = major
```

然后：

```Python
s1 = Student("Tom", 20, "Computer Science")
```

这句话执行的时候，可以大致理解成 Python 自动做了：

```Python
Student.__init__(s1, "Tom", 20, "Computer Science")
```

最终 `s1` 里面就有：

```Python
s1.name
s1.age
s1.major
```

比如：

```Python
print(s1.name)
print(s1.age)
print(s1.major)
```

输出：

```Plain Text
Tom
20
Computer Science
```

所以你说的：

> 我们可以把一些属性放到 `init` 方法里面，方便创建对象
> 
> 

这个理解基本正确。

更准确一点应该说：

> 我们通常在 `init` 中给新创建的对象设置初始属性。
> 
> 

---

## 最重要的是理解 `self`

比如：

```Python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

这里很多初学者会被：

```Python
self.name = name
```

搞晕。

其实左右两个 `name` 不是一个东西。

左边：

```Python
self.name
```

代表：

> 当前这个对象自己的 `name` 属性。
> 
> 

右边：

```Python
name
```

代表：

> 创建对象的时候传进来的参数。
> 
> 

所以：

```Python
self.name = name
```

可以翻译成：

> 把传进来的 `name` 保存到这个对象自己的 `name` 属性里面。
> 
> 

比如：

```Python
s1 = Student("Tom", 20)
```

那么实际上：

```Python
name = "Tom"
age = 20
```

然后：

```Python
self.name = "Tom"
self.age = 20
```

因为此时 `self` 代表 `s1`，所以可以进一步理解成：

```Python
s1.name = "Tom"
s1.age = 20
```

---

## `self` 到底是谁？

假设：

```Python
class Student:
    def __init__(self, name):
        self.name = name
```

创建两个对象：

```Python
s1 = Student("Tom")
s2 = Student("Jerry")
```

创建 `s1` 时：

```Python
self
```

代表：

```Python
s1
```

创建 `s2` 时：

```Python
self
```

代表：

```Python
s2
```

所以：

```Python
print(s1.name)
print(s2.name)
```

得到：

```Plain Text
Tom
Jerry
```

虽然它们都是 `Student` 类创建出来的，但每个对象可以保存属于自己的属性。

你可以把：

```Python
self
```

理解成一句话：

> **当前正在操作的这个对象。**
> 
> 

---

## 类和对象之间是什么关系？

可以把“类”想象成一个模板。

比如：

```Python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

`Student` 是模板。

然后：

```Python
s1 = Student("Tom", 20)
s2 = Student("Jerry", 21)
s3 = Student("Alice", 19)
```

根据同一个模板，可以创建很多对象：

```Plain Text
Student 类
   │
   ├── s1
   │    ├── name = "Tom"
   │    └── age = 20
   │
   ├── s2
   │    ├── name = "Jerry"
   │    └── age = 21
   │
   └── s3
        ├── name = "Alice"
        └── age = 19
```

这就是面向对象很核心的思想。

---

## `init` 还可以设置默认值

例如游戏角色：

```Python
class Player:
    def __init__(self, name):
        self.name = name
        self.hp = 100
        self.level = 1
```

创建：

```Python
p1 = Player("Xander")
```

虽然只传了：

```Python
"Xander"
```

但是这个对象一创建，就已经拥有：

```Python
p1.name   # Xander
p1.hp     # 100
p1.level  # 1
```

因为我们在 `init` 中规定：

```Python
self.hp = 100
self.level = 1
```

所以可以把 `init` 想象成：

> **对象的初始化说明书。**
> 
> 

一个 Player 创建出来以后：

```Plain Text
名字 = 用户提供
生命值 = 100
等级 = 1
```

---

## 属性和普通变量有什么区别？

比如：

```Python
class Student:
    def __init__(self, name):
        x = 100
        self.name = name
```

这里：

```Python
x = 100
```

只是 `init` 函数中的局部变量。

函数执行结束之后，我们不能这样访问：

```Python
s1.x
```

但是：

```Python
self.name = name
```

是给对象添加属性。

所以：

```Python
s1.name
```

可以访问。

因此要区分：

```Python
name = "Tom"
```

普通变量。

而：

```Python
self.name = "Tom"
```

对象属性。

---

## `init` 不仅能创建属性，也可以执行其他初始化逻辑

例如：

```Python
class BankAccount:
    def __init__(self, owner, money):
        self.owner = owner
        self.money = money

        print("账户创建成功")
```

执行：

```Python
account = BankAccount("Tom", 1000)
```

除了创建：

```Python
account.owner
account.money
```

还会输出：

```Plain Text
账户创建成功
```

所以 `init` 本质上是一个方法，只不过它比较特殊：

> 创建对象之后，Python 会自动调用它来完成对象的初始化。
> 
> 

---

## 一个完整一点的例子

```Python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def introduce(self):
        print(f"我叫{self.name}，今年{self.age}岁")


s1 = Student("Tom", 20)
s2 = Student("Jerry", 21)

s1.introduce()
s2.introduce()
```

输出：

```Plain Text
我叫Tom，今年20岁
我叫Jerry，今年21岁
```

这里你可以看到：

`init` 负责：

```Python
self.name = name
self.age = age
```

也就是**初始化对象的数据**。

而：

```Python
introduce()
```

负责这个对象可以做什么。

所以类里面通常有两类东西：

```Plain Text
属性 → 对象有什么
方法 → 对象能做什么
```

例如：

```Python
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def bark(self):
        print("汪汪")
```

那么：

```Plain Text
name、age → 属性
bark()    → 方法
```

---

### 你现阶段可以这样记

```Python
class 类名:
    def __init__(self, 参数):
        self.属性 = 参数
```

例如：

```Python
class Car:
    def __init__(self, brand, price):
        self.brand = brand
        self.price = price
```

创建：

```Python
car1 = Car("BMW", 300000)
```

就相当于给 `car1` 初始化：

```Plain Text
brand = BMW
price = 300000
```

所以一句话总结：

> **`init`**** 是对象的初始化方法。当我们通过 ****`类名()`**** 创建对象时，Python 会自动调用 ****`init`****，通常用它给新对象设置初始属性。**
> 
> 

另外严格来说，很多教材把 `init` 直接叫“构造函数”，方便理解；但 Python 更严谨的说法是它是**初始化方法**，真正负责创建对象的是 `new`。现阶段你完全可以先把 `init` 理解成“构造函数”，等学深入了再区分 `new` 和 `init`。

