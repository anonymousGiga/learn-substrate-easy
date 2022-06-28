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
编写mock，编写benchmarking时需要的mock和之前在use-tests中编写的差不多，我们这里直接贴代码，如下：
```
use crate as pallet_template;
use frame_support::traits::{ConstU16, ConstU64};
use frame_system as system;
use sp_core::H256;
use sp_runtime::{
	testing::Header,
	traits::{BlakeTwo256, IdentityLookup},
};

type UncheckedExtrinsic = frame_system::mocking::MockUncheckedExtrinsic<Test>;
type Block = frame_system::mocking::MockBlock<Test>;

// Configure a mock runtime to test the pallet.
frame_support::construct_runtime!(
	pub enum Test where
		Block = Block,
		NodeBlock = Block,
		UncheckedExtrinsic = UncheckedExtrinsic,
	{
		System: frame_system::{Pallet, Call, Config, Storage, Event<T>},
		TemplateModule: pallet_template::{Pallet, Call, Storage, Event<T>},
	}
);

impl system::Config for Test {
	type BaseCallFilter = frame_support::traits::Everything;
	type BlockWeights = ();
	type BlockLength = ();
	type DbWeight = ();
	type Origin = Origin;
	type Call = Call;
	type Index = u64;
	type BlockNumber = u64;
	type Hash = H256;
	type Hashing = BlakeTwo256;
	type AccountId = u64;
	type Lookup = IdentityLookup<Self::AccountId>;
	type Header = Header;
	type Event = Event;
	type BlockHashCount = ConstU64<250>;
	type Version = ();
	type PalletInfo = PalletInfo;
	type AccountData = ();
	type OnNewAccount = ();
	type OnKilledAccount = ();
	type SystemWeightInfo = ();
	type SS58Prefix = ConstU16<42>;
	type OnSetCode = ();
	type MaxConsumers = frame_support::traits::ConstU32<16>;
}

impl pallet_template::Config for Test {
	type Event = Event;
}

// Build genesis storage according to the mock runtime.
pub fn new_test_ext() -> sp_io::TestExternalities {
	system::GenesisConfig::default().build_storage::<Test>().unwrap().into()
}
```

这里需要注意的是，编写好mock后，我们对其导出，也就是在use-benchmarking/src/lib.rs添加如下代码：
```
#[cfg(test)]
mod mock;
```

# 3 编写benchmarking

编写benchmarking的目的主要是为调度函数生成对应的权重计算函数，对应到本例子中就主要是use-benchmarking的set_student_info函数生成对应的权重计算函数。我们首先在use-benchmarking/src目录下创建一个文件，名为benchmarking.rs，编写内容如下：
```
use super::*;

#[allow(unused)]
use crate::Pallet as UseBenchmarkingDemo;
use frame_benchmarking::{benchmarks, whitelisted_caller};
use frame_system::RawOrigin;

benchmarks! {
	set_student_info {  //1、准备条件
		let s in 0 .. 100;
		let caller: T::AccountId = whitelisted_caller();
	}:{     //2、调用调度函数
		let _ = UseBenchmarkingDemo::<T>::set_student_info(RawOrigin::Signed(caller).into(), s.into(), Default::default());
	}
	verify {//3、进行验证
		assert_eq!(<StudentsInfo<T>>::get::<<T as pallet::Config>::StudentNumberType>(s.into()), Default::default());
	}
        
	// 使用mock中的new_test_ext
	impl_benchmark_test_suite!(UseBenchmarkingDemo, crate::mock::new_test_ext(), crate::mock::Test);
}
```

下面我们就来逐步讲解。

编写benchmark以宏benchmarks!包含，里面的内容主要分三部分，做的事情分别是准备条件、调用调度函数、对执行结果验证。对于第一步，其实就是对我们调度函数中的变量进行赋值，详情可以参考https://docs.substrate.io/reference/how-to-guides/weights/add-benchmarks/

# 4 添加到runtime中
首先，还是需要把use-benchmarking这个pallet加到runtime的依赖中，所以在runtime/Cargo.toml中添加如下：
```
[dependencies]
...
pallet-use-benchmarking = { version = "4.0.0-dev", default-features = false, path = "../pallets/use-benchmarking" }
...

[features]
default = ["std"]
std = [
	...
	"pallet-use-benchmarking/std",
	...
]

runtime-benchmarks = [
	...
	"pallet-use-benchmarking/runtime-benchmarks",
	...
]

```

接下来，则是在runtime/src/lib.rs的代码中添加，我们先将use-benchmarking添加到runtime中，添加如下代码：
```
impl pallet_use_benchmarking::Config for Runtime {
	type Event = Event;
	type StudentNumberType = u32;
	type StudentNameType = u128;
}

construct_runtime!(
	pub enum Runtime where
		Block = Block,
		NodeBlock = opaque::Block,
		UncheckedExtrinsic = UncheckedExtrinsic
	{
		System: frame_system,
		RandomnessCollectiveFlip: pallet_randomness_collective_flip,
		Timestamp: pallet_timestamp,
		Aura: pallet_aura,
		Grandpa: pallet_grandpa,
		Balances: pallet_balances,
		TransactionPayment: pallet_transaction_payment,
		Sudo: pallet_sudo,
		...
		//添加下面这行
		UseBenchmarkingDemo: pallet_use_benchmarking,
	}
);
```
然后就是将use-benchmarking pallet添加到对应的benchmark宏中，需要添加的代码如下：
```
#[cfg(feature = "runtime-benchmarks")]
mod benches {
	define_benchmarks!(
		[frame_benchmarking, BaselineBench::<Runtime>]
		[frame_system, SystemBench::<Runtime>]
		[pallet_balances, Balances]
		[pallet_timestamp, Timestamp]
		[pallet_template, TemplateModule]
		//这行为添加的代码
		[pallet_use_benchmarking, UseBenchmarkingDemo]
	);
}
```

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
