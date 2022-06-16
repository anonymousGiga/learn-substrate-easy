# 编写tests

本节开始，我们来学习为pallet编写测试，我们在之前学过的use-errors pallet的基础上进行。首先，我们拷贝之前的use-errors pallet，然后重命名为use-test，在对应的Cargo.toml中将包名修改pallet-use-test。

通过前面的学习我们可以知道，pallet写好后需要通过runtime加载到链上（就是runtime/src/lib.rs中的construct_runtime宏包含的部分）。那么对应到我们的测试，如果对pallet进行测试，我们也需要构建一个runtime测试环境，然后在这个环境中加载pallet，对pallet进行测试。所以，编写pallet的测试就分为以下几部分：

* 编写mock runtime;
* 编写pallet的genesisconfig;
* 编写测试。

# 1 编写mock runtime

# 2 设置genesisconfig

# 3 编写测试函数
## 3.1 在测试函数中调用pallet的函数

## 3.2 在测试函数中使用pallet的存储

## 3.3 测试覆盖的一般套路

# 4 参考资料

https://docs.substrate.io/v3/runtime/testing/
