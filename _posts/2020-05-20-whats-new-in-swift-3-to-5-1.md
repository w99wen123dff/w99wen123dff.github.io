---
layout: post
title: Swift 3到5.1新特性整理
date: 2020-05-20 23:25:00.000000000 +08:00
---

# Swift 3到5.1新特性整理

> 原文链接： [https://hicc.me/whats-new-in-swift-3-to-5-1/](https://hicc.me/whats-new-in-swift-3-to-5-1/)



![](https://cdn.nlark.com/yuque/0/2020/jpeg/1458794/1589986255815-e4889e0e-5de3-4e45-88ec-4103c753bc4e.jpeg#align=left&display=inline&height=832&margin=%5Bobject%20Object%5D&originHeight=1079&originWidth=1349&size=0&status=done&style=none&width=1040)
[Hipo 2.0 重写](https://hicc.me/hipo-2/)从Swift 1的版本写到2的版本，后续Hipo功能稳定，更新慢了很多……，Swift本身却在长足的发展，5.0都已经发布了，本文对Swift 3.0 到Swift 5.1 的更新点做个总结。
为了方便阅读，准备从新到旧的总结。
下面所有的东西，都是来自[hackingwithswift.com](https://www.hackingwithswift.com/articles/126/whats-new-in-swift-5-0)。

---

## Swift 5.1
Swift 5.1的更新比较迟，[单独成篇Swift 5.1的变化](https://hicc.me/whats-new-in-swift-5-1/)。
## Swift 5.0
Swift 5.0 最重要的自然是[ABI Stability](https://swift.org/blog/abi-stability-and-more/)， 对此可以看这篇 [Swift ABI 稳定对我们到底意味着什么](https://onevcat.com/2019/02/swift-abi/) 。
当然还有其他的更新。
### `Result`类型
[SE-0235](https://github.com/apple/swift-evolution/blob/master/proposals/0235-add-result.md)提议的实现。用来在复杂对象中的错误处理。
`Result`类型有两个带泛型的枚举成员`success`和`failure`，而且`failure`的泛型必须遵循Swift的`Error`类型。
常规的使用
```swift
enum NetworkError: Error {
    case badURL
}
import Foundation
func fetchUnreadCount1(from urlString: String, completionHandler: @escaping (Result<Int, NetworkError>) -> Void)  {
    guard let url = URL(string: urlString) else {
        completionHandler(.failure(.badURL))
        return
    }
    // complicated networking code here
    print("Fetching \(url.absoluteString)...")
    completionHandler(.success(5))
}
fetchUnreadCount1(from: "https://www.hackingwithswift.com") { result in
    switch result {
    case .success(let count):
        print("\(count) unread messages.")
    case .failure(let error):
        print(error.localizedDescription)
    }
}
```
首先，`Result`有个`get()`方法，要么返回成功值，要么抛出错误。那么可以这么使用。
```swift
fetchUnreadCount1(from: "https://www.hackingwithswift.com") { result in
    if let count = try? result.get() {
        print("\(count) unread messages.")
    }
}
```
再次，`Result`可以接受一个闭包来初始化，如果闭包成功返回，就会把它放到`success`的一边，如果抛出错误，就放到`failure`的一边。
```swift
let result = Result { try String(contentsOfFile: someFile) }
```
最后，你可以使用你自己的错误枚举，但是Swift官方建议，你说用`Swift.Error`来作为`Error`的参数。
> “it’s expected that most uses of Result will use Swift.Error as the Error type argument.”

### Raw string
[SE-0200](https://github.com/apple/swift-evolution/blob/master/proposals/0200-raw-string-escaping.md) 引入了，使用`#`来包裹的Raw字符串，里面的字符不会做处理，特别是一些转义字符。
差值需要这样做
```swift
let answer = 42
let dontpanic = #"The answer to life, the universe, and everything is \#(answer)."#
```
这个对于正则的特别好用
```swift
let regex1 = "\\\\[A-Z]+[A-Za-z]+\\.[a-z]+"
let regex2 = #"\\[A-Z]+[A-Za-z]+\.[a-z]+"#
```
### 自定义字符串插值
[SE-0228](https://github.com/apple/swift-evolution/blob/master/proposals/0228-fix-expressiblebystringinterpolation.md)提案改进了Swift的字符串插值，让其更高效和自由。
```swift
struct User {
    var name: String
    var age: Int
}
extension String.StringInterpolation {
    mutating func appendInterpolation(_ value: User) {
        appendInterpolation("My name is \(value.name) and I'm \(value.age)")
    }
}
let user = User(name: "Guybrush Threepwood", age: 33)
print("User details: \(user)")
// User details: My name is Guybrush Threepwood and I'm 33,
```
// TODO: 更多使用，需要多研究
### 动态可调用类型
[SE-0216](https://github.com/apple/swift-evolution/blob/master/proposals/0216-dynamic-callable.md) 增加了`@dynamicCallable`属性，来支持方法的动态调用，类似`@dynamicMemberLookup`。
你可以将
```swift
struct RandomNumberGenerator {
    func generate(numberOfZeroes: Int) -> Double {
        let maximum = pow(10, Double(numberOfZeroes))
        return Double.random(in: 0...maximum)
    }
}
```
转变为
```swift
@dynamicCallable
struct RandomNumberGenerator {
    func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, Int>) -> Double {
        let numberOfZeroes = Double(args.first?.value ?? 0)
        let maximum = pow(10, numberOfZeroes)
        return Double.random(in: 0...maximum)
    }
}
let random = RandomNumberGenerator()
let result = random(numberOfZeroes: 0)
```

- `@dynamicCallable`参数
  - 无参数标签`withArguments`，你可以使用任何遵循`ExpressibleByArrayLiteral`的类型，例如 数组，数组切片，set等
  - 有参数标签的`withKeywordArguments`，使用任何遵循`ExpressibleByDictionaryLiteral`的类型，例如，字典，和key value 对，更多`KeyValuePairs`可以的看这里，[什么是KeyValuePairs？](https://www.hackingwithswift.com/example-code/language/what-are-keyvaluepairs)
- 你可以将其用在结构体，枚举，类和协议上
- 如果你使用`withKeywordArguments`而不是`withArguments`，你仍然按照无参数标签的方式使用，只是key是空字符串。
- 如果`withKeywordArguments`或者`withArguments`标记为抛出错误，调用类型也会抛出错误。
- 不能在扩展中使用`@dynamicCallable`
- 你仍然可以添加属性和方法。
### 处理未来的枚举值
[SE_0192](https://github.com/apple/swift-evolution/blob/master/proposals/0192-non-exhaustive-enums.md)的实现。
有时候枚举的switch中使用`default`来防治出错，但不会真正的使用，但是如果未来加了新的`case`，那些处理地方就会遗漏。现在可以添加`@unknkow`来出触发Xcode的提示。
```swift
func showNew(error: PasswordError) {
    switch error {
    case .short:
        print("Your password was too short.")
    case .obvious:
        print("Your password was too obvious.")
    @unknown default:
        print("Your password wasn't suitable.")
    }
}
```
这样，如果如果代码中，没有处理干净PasswordError (`switch` block is no longer exhaustive)，就会告警.
### 从`try?`抹平嵌套可选
```swift
struct User {
    var id: Int
    init?(id: Int) {
        if id < 1 {
            return nil
        }
        self.id = id
    }
    func getMessages() throws -> String {
        // complicated code here
        return "No messages"
    }
}
let user = User(id: 1)
let messages = try? user?.getMessages()
```
上面的例子中，Swift 4.2以及之前的，`message`会是 `String??`， 这样就不太合理，Swift 5中，就能返回抹平的`String?`
### 检查整数是否为偶数
[SE-0225](https://github.com/apple/swift-evolution/blob/master/proposals/0225-binaryinteger-iseven-isodd-ismultiple.md)添加了， `isMultiple(of:)`来检查整数是否为偶数， 和`if rowNumber % 2 == 0`效果一样。
```swift
let rowNumber = 4
if rowNumber.isMultiple(of: 2) {
    print("Even")
} else {
    print("Odd")
}
```
### 字典`compactMapValues()`方法
[SE-0218](https://github.com/apple/swift-evolution/blob/master/proposals/0218-introduce-compact-map-values.md)，为字典添加了`compactMapValues()`方法，这个就像结合了，数组`compactMap()`方法(遍历成员，判断可选的值，然后丢弃nil成员)和字典的`mapValues()`方法(只转换字典的value)。
```swift
let times = [
    "Hudson": "38",
    "Clarke": "42",
    "Robinson": "35",
    "Hartis": "DNF"
]
let finishers1 = times.compactMapValues { Int($0) }
let finishers2 = times.compactMapValues(Int.init)
let people6 = [
    "Paul": 38,
    "Sophie": 8,
    "Charlotte": 5,
    "William": nil
]
let knownAges = people6.compactMapValues { $0 }
print("compactMapValues, \(finishers1), \(finishers2)，\(knownAges)")
// compactMapValues, ["Clarke": 42, "Robinson": 35, "Hudson": 38], ["Robinson": 35, "Clarke": 42, "Hudson": 38]，["Charlotte": 5, "Sophie": 8, "Paul": 38]
```
### 撤回的功能： 带条件的计数
[SE-0220](https://github.com/apple/swift-evolution/blob/master/proposals/0220-count-where.md)， 引入了`count(where:)`函数，来计算遵循`Sequence`列表中满足条件成员的个数。
```swift
let scores = [100, 80, 85]
let passCount = scores.count { $0 >= 85 }
let pythons = ["Eric Idle", "Graham Chapman", "John Cleese", "Michael Palin", "Terry Gilliam", "Terry Jones"]
let terryCount = pythons.count { $0.hasPrefix("Terry") }
```
**这个功能因为性能问题，被撤回了。**
## Swift 4.2
### `CaseIterable`协议
[SE-0194](https://github.com/apple/swift-evolution/blob/master/proposals/0194-derived-collection-of-enum-cases.md)提议的实现，Swift4.2 增加了`CaseIterable`协议，能够给枚举的`allCases`属性自动产生所有的枚举的数组。
```swift
enum Pasta: CaseIterable {
    case cannelloni, fusilli, linguine, tagliatelle
}
for shape in Pasta.allCases {
    print("I like eating \(shape).")
}
```
当然还可以自行实现
```swift
enum Car: CaseIterable {
    static var allCases: [Car] {
        return [.ford, .toyota, .jaguar, .bmw, .porsche(convertible: false), .porsche(convertible: true)]
    }
    case ford, toyota, jaguar, bmw
    case porsche(convertible: Bool)
}
```
### 警告和错误指令
[SE-0196](https://github.com/apple/swift-evolution/blob/master/proposals/0196-diagnostic-directives.md)提议的实现。Swift 4.2提供这两个提示，来让Xcode在编译时候作出提示

- `#warning`，警告，主要为了提示后续需要处理，Xcode可以编译通过
- `#error`, 常用在Library中，强制提示，需要修复，否则不会编译通过。
```swift
func encrypt(_ string: String, with password: String) -> String {
    #warning("This is terrible method of encryption")
    return password + String(string.reversed()) + password
}
struct Configuration {
    var apiKey: String {
        #error("Please enter your API key below then delete this line.")
        return "Enter your key here"
    }
}
```
还可以和`#if`配合使用。
```swift
#if os(macOS)
#error("MyLibrary is not supported on macOS.")
#endif
```
### 动态查找成员
[SE-0195](https://github.com/apple/swift-evolution/blob/master/proposals/0195-dynamic-member-lookup.md)提议的实现。Swift 4.2提供了`@dynamicMemberLookup`的属性，和`subscript(dynamicMember:)`陪着使用，实现动态的属性的取值。
```swift
@dynamicMemberLookup
struct Person5 {
    subscript(dynamicMember member: String) -> String {
        let properties = ["name": "Tylor Swift", "city" : "Nashville"]
        return properties[member, default: ""]
    }
 }
let person5 = Person5()
print("person5.name: \(person5.name)")
print("person5.city: \(person5.city)")
print("person5.favoriteIceCream: \(person5.favoriteIceCream)")
// person5.name: Tylor Swift
// person5.city: Nashville
// person5.favoriteIceCream:
```
当然也有类似多态的用法。
```swift
@dynamicMemberLookup
struct Person5 {
    subscript(dynamicMember member: String) -> String {
        let properties = ["name": "Tylor Swift", "city" : "Nashville"]
        return properties[member, default: ""]
    }
    
    subscript(dynamicMember member: String) -> Int {
        let properties = ["age": 26, "height": 178]
        return properties[member, default: 0]
    }
 }
let person5 = Person5()
print("person5.age: \(person5.age)")
let age: Int = person5.age
print("person5.age2: \(age)")
// person5.age: 
// person5.age2: 26
```
注意你需要指定明确指定类型，Swift才能正确使用。
而且如果已经有存在属性，动态属性将不会生效
```swift
struct Singer {
    public var name = "Justin Bieber"
    subscript(dynamicMember member: String) -> String {
        return "Taylor Swift"
    }
}
let singer = Singer()
print(singer.name)
// Justin Bieber
```
`@dynamicMemberLookup`可以用在协议，结构体，枚举，类，甚至标注为`@objc`的类，以及它们的继承者。
例如，陪着协议的使用，你可以这样用
```swift
@dynamicMemberLookup
protocol Subscripting { }
extension Subscripting {
    subscript(dynamicMember member: String) -> String {
        return "This is coming from the subscript"
    }
}
extension String: Subscripting { }
let str = "Hello, Swift"
print(str.username)
```
Chris Lattner提议中的例子很有意义，
```swift
@dynamicMemberLookup
enum JSON {
   case intValue(Int)
   case stringValue(String)
   case arrayValue(Array<JSON>)
   case dictionaryValue(Dictionary<String, JSON>)
   var stringValue: String? {
      if case .stringValue(let str) = self {
         return str
      }
      return nil
   }
   subscript(index: Int) -> JSON? {
      if case .arrayValue(let arr) = self {
         return index < arr.count ? arr[index] : nil
      }
      return nil
   }
   subscript(key: String) -> JSON? {
      if case .dictionaryValue(let dict) = self {
         return dict[key]
      }
      return nil
   }
   subscript(dynamicMember member: String) -> JSON? {
      if case .dictionaryValue(let dict) = self {
         return dict[member]
      }
      return nil
   }
}
```
正常使用
```swift
let json = JSON.stringValue("Example")
json[0]?["name"]?["first"]?.stringValue
```
如果用上述的写法
```swift
json[0]?.name?.first?.stringValue
```
### 有条件地遵循协议的增强
Swift 4.1引入了有条件地遵循协议
```swift
extension Array: Purchaseable where Element: Purchaseable {
    func buy() {
        for item in self {
            item.buy()
        }
    }
}
```
但是在Swift 4.1中，如果你要确定对象是否遵循某个协议，会报错。Swift 4.2 修复了这个问题
```swift
let items: Any = [Book(), Book(), Book()]
if let books = items as? Purchaseable {
    books.buy()
}
```
还有，Swift 内置的类型，可选，数组，字典，区间，如果它们的成员遵循`Hashable`，那么它们也会自动遵循`Hashable`。
### 随机数产生和`shuffling`
[SE-0202](https://github.com/apple/swift-evolution/blob/master/proposals/0202-random-unification.md)提议的实现。Swift 4.2提供了原生的随机数方法。意味着你不需要使用`arc4random_uniform()`或者GameplayKit来实现了。
```swift
let randomInt = Int.random(in: 1..<5)
let randomFloat = Float.random(in: 1..<10)
let randomDouble = Double.random(in: 1...100)
let randomCGFloat = CGFloat.random(in: 1...1000)
let randomBool = Bool.random()
```
SE-0202同样还提议了`shuffle()`和`shuffled()`
```swift
var albums = ["Red", "1989", "Reputation"]
// shuffle in place
albums.shuffle()
// get a shuffled array back
let shuffled = albums.shuffled()
```
还有`randomElement()`方法。
```swift
if let random = albums.randomElement() {
    print("The random album is \(random).")
}
```
### 更简单，安全的Hash
[SE-0206](https://github.com/apple/swift-evolution/blob/master/proposals/0206-hashable-enhancements.md)的实现，让你更简单的为自建类型使用`Hashable`协议。
Swift 4.1 能够为遵循`Hashable`协议的类型自动生成hash值。但是如果你需要自行实现仍然需要写不少代码。
Swift 4.2 引入了`Hasher`结构，提供了随机种子，和通用的hash函数来简化过程
```swift
struct iPad: Hashable {
    var serialNumber: String
    var capacity: Int
    func hash(into hasher: inout Hasher) {
        hasher.combine(serialNumber)
    }
}
let first = iPad(serialNumber: "12345", capacity: 256)
let second = iPad(serialNumber: "54321", capacity: 512)
var hasher = Hasher()
hasher.combine(first)
hasher.combine(second)
let hash = hasher.finalize()
```
### 检查列表是否满足条件
[SE-0207](https://github.com/apple/swift-evolution/blob/master/proposals/0207-containsOnly.md)的实现，提供了`allSatisfy()`方法来检测数组中所有的元素是否都满足条件。
```swift
let scores = [85, 88, 95, 92]
let passed = scores.allSatisfy { $0 >= 85 }
```
### 原地字典的元素移除
[SE-0197](https://github.com/apple/swift-evolution/blob/master/proposals/0197-remove-where.md)提供一个全新的`removeAll(where:)`方法，以此来提供一个更高效，会操作原数据的类似`filter`的方法。
```swift
var pythons = ["John", "Michael", "Graham", "Terry", "Eric", "Terry"]
pythons.removeAll { $0.hasPrefix("Terry") }
print(pythons)
```
### Boolean toggling
[SE-0199](https://github.com/apple/swift-evolution/blob/master/proposals/0199-bool-toggle.md)提供了，对`Bool`的 `toggle()`方法，类似
```swift
extension Bool {
   mutating func toggle() {
      self = !self
   }
}
```
Swift 4.2 你可以这样
```swift
var loggedIn = false
loggedIn.toggle()
```
## Swift 4.1
### `Equatable`和`Hashable`协议
类和结构体做可比较，需要自己手动实现。
```swift
struct Person: Equatable {
    var firstName: String
    var lastName: String
    var age: Int
    var city: String
    static func ==(lhs: Person, rhs: Person) -> Bool {
        return lhs.firstName == rhs.firstName && lhs.lastName == rhs.lastName && lhs.age == rhs.age && lhs.city == rhs.city
    }
}
let person1 = Person(firstName: "hicc", lastName: "w", age: 20, city: "shenzhen")
let person2 = Person(firstName: "hicc", lastName: "w", age: 20, city: "shenzhen")
print("person1 1 == person2 : \(person1 == person2)")
// person1 1 == person2 : true
```
Swift 4.1 提供了`Equatable`的协议，它会自动的生成`==`方法。
当然你还是可以自己实现`==`方法(例如，业务有id之类的属性)。
还有之前实现一个对象的hash值也是一件麻烦的事情，你可能需要手动实现类似：
```swift
var hashValue: Int {
    return firstName.hashValue ^ lastName.hashValue &* 16777619
}
```
Swift 4.1 提供了`Hashable`的协议，可以自动生成`hashValue`，你也还是可以自行实现。
```swift
struct Person2: Equatable, Hashable {
    var firstName: String
    var lastName: String
    var age: Int
    var city: String
}
let person11 = Person2(firstName: "hicc", lastName: "w", age: 20, city: "shenzhen")
let person22 = Person2(firstName: "hicc", lastName: "w", age: 20, city: "shenzhen")
print("person11 1 == person22 : \(person11 == person22), \(person11.hashValue)")
// person11 1 == person22 : true, 5419288582170212869
```
### `Codable`协议，Key值转化策略
Swift 4提供了很方便的`Codable`协议，但是它使用下划线snake_case而不是驼峰式的方式来转化Key，不太自由。
Swift 4.1 中针对这种情况，提供了`keyDecodingStrategy`，以及`keyEncodingStrategy`属性(默认`.useDefaultKeys`)来解决这些问题。
```swift
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
do {
    let macs = try decoder.decode([Mac].self, from: jsonData)
    print(macs)
} catch {
    print(error.localizedDescription)
}
let encoder = JSONEncoder()
encoder.keyEncodingStrategy = .convertToSnakeCase
let encoded = try encoder.encode(someObject)
```
### 有条件地遵循协议
Swift 4.1 实现了[SE-0143](https://github.com/apple/swift-evolution/blob/master/proposals/0143-conditional-conformances.md)的提议，容许你类型在某下情况下才遵循某个协议。
```swift
extension Array: Purchaseable where Element: Purchaseable {
   func buy() {
      for item in self {
         item.buy()
      }
   }
}
```
这样会让你的代码，更加的安全。如下代码Swift中会拒绝编译，因为其未遵循`Coodable`协议.
```swift
import Foundation
struct Person {
   var name = "Taylor"
}
var people = [Person()]
var encoder = JSONEncoder()
try encoder.encode(people)
```
### 关联类型中的递归限制
Swift 4.1实现了[SE-0157](https://github.com/apple/swift-evolution/blob/master/proposals/0157-recursive-protocol-constraints.md)提议，在递归协议中，关联类型可以被定义它的协议所限制。
```swift
protocol Employee {
   associatedtype Manager: Employee
   var manager: Manager? { get set }
}
```
// TODO: 现在感受不太清楚，后续有深入了解在补充。
### `canImport`函数
[SE-0075](https://github.com/apple/swift-evolution/blob/master/proposals/0075-import-test.md)提议的实现。Swift 4.1引入了`canImport`函数，让你可以检查某个模块能否被导入。
```swift
#if canImport(SpriteKit)
   // this will be true for iOS, macOS, tvOS, and watchOS
#else
   // this will be true for other platforms, such as Linux
#endif
```
之前还有类似的方法
```swift
#if !os(Linux)
   // Matches macOS, iOS, watchOS, tvOS, and any other future platforms
#endif
#if os(macOS) || os(iOS) || os(tvOS) || os(watchOS)
   // Matches only Apple platforms, but needs to be kept up to date as new platforms are added
#endif
```
### `targetEnvironment`函数
[SE-0190](https://github.com/apple/swift-evolution/blob/master/proposals/0190-target-environment-platform-condition.md)提议的实现，Swift 4.1 提供了`targetEnvironment`函数，来检测是模拟器还是真实的硬件。
```swift
#if targetEnvironment(simulator)
   // code for the simulator here
#else
   // code for real devices here
#endif
```
### flatMap改名为compactMap
flatMap之前一个很有用的作用是能够过滤数组中为`nil`的元素，Swift 4.2重命名为指意明确，更强大的`compactMap`
```swift
let array = ["1", "2", "Fish"]
let numbers = array.compactMap { Int($0) }
// [1, 2]
```
## Swift 4.0
### `Coodable`协议
Swift 4之前使用`NSCoding`来做encoding和decoding的事情，但是需要一些模版代码，也容易出错，Swift 4中 `Coodable`协议就是为这个而存在。
使用起来简单到不可思议。
```swift
struct Language: Codable {
    var name: String
    var version: Int
}
let swift = Language(name: "Swift", version: 4)
```
完整的使用
```swift
let encoder = JSONEncoder()
if let encoded = try? encoder.encode(swift) {
    if let json = String(data: encoded, encoding: .utf8) {
        print("swift strng\(json)")
    }
    
    let decoder = JSONDecoder()
    if let decoded = try? decoder.decode(Language.self, from: encoded) {
        print("Swift name: \(decoded.name)")
    }
}
```
### 多行字符串字面量
跨越多行的字符串可以使用`"""`来包裹。
```swift
let quotation = """
The White Rabbit put on his spectacles. "Where shall I begin,
please your Majesty?" he asked.
"Begin at the beginning," the King said gravely, "and go on
till you come to the end; then stop."
"""
```
### 改进Key-value编码中的`keypaths`
`keypaths`是指对属性的引用而不去真正读取属性的值。
```swift
struct Crew {
    var name: String
    var rank: String
}
struct Starship {
    var name: String
    var maxWarp: Double
    var captain: Crew
}
let janeway = Crew(name: "Kathryn Janeway", rank: "Captain")
let voyager = Starship(name: "Voyager", maxWarp: 9.975, captain: janeway)
let nameKeyPath = \Starship.name
let maxWarpKeyPath = \Starship.maxWarp
let captainName = \Starship.captain.name
let starshipName = voyager[keyPath: nameKeyPath]
let starshipMaxWarp = voyager[keyPath: maxWarpKeyPath]
let starshipCaptain = voyager[keyPath: captainName]
print("starshipName \(starshipName),\(starshipCaptain)")
// starshipName Voyager, Kathryn Janeway
```
### 改进字典函数
Swift 4改进了字典的诸多函数。

- `filter`返回的是个字典
- `map` 返回的仍然是数组
- `mapValues`，返回的则是字典
- `grouping`初始化方法，可以将数组处理成字典
- `default`赋值和取值会比较方便。
```swift
let cities = ["Shanghai": 24_256_800, "Karachi": 23_500_000, "Beijing": 21_516_000, "Seoul": 9_995_000];
let massiveCities = cities.filter { $0.value > 10_000_000 }
let populations = cities.map { $0.value * 2 }
let roundedCities = cities.mapValues { "\($0 / 1_000_000) million people" }
let groupedCities = Dictionary(grouping: cities.keys) { $0.first! }
let groupedCities2 = Dictionary(grouping: cities.keys) { $0.count }
var favoriteTVShows = ["Red Dwarf", "Blackadder", "Fawlty Towers", "Red Dwarf"]
var favoriteCounts = [String: Int]()
for show in favoriteTVShows {
    favoriteCounts[show, default: 0] += 1
}
print("dic\(massiveCities),\n\(populations),\n\(roundedCities),\n\(groupedCities),\n\(groupedCities2),\n\(favoriteCounts)")
// dic["Shanghai": 24256800, "Beijing": 21516000, "Karachi": 23500000],
// [43032000, 47000000, 19990000, 48513600],
///["Beijing": "21 million people", "Karachi": "23 million people", "Seoul": "9 million people", "Shanghai": "24 million people"],
// ["S": ["Seoul", "Shanghai"], "B": ["Beijing"], "K": ["Karachi"]],
// [8: ["Shanghai"], 5: ["Seoul"], 7: ["Beijing", "Karachi"]],
// ["Blackadder": 1, "Fawlty Towers": 1, "Red Dwarf": 2]
```
### 字符串又变成了Collection类型
字符串是Collection类型，这样就有了诸多便利的方法。
```swift
let quote = "It is a truth universally acknowledged that new Swift versions bring new features."
let reversed = quote.reversed()
for letter in quote {
    print(letter)
}
```
### 单侧区间
Swift 4 支持了单侧区间， 缺失的一边为0或者为集合的尽头
```swift
let characters = ["Dr Horrible", "Captain Hammer", "Penny", "Bad Horse", "Moist"]
let bigParts = characters[..<3]
let smallParts = characters[3...]
print(bigParts)
print(smallParts)
// ["Dr Horrible", "Captain Hammer", "Penny"]
// ["Bad Horse", "Moist"]
```
## Swift 3.1
### 扩展限制的优化
Swift支持对扩展做限制。
```swift
extension Collection where Iterator.Element: Comparable {
    func lessThanFirst() -> [Iterator.Element] {
        guard let first = self.first else { return [] }
        return self.filter { $0 < first }
    }
}
let items = [5, 6, 10, 4, 110, 3].lessThanFirst()
print(items)
```
```swift
extension Array where Element: Comparable {
    func lessThanFirst() -> [Element] {
        guard let first = self.first else { return [] }
        return self.filter { $0 < first }
    }
}
let items = [5, 6, 10, 4, 110, 3].lessThanFirst()
print(items)
```
上述3.0的对扩展的限制都是通过协议实现。Swift 3.1 支持使用类型来限制。
```swift
extension Array where Element == Int {
    func lessThanFirst() -> [Int] {
        guard let first = self.first else { return [] }
        return self.filter { $0 < first }
    }
}
let items = [5, 6, 10, 4, 110, 3].lessThanFirst()
print(items)
```
### 嵌套类型支持泛型
Swift 3.1支持了嵌套类型中使用泛型。
```swift
struct Message<T> {
    struct Attachment {
        var contents: T
    }
    var title: T
    var attachment: Attachment
}
```
### 序列(Sequences)协议增加了`prefix(while:)`, `drop(while:)`两个方法

- `prefix(while:)`: 遍历所有元素，直到遇到不满足条件的元素 ，并且返回满足的元素
- `drop(while:)`： 就是返回 `prefix(while:)`相反的就好。
```swift
let names = ["Michael Jackson", "Michael Jordan", "Michael Caine", "Taylor Swift", "Adele Adkins", "Michael Douglas"]
let prefixed = names.prefix { $0.hasPrefix("Michael") }
print(prefixed)
let dropped = names.drop { $0.hasPrefix("Michael") }
print(dropped)
```
## Swift 3.0
### 函数调用必须使用参数标签
Swift特点是函数可以分别制定参数标签(argument label)和参数名称(parameter name)
```swift
func someFunction(argumentLabel parameterName: Int) {
}
// 使用必须带上参数标签
someFunction(argumentLabel: 1)
// 如果不指定，参数名称可以作为菜参数标签
func someFunction(firstParameterName: Int, secondParameterName: Int) {
}
someFunction(firstParameterName: 1, secondParameterName: 2)
```
如果你不想使用参数标签，可以使用`_`代替
```swift
func someFunction(_ firstParameterName: Int, secondParameterName: Int) {
}
someFunction(1, secondParameterName: 2)
```
### 移除多余代码
主要是一些内置对象，以及和平台相关的精简，让代码更加易读。
```swift
// Swift 2.2
let blue = UIColor.blueColor()
let min = numbers.minElement()
attributedString.appendAttributedString(anotherString)
names.insert("Jane", atIndex: 0)
UIDevice.currentDevice()
// Swift 3
let blue = UIColor.blue
let min = numbers.min()
attributedString.append(anotherString)
names.insert("Jane", at: 0)
UIDevice.current
```
以及
```swift
// 第一行是Swift 2.2
// 迪尔汗是Swift 3
"  Hello  ".stringByTrimmingCharactersInSet(.whitespaceAndNewlineCharacterSet())
"  Hello  ".trimmingCharacters(in: .whitespacesAndNewlines)
"Taylor".containsString("ayl")
"Taylor".contains("ayl")
"1,2,3,4,5".componentsSeparatedByString(",")
"1,2,3,4,5".components(separatedBy: ",")
myPath.stringByAppendingPathComponent("file.txt")
myPath.appendingPathComponent("file.txt")
"Hello, world".stringByReplacingOccurrencesOfString("Hello", withString: "Goodbye")
"Hello, world".replacingOccurrences(of: "Hello", with: "Goodbye")
"Hello, world".substringFromIndex(7)
"Hello, world".substring(from: 7)
"Hello, world".capitalizedString
"Hello, world".capitalized
```
以及， `lowercaseString` -> `lowercased()`，`uppercaseString` ->`uppercased()`
```swift
dismissViewControllerAnimated(true, completion: nil)
dismiss(animated: true, completion: nil)
dismiss(animated: true)
prepareForSegue()
override func prepare(for segue: UIStoryboardSegue, sender: AnyObject?)
```
### 枚举和属性从大驼峰替换为小驼峰
正如标题说的，一方面这是Swift推荐的用法，另外就是内置对象的变化
```swift
UIInterfaceOrientationMask.Portrait // old
UIInterfaceOrientationMask.portrait // new
NSTextAlignment.Left // old
NSTextAlignment.left // new
SKBlendMode.Replace // old
SKBlendMode.replace // new
```
还有就是Swift可选类型是通过枚举来实现的
```swift
enum Optional {
    case None
    case Some(Wrapped)
}
```
如果使用`.Some`来处理可选，也需要更改
```swift
for case let .some(datum) in data {
    print(datum)
}
for case let datum? in data {
    print(datum)
}
```
### 更swift地改进C函数
大体来说就是让C函数使用更加的Swift
```swift
// Swift 2.2
let ctx = UIGraphicsGetCurrentContext()
let rectangle = CGRect(x: 0, y: 0, width: 512, height: 512)
CGContextSetFillColorWithColor(ctx, UIColor.redColor().CGColor)
CGContextSetStrokeColorWithColor(ctx, UIColor.blackColor().CGColor)
CGContextSetLineWidth(ctx, 10)
CGContextAddRect(ctx, rectangle)
CGContextDrawPath(ctx, .FillStroke)
UIGraphicsEndImageContext()
// Swift 3
if let ctx = UIGraphicsGetCurrentContext() {
    let rectangle = CGRect(x: 0, y: 0, width: 512, height: 512)
    ctx.setFillColor(UIColor.red.cgColor)
    ctx.setStrokeColor(UIColor.black.cgColor)
    ctx.setLineWidth(10)
    ctx.addRect(rectangle)
    ctx.drawPath(using: .fillStroke)
    UIGraphicsEndImageContext()
}
```
以及
```swift
// 第一行是Swift 2.2
// 第二行是Swift 3
CGAffineTransformIdentity
CGAffineTransform.identity
CGAffineTransformMakeScale(2, 2)
CGAffineTransform(scaleX: 2, y: 2)
CGAffineTransformMakeTranslation(128, 128)
CGAffineTransform(translationX: 128, y: 128)
CGAffineTransformMakeRotation(CGFloat(M_PI))
CGAffineTransform(rotationAngle: CGFloat(M_PI))
```
### 名次和动词
这部分属于Swift更加语义化的改进，到现在5.1的时候一直在改进，目前[官网最近的规范]([Swift.org - API Design Guidelines][https://swift.org/documentation/api-design-guidelines/](https://swift.org/documentation/api-design-guidelines/))方法的部分是:

- 按照它们的副作用来命名函数和方法
  - 无副作用的按照名次来命名。`x.distance(to: y)`， `i.successor()`
  - 有副作用的按照动词来命名。`print(x)`，`x.sort()`，`x.append(y)`
  - 有修改和无修改命名
    - 动词的方法中，无修改的使用过去时`ed`（通常是，不修改原数据，而是返回新的），有修改的使用现在时`ing`。
    - 名词的方法中，无修改的使用名词，有修改的前面加上`from`。






| Mutating | Nonmutating |
| --- | --- |
| x.sort() | z = x.sorted() |
| x.append() | z = x.appending(y) |








| Nonmutating | Mutating |
| --- | --- |
| x = y.union(z) | y.formUnion(z) |
| j = c.successor(i) | c.formSuccessor(&i) |









