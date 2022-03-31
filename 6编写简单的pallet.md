# 编写简单的pallet
## 1 node-template的结构
我们下载node-template（地址：https://github.com/substrate-developer-hub/substrate-node-template), 然后进入到node-template查看目录结构：
```
~/Source/learn/substrate-node-template$ tree -L 3
.
├── Cargo.lock
├── Cargo.toml
├── docker-compose.yml
├── docs
│   └── rust-setup.md
├── LICENSE
├── node
│   ├── build.rs
│   ├── Cargo.toml
│   └── src
│       ├── chain_spec.rs
│       ├── cli.rs
│       ├── command.rs
│       ├── lib.rs
│       ├── main.rs
│       ├── rpc.rs
│       └── service.rs
├── pallets
│   └── template
│       ├── Cargo.toml
│       ├── README.md
│       └── src
├── README.md
├── runtime
│   ├── build.rs
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── rustfmt.toml
├── scripts
│   ├── docker_run.sh
│   └── init.sh
└── shell.nix
```
在上述的目录结构中，node目录中是链的一些基础功能的实现（或者说比较底层的实现，如网络、rpc，搭建链的最基础的code); pallet目录中放置的就是各个pallet，也就是业务相关的模块; runtime目录中可以简单理解为把所有pallet组合到一起，也就是业务相关的逻辑，这部分和pallet目录中是我们开发中经常要动到的部分，而node中则相对来说动的少一点。

如果用一张图来展示它们之间的关系的话，可能是这样(不太准确，但大体是这么个意思)：
![node-template关系](assets/node-template关系.PNG)

当然，对于pallets来说，在runtime中使用的pallet，有些是我们自己开发的pallet，有些是substrate中已经开发好的pallet，甚至还有些是pallet是第三方开发的pallet。

## 2 编写pallet 
下面我们就开始写一个简单的pallet。

### 2.1 编写pallet的一般格式

写pallet的基本格式如下：
```
// 1. Imports and Dependencies
pub use pallet::*;
#[frame_support::pallet]
pub mod pallet {
    use frame_support::pallet_prelude::*;
    use frame_system::pallet_prelude::*;

    // 2. Declaration of the Pallet type
    // This is a placeholder to implement traits and methods.
    #[pallet::pallet]
    #[pallet::generate_store(pub(super) trait Store)]
    pub struct Pallet<T>(_);

    // 3. Runtime Configuration Trait
    // All types and constants go here.
    // Use #[pallet::constant] and #[pallet::extra_constants]
    // to pass in values to metadata.
    #[pallet::config]
    pub trait Config: frame_system::Config { ... }

    // 4. Runtime Storage
    // Use to declare storage items.
    #[pallet::storage]
    #[pallet::getter(fn something)]
    pub MyStorage<T: Config> = StorageValue<_, u32>;

    // 5. Runtime Events
    // Can stringify event types to metadata.
    #[pallet::event]
    #[pallet::generate_deposit(pub(super) fn deposit_event)]
    pub enum Event<T: Config> { ... }

    // 6. Hooks
    // Define some logic that should be executed
    // regularly in some context, for e.g. on_initialize.
    #[pallet::hooks]
    impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> { ... }

    // 7. Extrinsics
    // Functions that are callable from outside the runtime.
    #[pallet::call]
    impl<T:Config> Pallet<T> { ... }

}
```
在我们开始写一个pallet的时候，首先先把这个模板贴到编辑器里面，然后再针对我们具体的需求进行修改。所以从这里可以看出，一个pallet，如果所有功能都包括的话，基本上分为这几大部分(对应上面代码注释中的1-7)：
```
1. 依赖; 
2. pallet类型声明;
3. config trait;
4. 定义要使用的链上存储;
5. 事件;
6. 钩子函数;
7. 交易调用函数;
```
1和2基本上是固定的写法，而对于后面的3-7部分，则是根据实际需要写或者不写。关于模板中每部分的解释，可以参考[文档](https://docs.substrate.io/v3/runtime/frame/#pallets).

### 2.2 编写pallet
接下来我们将编写一个simple-pallet.

#### 2.2.1 simple-pallet功能介绍
simple-pallet是一个存证的pallet，简单说就是提供一个存取一段hash到链上的功能，和从链上读取的功能。

#### 2.2.2 创建目录
进去到我们前面下载的substrate-node-template中，进入到目录pallets中，我们可以创建我们自己的simple-pallet(一般都是在template基础上进行修改)：
```
#先进入到substrate-node-template目录，然后执行如下
cd pallets
cp template/ simple-pallet -rf
cd simple-pallet/src/
rm benchmarking.rs mock.rs tests.rs
```
接下来修改Cargo.toml，打开substrate-node-template/pallets/simple-pallet目录下的Cargo.toml文件，然后进行修改，主要修改内容如下：
```
[package]
...
description = 修改成自己的
authors = 修改成自己的
...
repository = "https://github.com/substrate-developer-hub/substrate-node-template/"
```
对于这个文件中的其它的依赖我们可以暂时先不修改，等代码写完可以再回来删除多余的依赖。

#### 2.2.3 编写代码

删除substrate-node-template/pallets/simple-pallet/src/lib.rs中的代码，然后将上面2.1节中pallet的一般格式的代码拷贝到这个文件中。接下来我们开始写代码，首先对于注释中1和2的部分，我们不用修改，对于注释6的部分我们也需要删除掉（这个例子中使用不到）。

那么接下来，我们需要修改的就是里面的3、4、5、7的部分，其实对于很多其它的pallet来说，可能也只需要修改这几部分。

首先，我们将注释3所在部分confit修改成如下：
```
    #[pallet::config]
	pub trait Config: frame_system::Config {
        type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
    }
```
这里其实就是定义了一个关联类型，这个关联类型需要满足后面的类型约束（From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>）。至于为什么是这样的约束，我们其实可以从字面意思进行理解，一个是可以转换成Event，另外一个就是它是frame_system::Config的Event类型。**对于大部分pallet来说, 如果需要使用到Event，那么都需要在这个Config中进行定义，定义的方式基本一样.**

接下来，我们修改注释4的部分如下：
```
    #[pallet::storage]
    pub type Proofs<T: Config> =
        StorageMap<_, Blake2_128Concat, Vec<u8>, Vec<u8>), ValueQuery>;
```
关于substrate中的存储，更详细的资料可以参考[文档](https://docs.substrate.io/v3/runtime/storage/)。这里我们简单解释一下，这部分就是在链上定义了一个存储，是一个key-value方式的存储结构，用于存储我们后面要使用的存证，key是Vec<u8>的格式，value也是Vec<u8>的格式。

再接下来，我们修改注释5的部分如下：
```
    #[pallet::event]
    #[pallet::generate_deposit(pub(super) fn deposit_event)]
    pub enum Event<T: Config> {
        ClaimCreated(Vec<u8>, Vec<u8>),
    }
```
这里的Event是用来在我们具体的函数中做完动作之后发出的，一般用来通知前端做一些处理。这里我们在Event中定义了一个事件，就是创建存证。

最后，我们修改注释7的部分来实现前面说的创建存证的逻辑，如下：
```
    #[pallet::call]
    impl<T:Config> Pallet<T> { 
        #[pallet::weight(0)]
        pub fn create_claim(origin: OriginFor<T>, claim: Vec<u8>, account: Vec<u8>) -> DispatchResultWithPostInfo {
            let sender = ensure_signed(origin)?;

            Proofs::<T>::insert(
                &claim,
                &account,
            );

            Self::deposit_event(Event::ClaimCreated(account, claim));

            Ok(().into())
        }
    }	
```



## 3 将pallet添加到runtime中

## 4 调试使用pallet中的功能

## 5 参考文档
https://docs.substrate.io/v3/runtime/frame/#pallets
