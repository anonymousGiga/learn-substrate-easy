# 编写benchmarking1：编写简单的benchmarking 

编写benchmarking分两种情况，如下：

* 对函数进行性能测试时需要的构造条件不会涉及到本pallet以外的其它pallet；
* 在对函数进行性能测试时需要先使用其它的pallet构造测试的先决条件。

第一种情况相对来说比较简单，这个也比较好找到例子。第二种情况则比较复杂，写起来也比较麻烦。不过在我们的开发中，大部分都是第一种情况。

针对这两种情况，我们分别来讲解如何编写代码。本节，主要讲第一种情况。

# 1 编写pallet业务代码
我们创建一个pallet，名字叫做use-benchmarking，对应的use-benchmarking/src/lib.rs的代码如下：
```
#![cfg_attr(not(feature = "std"), no_std)]

pub use pallet::*;
#[frame_support::pallet]
pub mod pallet {
	use codec::Codec;
	use frame_support::{
		pallet_prelude::*, sp_runtime::traits::AtLeast32BitUnsigned, sp_std::fmt::Debug,
	};
	use frame_system::pallet_prelude::*;
	use scale_info::prelude::vec::Vec;

	use sp_io::hashing::{blake2_128, twox_128};

	#[pallet::pallet]
	#[pallet::generate_store(pub(super) trait Store)]
	pub struct Pallet<T>(_);

	// 3. Runtime Configuration Trait
	#[pallet::config]
	pub trait Config: frame_system::Config {
		type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;

		//声明StudentNumber类型
		type StudentNumberType: Member
			+ Parameter
			+ AtLeast32BitUnsigned
			+ Codec
			+ Copy
			+ Debug
			+ Default
			+ MaxEncodedLen
			+ MaybeSerializeDeserialize;

		//声明StudentName类型
		type StudentNameType: Parameter
			+ Member
			+ AtLeast32BitUnsigned
			+ Codec
			+ Default
			+ From<u128>
			+ Into<u128>
			+ Copy
			+ MaxEncodedLen
			+ MaybeSerializeDeserialize
			+ Debug;
	}

	// 4. Runtime Storage
	// 用storageMap存储学生信息，（key， value）分别对应的是学号和姓名.
	#[pallet::storage]
	#[pallet::getter(fn students_info)]
	pub type StudentsInfo<T: Config> =
		StorageMap<_, Blake2_128Concat, T::StudentNumberType, T::StudentNameType, ValueQuery>;

	// 5. Runtime Events
	// Can stringify event types to metadata.
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {
		SetStudentInfo(T::StudentNumberType, T::StudentNameType),
	}

	// 8. Runtime Errors
	#[pallet::error]
	pub enum Error<T> {
		// 相同学号的只允许设置一次名字
		SetStudentsInfoDuplicate,
	}

	// 7. Extrinsics
	// Functions that are callable from outside the runtime.
	#[pallet::call]
	impl<T: Config> Pallet<T> {
		#[pallet::weight(100)]
		pub fn set_student_info(
			origin: OriginFor<T>,
			student_number: T::StudentNumberType,
			student_name: T::StudentNameType,
		) -> DispatchResultWithPostInfo {
			ensure_signed(origin)?;

			if StudentsInfo::<T>::contains_key(student_number) {
				return Err(Error::<T>::SetStudentsInfoDuplicate.into())
			}

			StudentsInfo::<T>::insert(&student_number, &student_name);
			Self::deposit_event(Event::SetStudentInfo(student_number, student_name));

			Self::generate_key();

			Ok(().into())
		}
	}
}
```

# 2 编写mock代码
编写mock，编写benchmarking时需要的mock和编写tests差不多，甚至是更简单。

# 3 编写benchmarking

# 4 添加到runtime中

# 5 编译&生成weights.rs文件
首先时编译，编译的命令需要带上```--features runtime-benchmarks```， 完整的编译命令如下：
```
cargo build --features runtime-benchmarks
```

然后就是生成对应的weights文件，生成之前，我们需要从substrate的repo中拷贝模板放到我们的substrate-node-template目录下：
```
mkdir .maintain
cd .maintain
cp substrate/.maintain/frame-weight-template.hbs .   #此处的substrate同官方的repo仓库
```

然后就是生成weights文件的命令，如下：
```
 ./target/debug/node-template benchmark --chain dev --execution wasm --wasm-execution compiled --pallet pallet_use_benchmarking --extrinsic "*" --steps 20 --repeat 10 --output ./pallets/use-benchmarking/src/weights.rs --template ./.maintain/frame-weight-template.hbs
```

执行完后就会在```./pallets/use-benchmarking/src/```目录下生成对应的weights.rs文件

# 6 将生成的权重函数应用到pallet中

# 7 参考文档
https://docs.substrate.io/v3/runtime/benchmarking/

# 8 完整源码参考

https://github.com/anonymousGiga/learn-substrate-easy-source/tree/main/substrate-node-template/pallets/use-benchmarking
