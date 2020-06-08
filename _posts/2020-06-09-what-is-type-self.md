
---
layout: post
title:  如何理解Swift中的Any、AnyObject、X.self、X.Type、AnyClass、Self
date: 2020-06-09 01:25:00.000000000 +08:00
---

# Any、AnyObjectW

Any: 类似于OC中的instanceType，可以代表任意类型
 AnyObject: 代表任意 类 类型，像枚举、结构体，都不行

# is as

is：判断是否是某种类型（既可以是类与类之间的判断，也可以是类与协议之间的判断）
 as：做类型强制转换



```dart
protocol testProtocol {}
class Animal {}
class Person: testProtocol, Animal {}
var p = Person()
print(p is Animal) // true
print(p is testProtocol) // true
ptint(p is Person) // true

var num: Any = 10;
print(num is Int) // true
```

# X.self、X.Type、AnyClass

X.self: 是一个元类型（metadata）的指针，存放着类型相关的信息
 x.self属于X.type类型

先来说下X.self ：



```csharp
class Person {}
var P = Person()
```

其内存分配如下：



![img](/assets/resources/1754678-9875c5282002aae9.png)

image.png

还有一点在swift中不像OC一样需要继承一个通用的基类NSObject，在swift中如果不继承任何类的话，自己就是基类

> **Swift classes do not inherit from a universal base class. Classes you define without specifying a superclass automatically become base classes for you to build upon.”** 摘录来自: Apple Inc. “The Swift Programming Language (Swift 5.0)。” Apple Books. [https://books.apple.com/us/book/the-swift-programming-language-swift-5-0/id881256329](https://links.jianshu.com/go?to=https%3A%2F%2Fbooks.apple.com%2Fus%2Fbook%2Fthe-swift-programming-language-swift-5-0%2Fid881256329)

但是在swift中有个隐藏的基类，看如下代码：



```swift
class Person{}
// 输出：Optional(Swift._SwiftObject)
print(class_getSuperclass(Person.self))
```

可以看出就算没有继承任何类，他还是有父类的。所有的swift类都有个隐藏的基类：Swift._SwiftObject
 Swift._SwiftObject文件在 swift源码可以找到：[https://github.com/apple/swift/blob/master/stdlib/public/runtime/SwiftObject.h](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fapple%2Fswift%2Fblob%2Fmaster%2Fstdlib%2Fpublic%2Fruntime%2FSwiftObject.h)

![img](/assets/resources/1754678-047e62c8c8cdafd6.png)

image.png



大致搞懂了X.self是什么东西，再来说下X.Type
 X.self 也是有自己的类型的，就是X.Type，就是元类型的类型



```swift
class Person {}
class Student : Person {}
var pType = Person.self // pType的类型就是Person.Type
var stuType: Studentp.Type  = Student.self // stuType的类型就是 Studentp.Type
pType = Student.self // 因为子类，所有可以这样写
```

也可以定义一个接收任意类型的变量 `AnyObject.Type`



```php
var anyType: AnyObject.Type = Perosn.self
anyType = Student.self
```

AnyClass：就是 AnyObject.Type的别名
 `public typealias AnyClass = AnyObject.Type`
 ok现在AnyObject 也搞清楚了

## 应用

这个东西有什么用呢？举个简单的例子



```swift
class Animal: NSObject {
    // 必须是 required 保证子类调用init的时候初始化成功
    required override init() {
        super.init()
    }

}
class Dog: Animal { }

class Cat: Animal { }

class Pig: Animal { }

func create(_ classes:[Animal.Type]) -> [Animal] {
        var arr = [Animal]()
        for cls in classes {
            arr.append(cls.init())
        }
        return arr
}
create([Dog.self, Cat.self, Pig.self])
```

# Self

## 1、Self代表当前类型

可以直接理解为 Self是当前类的代称。

也就是说可以理解为 `typealias Self = ClassName` ， 下面的例子中可以理解为`typealias Self = Person`

```swift
class Person {
    var age = 10
    static var count = 2
    func test() {
        print(self.age)
        // 访问count的时候，一般通过类名直接访问
        print(Person.count)
        // 也可以使用 Self 访问
        print(Self.count)
    }
}
```

## 2、一般作为函数的返回值使用，限定返回值跟调用者必须类型一致



```swift
protocol Runable {
    func test() -> Self
}

class Person : Runable {
    required init() { }
    func test() -> Self {
        return type(of: self).init()
    }
}

class Student: Person { }

let p = Person().test()
let s = Student().test()
// 输出：swift___闭包.Person
print(p)
// 输出：swift___闭包.Student    
rint(s)
```

# Protocol

很多人对 Protocol 的元类型容易理解错。Protocol 自身不是一个类型，只有当一个对象实现了 protocol 后才有了类型对象。所以 Protocol.self 不等于 Protocol.Type。如果你写下面的代码会得到一个错误：

```swift
protocol MyProtocol { }
let metatype: MyProtocol.Type = MyProtocol.self
```

正确的理解是 MyProtocol.Type 也是一个有效的元类型，那么就需要是一个可承载的类型的元类型。所以改成这样就可以了：

```swift
struct MyType: MyProtocol { }
let metatype: MyProtocol.Type = MyType.self 
```

那么 Protocol.self 是什么类型呢？为了满足你的好奇心苹果为你造了一个类型：

```swift
let protMetatype: MyProtocol.Protocol = MyProtocol.self
```

# 一个实战

为了让大家能够熟悉元类型的使用我举一个例子。 假设我们有两个 Cell 类，想要一个工厂方法可以根据类型初始化对象。下面是两个 Cell 类：

```swift
protocol ContentCell { }

class IntCell: UIView, ContentCell {
    required init(value: Int) {
        super.init(frame: CGRect.zero)
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

class StringCell: UIView, ContentCell {
    required init(value: String) {
        super.init(frame: CGRect.zero)
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

工厂方法的实现是这样的：

```swift
func createCell(type: ContentCell.Type) -> ContentCell? {
    if let intCell = type as? IntCell.Type {
        return intCell.init(value: 5)
    } else if let stringCell = type as? StringCell.Type {
        return stringCell.init(value: "xx")
    }
    return nil
}

let intCell = createCell(type: IntCell.self)
```

当然我们也可以使用类型推断，再结合泛型来使用：

```swift
func createCell<T: ContentCell>() -> T? {
    if let intCell = T.self as? IntCell.Type {
        return intCell.init(value: 5) as? T
    } else if let stringCell = T.self as? StringCell.Type {
        return stringCell.init(value: "xx") as? T
    }
    return nil
}

// 现在就根据返回类型推断需要使用的元类型
let stringCell: StringCell? = createCell()
```

在 [Reusable](https://github.com/AliSoftware/Reusable) 中的 tableView 的 dequeue 采用了类似的实现：

```swift
func dequeueReusableCell<T: UITableViewCell>(for indexPath: IndexPath, cellType: T.Type = T.self) -> T
    where T: Reusable {
      guard let cell = self.dequeueReusableCell(withIdentifier: cellType.reuseIdentifier, for: indexPath) as? T else {
        fatalError("Failed to dequeue a cell")
      }
      return cell
  }
```

dequeue 的时候就可以根据目标类型推断，不需要再额外声明元类型：

```swift
 class MyCustomCell: UITableViewCell, Reusable 
tableView.register(cellType: MyCustomCell.self)

let cell: MyCustomCell = tableView.dequeueReusableCell(for: indexPath)
```

# Reference

- [Whats Type And Self Swift Metatypes](https://swiftrocks.com/whats-type-and-self-swift-metatypes.html)
- [ANYCLASS，元类型和 .SELF](https://swifter.tips/self-anyclass/)