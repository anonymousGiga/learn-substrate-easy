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

#[cfg(test)]
mod mock;

#[cfg(test)]
mod tests;

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
	pub trait Config: frame_system::Config {
		type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
	}

	// 4. Runtime Storage
	// use storageValue store class.
	#[pallet::storage]
	#[pallet::getter(fn my_class)]
	pub type Class<T: Config> = StorageValue<_, u32, ValueQuery>;

	// use storageMap store (student number -> student name).
	#[pallet::storage]
	#[pallet::getter(fn students_info)]
	pub type StudentsInfo<T: Config> = StorageMap<_, Blake2_128Concat, u32, u128, ValueQuery>;

	#[pallet::storage]
	#[pallet::getter(fn dorm_info)]
	pub type DormInfo<T: Config> = StorageDoubleMap<
		_,
		Blake2_128Concat,
		u32, //dorm number
		Blake2_128Concat,
		u32, //bed number
		u32, // student number
		ValueQuery,
	>;

	// 5. Runtime Events
	// Can stringify event types to metadata.
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {
		SetClass(u32),
		SetStudentInfo(u32, u128),
		SetDormInfo(u32, u32, u32),
	}

	// 8. Runtime Errors
	#[pallet::error]
	pub enum Error<T> {
		// Class 只允许设置一次
		SetClassDuplicate,
		// 相同学号的只允许设置一次名字
		SetStudentsInfoDuplicate,
		// 相同床位只允许设置一次
		SetDormInfoDuplicate,
	}

	// 7. Extrinsics
	// Functions that are callable from outside the runtime.
	#[pallet::call]
	impl<T: Config> Pallet<T> {
		#[pallet::weight(0)]
		pub fn set_class_info(origin: OriginFor<T>, class: u32) -> DispatchResultWithPostInfo {
			ensure_root(origin)?;

			if Class::<T>::exists() {
				return Err(Error::<T>::SetClassDuplicate.into())
			}

			Class::<T>::put(class);
			Self::deposit_event(Event::SetClass(class));

			Ok(().into())
		}

		#[pallet::weight(0)]
		pub fn set_student_info(
			origin: OriginFor<T>,
			student_number: u32,
			student_name: u128,
		) -> DispatchResultWithPostInfo {
			ensure_signed(origin)?;

			if StudentsInfo::<T>::contains_key(student_number) {
				return Err(Error::<T>::SetStudentsInfoDuplicate.into())
			}

			StudentsInfo::<T>::insert(&student_number, &student_name);
			Self::deposit_event(Event::SetStudentInfo(student_number, student_name));

			Ok(().into())
		}

		#[pallet::weight(0)]
		pub fn set_dorm_info(
			origin: OriginFor<T>,
			dorm_number: u32,
			bed_number: u32,
			student_number: u32,
		) -> DispatchResultWithPostInfo {
			ensure_signed(origin)?;

			if DormInfo::<T>::contains_key(dorm_number, bed_number) {
				return Err(Error::<T>::SetDormInfoDuplicate.into())
			}

			DormInfo::<T>::insert(&dorm_number, &bed_number, &student_number);
			Self::deposit_event(Event::SetDormInfo(dorm_number, bed_number, student_number));

			Ok(().into())
		}
	}
}
```

# 1 编写mock runtime
mock runtime是我们进行pallet测试时需要提供的runtime，用于在测试环境中为pallet的函数提供必要的运行环境。在需要测试的pallet的src目录下

# 2 设置genesisconfig

# 3 编写测试函数
## 3.1 在测试函数中调用pallet的函数

## 3.2 在测试函数中使用pallet的存储

## 3.3 测试覆盖的一般套路

# 4 参考资料

https://docs.substrate.io/v3/runtime/testing/
