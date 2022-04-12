# storage使用介绍
最开始的时候本来没准备介绍这一章节，觉得storage直接看官方文档就好，但是和身边的小伙伴交流，觉得还是应该讲一下，毕竟这部分是在pallet时经常会使用到的部分。

首先我们来讲讲对storage的理解。第一次接触substrate的时候，和容易让人把storage和其它区块链架构中的storage（持久化存储）混淆。在其它的区块链中，如ethereum或者bitcoin中，节点对区块链相关的数据使用leveldb这样的数据库进行持久化存储，substrate中也有些持久化存储。但是，这些持久化存储不是我们本节要说的我们在pallet中使用的storage。**在pallet中要使用的storage更多的其实是一个应用层的概念**，如果用城市建造来类比，持久化存储就像是整个城市的马路或者是管道，而我们谈论的storage则是某个具体建筑或者房屋里面的水管会小路，至于这些小水管（或小路）是怎么和整个城市的大路联系起来的，不是我们讨论的范围。

# storage的使用方式
前面我们在pallet的组成中介绍过storage的使用方式，但是这里我们既然是重新来讲这部分，那肯定要讲的深入一点。这里我们把[substrate官方文档](https://docs.substrate.io/rustdocs/latest/frame_support/attr.pallet.html#storage-palletstorage-optional)中的使用方式拿过来，然后逐个讲解：
```
1 #[pallet::storage]
2 #[pallet::getter(fn $getter_name)] // optional
3 $vis type $StorageName<$some_generic> $optional_where_clause
4	  = $StorageType<$generic_name = $some_generics, $other_name = $some_other, ...>;
```
上面这几行代码是官方文档中告诉我们如何在pallet中定义storage的代码，为了方便讲解，我加上了行号。

* 第一行，```#[pallet::storage]```是定义的storage时的固定写法，表示下面定义了一个storage。在定义storage时，无论你怎么使用，都必须写这一行。
* 第二行，```#[pallet::getter(fn $getter_name)]```，在有些定义storage时会使用这一行，有些不会。这一行的意思是自动为下面定义的storage生成一个getter函数，函数的名字就是$getter_name。例如我定义如下的storage：
	```
	#[pallet::storage]
	#[pallet::getter(fn my_id)]
	pub type MyId<T: Config> = StorageValue<_, u8, ValueQuery>;
	```
        这里我定义了一个存储MyId，就自动为其生成了getter函数，函数名字是my_id，后续可以在pallet使用my_id()函数来获取该Storage中存储的值。
* 第三行和第四行就是真正定义Storage。定义的格式一定是$vis type开头（其中$vis是public、或者无这些范围修饰符，也就是表示其在代码中的使用范围）。接下来的$StorageName就是存储的名字，然后紧接着的尖括号中的$some_generic是泛型类型，而$optional_where_clause是对应泛型类型的约束。所以，上面那个例子我们也可以定义成这样：
	```
	#[pallet::storage]
	#[pallet::getter(fn my_id)]
	pub type MyId<T> where T: Config = StorageValue<_, u8, ValueQuery>;
	```
* 而第四行中的$StorageType则是具体的storage类型（也就是StorageValue\StorageMap\StorageDoubleMap\StorageNMap中的一种），接着的尖括号中的第一个参数```$generic_name = $some_generics```主要用来生产storage的前缀（有兴趣的小伙伴可以深入研究下，可能和底层存储有关），在具体使用中一般都使用```_```即可,尖括号中从第二个参数起，就和具体的Storage类型相关，需要参见具体的Storage类型。

# 使用示例
下面我们再来使用示例


