# 编写pallet的Rust前置知识

## 1 rust中的trait学习
在substrate的开发中，或者说pallet的开发中，trait的使用是非常常见的，所以理解Rust中的trait非常重要。本节不会从头介绍trait的各种知识，如果你对Rust中的trait还不太了解，建议先学习[trait基础知识](https://course.rs/basic/trait/trait.html)后，再来学习本教程接下来的内容。接下来的内容都是假定你已经对trait有了基本的了解。

### 1.1 trait的孤儿规则
Rust中的trait在使用上和其它编程语言中的接口类似，为一个类型实现某个trait就类似于在其它编程语言中为某个类型实现对应的接口。但是在使用trait的时候，有一条 **非常重要的原则**(为什么重要？因为你如果不知道话，那么在某些时候会发现哪都应该ok，但是就是编译不过)，那就是：
**如果你想要为类型A实现trait T，那么A或者T至少有一个是在当前作用域中定义的**

举两个例子。
* 正确的例子：
```
//例子1
pub trait MyTrait {
    fn print();
}

pub struct MyType;

impl MyTrait for MyType {
    fn print() {
        println!("This is ok.");
    }
}
```
上面的例子1能够正确的编译，因为它遵循孤儿规则。

* 错误的例子：
```
//例子2
use std::fmt::{Error, Formatter};
impl std::fmt::Debug for () {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result<(), Error> {
        Ok(())
    }
}
```
例子2无法编译通过，因为不管是trait Debug的定义还是类型()的定义都是外部的，所以无法在我们的代码中为类型()实现trait Debug。

### 1.2 trait对象
Rust中不直接将trait当作数据类型使用，但是可以将实现了某个trait的具体的类型当作trait对象使用。看以下例子：
```
trait Drive{
    fn drive(&self);
}

struct Truck;

impl Drive for Truck {
    fn drive(&self) {
        println!("Truck run!");
    }
}


struct MotorCycle;

impl Drive for MotorCycle {
    fn drive(&self) {
        println!("MotorCycle run!");
    }
}

fn use_transportation(t: Box<dyn Drive>) {
   t.drive(); 
}

fn main() {
    let truck = Truck;
    use_transportation(Box::new(truck));

    let moto = MotorCycle;
    use_transportation(Box::new(moto));
}
```
在上面的例子中，```use_transportation```的参数就是一个trait对象，不关注具体的类型，只需要它具备drive能力即可。

### 1.3 trait的继承
Rust只支持Trait之间的继承，比如Trait A继承Trait B，语法为：
```
trait B{}
trait A: B{}
```
Trait A继承Trait B后，当某个类型C想要实现Trait A时，还必须要同时也去实现trait B。

### 1.4 关联类型
关联类型是在trait定义的语句块中，申明一个自定义类型，这样就可以在trait的方法签名中使用该类型。如下：
```
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```
当为某个类型实现具有关联类型的trait时，需要指定关联类型为具体的类型，就像下面这样：
```
struct Counter(u32);
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
    }
}

fn main() {
    let c = Counter(1);
    c.next();
}
```

## 2 substrate的设计原理

### 2.1 一个例子

### 2.2 substrate的设计原理

## 3 参考文档
https://course.rs/basic/trait/trait.html
https://rust-book.junmajinlong.com/ch11/03_trait_inherite.html
