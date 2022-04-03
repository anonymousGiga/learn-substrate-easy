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

## 3 Runtime配置trait

## 4 存储

## 5 事件

## 6 钩子函数

## 7 交易

## 8 小结

## 9 参考文档
https://kaisery.github.io/trpl-zh-cn/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#%E4%BD%BF%E7%94%A8-pub-%E5%85%B3%E9%94%AE%E5%AD%97%E6%9A%B4%E9%9C%B2%E8%B7%AF%E5%BE%84
