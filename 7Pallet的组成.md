# Pallet的组成

在前一节[编写简单的pallet](6编写简单的pallet.md)中，我们已经实现了一个简单的pallet，同时也讲解了写pallet的模板。基本上写pallet都是在该模板的基础上进行修改，完成特定pallet的功能。所以，要想熟练的开发pallet，我们必须得把pallet中的各个组成部分弄清楚。本节，我们就按照模板中的各个部分的顺序来讲解pallet的组成。


## 1 导出和依赖
导出和依赖的代码如下：
```
// 1. Imports and Dependencies
pub use pallet::*;
#[frame_support::pallet]
pub mod pallet {
    use frame_support::pallet_prelude::*;
    use frame_system::pallet_prelude::*;
    ...
}
```
学过Rust的都知道，```Pub mod pallet{}```就是将我们的pallet暴露出来，可以让外部使用（如果这部分不能理解的，建议参考[文档](https://kaisery.github.io/trpl-zh-cn/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#%E4%BD%BF%E7%94%A8-pub-%E5%85%B3%E9%94%AE%E5%AD%97%E6%9A%B4%E9%9C%B2%E8%B7%AF%E5%BE%84)）. 

接下来我们说依赖，第一行```pub use pallet::*;```是可以使用pallet中的所有类型，函数，数据等。还有 ```   use frame_support::pallet_prelude::*; use frame_system::pallet_prelude::*;```这两行，引入了相关的依赖。我们在写自己的pallet时候，当pallet使用到什么依赖，可以在这里引入。


## 2 Pallet中的类型声明
```
 // 2. Declaration of the Pallet type
    // This is a placeholder to implement traits and methods.
    #[pallet::pallet]
    #[pallet::generate_store(pub(super) trait Store)]
    pub struct Pallet<T>(_);
```
接下来是Pallet类型声明，它是一系列trait和方法的拥有者，实际的作用类似于占位符。如果对这个还不理解的话，我们可以看如下Rust程序的例子：
```
trait MyTrait {
    fn info(&self);
}

struct PlaceHolder(); //本身并没有字段

impl PlaceHolder {
    fn method(&self) {
        println!("This is method.");
    }
}

impl MyTrait for PlaceHolder {
    fn info(&self) {
        println!("This is info method.");
    }
}

fn main() {
    let p = PlaceHolder();
    p.method();
    p.info();
}
```
所以在这部分中定义的```pub struct Pallet<T>(_)```就和上面的Rust例子中定义的```struct PlaceHolder();```作用一样，是method和info方法的主体。

## 3 Runtime配置trait
这部分是指定Runtime的配置trait，Pallet中使用的一些类型和常量在此trait中进行配置。通常的使用方式如下：
```
    #[pallet::config]
    pub trait Config: frame_system::Config {
        type Id: Member
			+ Parameter
			+ AtLeast32BitUnsigned
			+ Codec
			+ Copy
			+ Debug
			+ Default
			+ MaybeSerializeDeserialize;
            
        #[pallet::constant]
		type Limit: Get<u32>;
    }
```
例如我们这里定义了一个类型Id以及一个常量Limit，定义的格式就是```type 类型名/常量名: trait约束 ```，不同的是常量名字上面会加上```#[pallet::constant]```。此处定义的类型以及常量，会在runtime中（就是代码runtime/src/lib.rs中）使用时，会指定具体的类型。

对于Config中的类型，我们可以在我们整个Pallet中使用，使用的方式就是T::类型名/常量名。例如此处定义的Id，我们使用时就是T::Id。

## 4 存储
存储（Storage）允许我们在链上存储数据，使用它存储的数据可以通过Runtime进行访问。substrate提供了四种存储方式，分别是：

* Storage Value
* Storage Map
* Storage Double Map
* Storage N MAp

从字面意思，我们基本上也可以看出几种存储的区别，StorageValue是存储单个的值，StorageMap是以map的方式存储（key-value）,StorageDoubleMap则是以双键的方式存储（就是两个key对应value的方式），StorageNMap则是N个key的方式。

关于存储的介绍可以参考[官方文档](https://docs.substrate.io/v3/runtime/storage/)

定义存储通常的方式如下：
```
#[pallet::storage]
pub type MyStorageValue<T: Config> = StorageValue<_, u32, ValueQuery>;

#[pallet::storage]
pub type MyStorageMap<T: Config> = StorageMap<_, Twox64Concat, u32, u32, ValueQuery>;
```
首先是需要添加#[pallet::storage]宏，然后使用pub type 存储名 = 某种存储类型<...>。至于尖括号里面具体填的东西，可以看Storage的Rust文档，如StorageMap就可以参考[这里](https://docs.substrate.io/rustdocs/latest/frame_support/storage/types/struct.StorageMap.html)

## 5 事件
当pallet需要把运行时上的更改或变化通知给外部主体时，就需要用到事件。事件是一个枚举类型，如下：
```
#[pallet::event]
#[pallet::metadata(u32 = "Metadata")]
pub enum Event<T: Config> {
    /// Set a value.
    ValueSet(u32, T::AccountId),
}
```
在区块链写交易函数的时候，一般分为三步，分别是判断条件、修改状态、发出事件。例如我们上一节定义了```pub enum Event<T: Config> {ClaimCreated(u32, u128) }```事件，那么交易函数中就可以使用```Self::deposit_event(Event::ClaimCreated(id, claim));```发出事件。

## 6 钩子函数
钩子函数，是在区块链运行过程中希望固定执行的函数，例如我们希望在每个区块构建之前、之后的时候执行某些逻辑等，就可以把这些逻辑放在钩子函数中。钩子函数一共有：
```
pub trait Hooks<BlockNumber> {
    fn on_finalize(_n: BlockNumber) { ... }
    fn on_idle(_n: BlockNumber, _remaining_weight: Weight) -> Weight { ... }
    fn on_initialize(_n: BlockNumber) -> Weight { ... }
    fn on_runtime_upgrade() -> Weight { ... }
    fn pre_upgrade() -> Result<(), &'static str> { ... }
    fn post_upgrade() -> Result<(), &'static str> { ... }
    fn offchain_worker(_n: BlockNumber) { ... }
    fn integrity_test() { ... }
}
```
从函数名字上，我们也基本上可以判断出这些钩子函数什么时候执行。on_finalize是在区块finalize的时候执行

## 7 交易

## 8 小结

## 9 参考文档
https://kaisery.github.io/trpl-zh-cn/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#%E4%BD%BF%E7%94%A8-pub-%E5%85%B3%E9%94%AE%E5%AD%97%E6%9A%B4%E9%9C%B2%E8%B7%AF%E5%BE%84
