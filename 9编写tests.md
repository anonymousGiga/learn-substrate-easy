# 编写tests

本节开始，我们来学习为pallet编写测试，我们在之前学过的use-errors pallet的基础上进行。首先，我们拷贝之前的use-errors pallet，然后重命名为use-test，在对应的Cargo.toml中将包名修改pallet-use-test。

通过前面的学习我们可以知道，pallet写好后需要通过runtime加载到链上（就是runtime/src/lib.rs中的construct_runtime宏包含的部分）。那么对应到我们的测试，如果对pallet进行测试，我们也需要构建一个runtime测试环境，然后在这个环境中加载pallet，对pallet进行测试。所以，编写pallet的测试就分为以下几部分：

* 编写mock runtime;
* 编写pallet的genesisconfig;
* 编写测试。

# 0 准备pallet
为了进行测试，我们先准备一个pallet，其名字叫做pallet-use-test，其src/lib.rs如下：
```
#![cfg_attr(not(feature = "std"), no_std)]


// 1. Imports and Dependencies
pub use pallet::*;
#[frame_support::pallet]
pub mod pallet {
	use codec::Codec;
	use frame_support::{
		pallet_prelude::*, sp_runtime::traits::AtLeast32BitUnsigned, sp_std::fmt::Debug,
	};
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
	pub trait Config: frame_system::Config {
		type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
		type ClassType: Member
			+ Parameter
			+ AtLeast32BitUnsigned
			+ Codec
			+ Copy
			+ Debug
			+ Default
			+ MaxEncodedLen
			+ MaybeSerializeDeserialize;
	}

	// 4. Runtime Storage
	// use storageValue store class.
	#[pallet::storage]
	#[pallet::getter(fn my_class)]
	pub type Class<T: Config> = StorageValue<_, T::ClassType, ValueQuery>;

	// 5. Runtime Events
	// Can stringify event types to metadata.
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {
		SetClass(T::ClassType),
	}


	// 7. Extrinsics
	// Functions that are callable from outside the runtime.
	#[pallet::call]
	impl<T: Config> Pallet<T> {
		#[pallet::weight(0)]
		pub fn set_class_info(
			origin: OriginFor<T>,
			class: T::ClassType,
		) -> DispatchResultWithPostInfo {
			ensure_root(origin)?;

			Class::<T>::put(class);
			Self::deposit_event(Event::SetClass(class));

			Ok(().into())
		}
	}
}

```

# 1 编写mock runtime
mock runtime是我们进行pallet测试时需要提供的runtime，用于在测试环境中为pallet的函数提供必要的运行环境。在需要测试的pallet的src目录下创建mock.rs，内容如下：
```
use crate as pallet_use_test;

use frame_support::traits::{ConstU16, ConstU64};
use frame_system as system;
use sp_core::H256;
use sp_runtime::{
	testing::Header,
	traits::{BlakeTwo256, IdentityLookup},
};

type UncheckedExtrinsic = frame_system::mocking::MockUncheckedExtrinsic<Test>;
type Block = frame_system::mocking::MockBlock<Test>;

frame_support::construct_runtime!(
	pub enum Test where
		Block = Block,
		NodeBlock = Block,
		UncheckedExtrinsic = UncheckedExtrinsic,
	{
		System: frame_system,
		UseTestDemo: pallet_use_test,
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

impl pallet_use_test::Config for Test {
	type Event = Event;
	type ClassType = u32;
}

pub use frame_support::pallet_prelude::GenesisBuild;

pub fn new_test_ext() -> sp_io::TestExternalities {
	system::GenesisConfig::default().build_storage::<Test>().unwrap().into()
}

```
从上面的代码可以看出，写mock runtime的方式基本上和在runtime/src/lib.rs中加载pallet的写法基本是一样的，只不过在mock runtime中，我们只需要加载我们需要测试的必要的pallet就可以了。另外在配置pallet的时候也只需要能满足测试使用就可以了，而不用配置实际的类型。


# 2 设置genesisconfig

在上面的代码中，我们还创建了一个new_test_ext函数，这个函数中，我们为测试需要的一些pallet进行初始配置，此处我们只需要为System进行默认的配置，在实际的测试情况中，往往需要为被测试的pallet以及相关的pallet提供一些初始设置。现在，我们这里的pallet-use-test还没有genesisConfig。

下面我们为pallet-use-test添加genesisConfig，在use-test/sec/lib.rs中添加代码如下：
```
	#[pallet::genesis_config]
	pub struct GenesisConfig<T: Config> {
		pub class: T::ClassType,
	}

	#[cfg(feature = "std")]
	impl<T: Config> Default for GenesisConfig<T> {
		fn default() -> Self {
			Self { class: Default::default() }
		}
	}

	#[pallet::genesis_build]
	impl<T: Config> GenesisBuild<T> for GenesisConfig<T> {
		fn build(&self) {
			Class::<T>::put(self.class);
		}
	}
```
在这个代码中，我们为pallet添加了默认的class，而这个配置在实际使用中，需要在我们的chainspec文件里面配置上此值（配置chainspec涉及到node/src/chainspec.rs和chainspec的json文件，这些内容我们后续再讲，此处我在代码的示例配置了）。

**相应的，我们也需要修改上面mock.rs里面的new_test_ext函数如下：**
```
pub fn new_test_ext() -> sp_io::TestExternalities {
	let mut storage = system::GenesisConfig::default().build_storage::<Test>().unwrap().into();
	let config: pallet_use_test::GenesisConfig<Test> = pallet_use_test::GenesisConfig { class: 2 };
	config.assimilate_storage(&mut storage).unwrap();

	storage.into()
}
```
写好mock.rs后，还需要在lib.rs中将其导出，因此在use-test/sec/lib.rs中添加：
```
#[cfg(test)]
mod mock;
```

# 3 编写测试函数
mock runtime准备好后就可以写测试函数了，我们在use-test/src目录下创建一个tests.rs的文件，添加测试函数的代码：
```
use super::pallet::Class;
use crate::mock::*;
use frame_support::{assert_noop, assert_ok};
use sp_runtime::traits::BadOrigin;

#[test]
fn test_set_class_info() {
	new_test_ext().execute_with(|| {
		assert_noop!(UseTestDemo::set_class_info(Origin::signed(1), 42), BadOrigin);
		assert_ok!(UseTestDemo::set_class_info(Origin::root(), 42));
		assert_eq!(Class::<Test>::get(), 42);
	});
}
```
* 在测试函数中调用pallet的函数:
	

* 在测试函数中使用pallet的存储:



# 4 参考资料

https://docs.substrate.io/v3/runtime/testing/
