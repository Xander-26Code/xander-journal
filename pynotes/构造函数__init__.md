# Python `__init__` 详解：从 `self` 到对象初始化的常见陷阱

学习 Python 类的时候，很多人都能照着写出这段代码：

```python
class Student:
    def __init__(self, name):
        self.name = name
```

但一旦追问：“为什么要写 `self`？”“等号两边的 `name` 有什么区别？”“`__init__` 返回的对象去了哪里？”理解就容易变得模糊。

本文沿着创建一个学生对象的过程，解释 `__init__`、实例属性、默认参数和继承初始化，最后整理一个可以直接运行的学生选课示例。

先明确术语：不少教程把 `__init__` 叫作“构造函数”，但更精确地说，它是**初始化方法**。普通对象创建流程中，`__new__` 负责创建实例，`__init__` 负责设置实例的初始状态。这一区分来自 [Python 官方数据模型](https://docs.python.org/3/reference/datamodel.html#object.__init__)。

## 1. 从 `Student()` 到一个可用的对象

一个最简单的类可以没有任何方法：

```python
class Student:
    pass


student = Student()
print(isinstance(student, Student))  # True
```

`Student` 是类，`student` 这个变量引用了一个 `Student` 实例。不自己定义 `__init__`，也可以创建对象。

不过，我们通常希望学生对象创建完成后就有姓名、年龄和专业，而不是每次都在外面手动补属性。这时就需要初始化方法：

```python
class Student:
    def __init__(self, name, age, major):
        self.name = name
        self.age = age
        self.major = major


student = Student("Tom", 20, "Computer Science")

print(student.name)
print(student.age)
print(student.major)
```

输出：

```text
Tom
20
Computer Science
```

对这样的普通类，`Student(...)` 会先创建实例，再自动调用初始化方法，最后把初始化好的实例交给调用方。你不需要在创建之后再补一句 `student.__init__(...)`。

而且，执行初始化方法时，右侧的 `Student(...)` 还没有返回，左侧变量 `student` 也尚未完成这次赋值。**`self` 能访问新实例，并不是因为它提前知道了外面的变量名。**

## 2. `self` 指的是谁？

`self` 是实例方法的第一个参数，表示当前操作的实例。调用绑定的实例方法时，Python 会自动传入这个实例。

例如：

```python
class Student:
    def __init__(self, name):
        self.name = name

    def introduce(self):
        return f"我叫 {self.name}"


tom = Student("Tom")
jerry = Student("Jerry")

print(tom.introduce())
print(jerry.introduce())
```

输出：

```text
我叫 Tom
我叫 Jerry
```

执行 `tom.introduce()` 时，`self` 指向 `tom` 引用的实例；执行 `jerry.introduce()` 时，`self` 指向另一个实例。就参数传递而言，这个普通实例方法的调用可以理解为：

```python
# 接着上面的定义执行
print(Student.introduce(tom))  # 我叫 Tom
```

同理，初始化新实例时，Python 会把它作为 `__init__` 的第一个参数，再传入 `"Tom"` 等参数。类调用还涉及创建实例，因此不能把整个 `Student("Tom")` 简化成只有一次普通的 `__init__` 调用。

`self` 不是关键字，技术上可以换名字，但这是 Python 社区稳定的命名约定，实际代码应当遵守。实例方法的绑定规则可参阅 [官方类与方法教程](https://docs.python.org/3/tutorial/classes.html#method-objects)。

## 3. `self.name = name` 为什么不是重复赋值？

这一行的左右两边属于不同的位置：

| 表达式 | 含义 |
| --- | --- |
| `name` | 传入初始化方法的局部参数 |
| `self.name` | 当前实例上的 `name` 属性 |
| `self.name = name` | 将参数引用的值保存到实例属性中 |

左右同名只是为了阅读方便，完全可以写成：

```python
class Student:
    def __init__(self, student_name):
        self.name = student_name
```

这里仍然创建了名为 `name` 的实例属性。

再看局部变量与属性的区别：

```python
class Student:
    def __init__(self, name):
        normalized_name = name.strip()
        self.name = normalized_name


student = Student("  Tom  ")
print(student.name)                        # Tom
print(hasattr(student, "normalized_name"))  # False
```

`normalized_name` 是这次调用中的局部变量，没有自动成为对象的属性。若写 `student.normalized_name`，就会得到 `AttributeError`。

还要注意，赋值通常保存的是对象引用，并不自动复制数据。传入列表后直接写 `self.courses = courses`，调用方与实例可能持有同一个列表；如果希望拥有独立列表，需要主动复制。

## 4. 初始状态可以来自参数，也可以由类设定

并不是所有属性都必须从外部传入。例如游戏角色需要名字，但生命值和等级可以使用初始设定：

```python
class Player:
    def __init__(self, name, hp=100):
        self.name = name
        self.hp = hp
        self.level = 1


normal = Player("Xander")
boss = Player("Boss", hp=500)

print(normal.name, normal.hp, normal.level)  # Xander 100 1
print(boss.name, boss.hp, boss.level)        # Boss 500 1
```

这里有三种来源：名字必须传入，生命值有可覆盖的默认值，等级固定从 `1` 开始。

初始化方法也适合检查对象成立所需的基本条件。例如要求年龄非负，可以在赋值前检查，遇到不合法的值就抛出异常。这样调用方拿到对象时，它已经具备约定的初始状态。

不过，`__init__` 中的校验只约束这次初始化；如果属性仍然公开可写，后续直接执行 `student.age = -1` 不会自动重新触发校验。需要持续维护约束时，还要设计属性访问或更新方法。

## 5. 实例属性与类属性：为什么两个学生共享了课程？

下面是一个很常见的错误：

```python
# 错误示例：把每个学生自己的课程放成了共享类属性
class Student:
    courses = []

    def __init__(self, name):
        self.name = name


tom = Student("Tom")
jerry = Student("Jerry")

tom.courses.append("Python")
print(jerry.courses)  # ['Python']
```

`courses` 定义在类体里，是类属性。两个实例都没有自己的同名属性，读取时都找到同一个类属性列表；`append()` 修改的也是这个共享列表。

如果课程属于每个学生自己，就应当在初始化时分别创建：

```python
class Student:
    school = "Python 学院"  # 适合由实例共同查询的类属性

    def __init__(self, name):
        self.name = name
        self.courses = []  # 每次初始化都会创建一个新列表


tom = Student("Tom")
jerry = Student("Jerry")

tom.courses.append("Python")
print(tom.courses)    # ['Python']
print(jerry.courses)  # []
print(tom.school)     # Python 学院
```

类属性并不是不能使用，关键是数据应当共享还是独立。对于这些普通类，`tom.school = "另一所学校"` 会创建同名实例属性，遮蔽类属性，并不直接修改 `Student.school`。这些区别见 [官方类变量与实例变量说明](https://docs.python.org/3/tutorial/classes.html#class-and-instance-variables)。

## 6. 放进 `__init__` 就安全了？还要避开可变默认参数

下面的写法虽然有 `self.courses`，仍然会意外共享列表：

```python
# 错误示例：多个默认调用复用同一个列表
class Student:
    def __init__(self, name, courses=[]):
        self.name = name
        self.courses = courses


tom = Student("Tom")
jerry = Student("Jerry")

tom.courses.append("Python")
print(jerry.courses)  # ['Python']
```

原因是默认参数表达式在函数定义时求值，不是每次调用时重新求值。上面的两个实例属性最终指向同一个默认列表。这个规则也适用于普通函数，详见 [官方默认参数说明](https://docs.python.org/3/tutorial/controlflow.html#default-argument-values)。

常用的改法是用 `None` 表示“没有提供课程”：

```python
class Student:
    def __init__(self, name, courses=None):
        self.name = name
        self.courses = [] if courses is None else list(courses)


initial_courses = ["Python"]
tom = Student("Tom", initial_courses)
jerry = Student("Jerry")

initial_courses.append("SQL")
print(tom.courses)    # ['Python']
print(jerry.courses)  # []
```

这里的 `list(courses)` 还复制了传入的列表，避免外部追加课程直接影响实例。它是浅复制：如果元素本身是列表或字典，内层可变对象仍可能共享。本例的课程名都是字符串，这样处理就足够。

## 7. `__init__` 和 `__new__`：创建与初始化的边界

理解日常代码时，可以先用这个简化流程：

```text
调用 Student(...)
       ↓
__new__ 创建并返回实例
       ↓
__init__ 初始化这个实例
       ↓
类调用返回实例，赋值给外部变量
```

下面的例子只用于观察顺序，普通业务类通常不需要重写 `__new__`：

```python
class Student:
    def __new__(cls, name):
        print("1. 创建实例")
        return super().__new__(cls)

    def __init__(self, name):
        print("2. 初始化实例")
        self.name = name


student = Student("Tom")
print("3. 获得对象：", student.name)
```

输出：

```text
1. 创建实例
2. 初始化实例
3. 获得对象： Tom
```

`cls` 表示类，`self` 表示实例。两者参与的阶段不同。

严格来说，自动调用 `__init__` 还有前提：在通常的类调用流程中，`__new__` 返回了所请求类的实例，才会继续相应的初始化。如果 `__new__` 返回其他类型的对象，就不会按这个流程调用该类的 `__init__`。完整规则见 [官方 `__new__` 说明](https://docs.python.org/3/reference/datamodel.html#object.__new__)。

### 为什么不能在 `__init__` 里 `return self`？

因为 `__init__` 的职责是初始化已有实例，它必须返回 `None`。不写 `return` 就会隐式返回 `None`。

下面是错误示例：

```python
class BrokenStudent:
    def __init__(self, name):
        self.name = name
        return self


BrokenStudent("Tom")  # TypeError：__init__ 应返回 None
```

`student = Student(...)` 中获得的实例，是类调用流程返回的，不是由 `__init__` 返回的。需要根据不同输入构造对象时，可以另外设计工厂函数或类方法，不要通过改变 `__init__` 的返回值实现。

## 8. 继承之后，父类会自动初始化吗？

如果子类没有定义自己的 `__init__`，可以通过继承使用父类的初始化方法。

但子类一旦覆盖 `__init__`，Python 不会自动替你把父类的初始化方法再调用一遍。如果仍然需要父类的初始化逻辑，就要明确调用：

```python
class Person:
    def __init__(self, name):
        self.name = name


class Student(Person):
    def __init__(self, name, student_id):
        super().__init__(name)
        self.student_id = student_id


student = Student("Tom", "S001")
print(student.name)        # Tom
print(student.student_id)  # S001
```

在这个单继承例子里，`super().__init__(name)` 调用 `Person` 的初始化逻辑，并作用于同一个实例。如果删掉这一行，`student_id` 会被设置，但 `name` 不会自动出现。

`super()` 的一般规则是沿方法解析顺序寻找下一个实现；在多继承里，不能简单理解为“永远调用写在旁边的某一个父类”。需要深入时可阅读 [官方 `super()` 文档](https://docs.python.org/3/library/functions.html#super)。

## 9. 完整示例：创建一个能选课的学生对象

把前面的知识放到一起。下面的示例包含名字处理、年龄校验、独立的课程列表，以及对象创建后的行为。可以保存为 `student_demo.py` 直接运行：

```python
class Student:
    def __init__(self, name, age, courses=None):
        if not isinstance(name, str):
            raise TypeError("姓名必须是字符串")
        name = name.strip()
        if not name:
            raise ValueError("姓名不能为空")

        # bool 是 int 的子类，这里明确不接受 True / False 作为年龄。
        if isinstance(age, bool) or not isinstance(age, int):
            raise TypeError("年龄必须是整数")
        if age < 0:
            raise ValueError("年龄不能为负数")

        self.name = name
        self.age = age
        self.courses = []

        if courses is not None:
            for course in courses:
                self.enroll(course)

    def enroll(self, course):
        if not isinstance(course, str):
            raise TypeError("课程名必须是字符串")
        course = course.strip()
        if not course:
            raise ValueError("课程名不能为空")
        if course not in self.courses:
            self.courses.append(course)

    def introduce(self):
        courses_text = "、".join(self.courses) or "暂无"
        return f"我叫 {self.name}，今年 {self.age} 岁，已选课程：{courses_text}"


def main():
    tom = Student("  Tom  ", 20, ["Python"])
    jerry = Student("Jerry", 21)

    tom.enroll("SQL")
    tom.enroll("Python")

    print(tom.introduce())
    print(jerry.introduce())


if __name__ == "__main__":
    main()
```

输出：

```text
我叫 Tom，今年 20 岁，已选课程：Python、SQL
我叫 Jerry，今年 21 岁，已选课程：暂无
```

`courses` 在这里约定为课程名的可迭代集合，例如列表或元组；只有一门课程时也应传 `["Python"]`，而不是裸字符串，否则迭代得到的会是单个字符。

初始化方法确保对象拥有可用的初始数据，`enroll()` 负责后续选课行为，`introduce()` 根据当前状态生成介绍。每个实例都拥有独立的课程列表，重复选课也不会重复记录。

这个例子还在初始化过程中复用了 `enroll()`。对于这类简单类很直观；如果把它作为可继承的基类设计，则要留意子类可能覆盖该方法，而此时子类自己的属性可能还没初始化完。

## 10. 写 `__init__` 时，优先检查这五件事

- **拼写是否正确**：自动初始化识别的是 `__init__`；`init` 和 `_init_` 都只是其他名字。
- **实例是否拿到了自己的状态**：该独立的数据应在实例上保存，避免意外共享可变类属性。
- **默认参数是否安全**：列表、字典、集合等可变对象不要随意作为默认参数。
- **继承链是否得到正确初始化**：覆盖初始化方法后，需要的上级初始化逻辑应明确调用。
- **是否把过多工作塞进了初始化**：属性赋值和必要校验很合适；耗时下载、启动后台任务等操作，通常值得设计成显式方法。

此外，手动调用 `student.__init__(...)` 不会创建另一个对象，只会再次对当前实例执行初始化逻辑，可能覆盖已有状态。需要重置或更新时，使用有明确语义的方法更容易理解。

当你再看到 `self.name = name`，可以沿着一条清晰的线理解它：调用方提供参数，Python 把当前实例传给 `self`，初始化方法把数据保存到实例上，后续方法再通过这个实例访问和修改状态。理解这条线，比背下一个类定义模板更有用。
