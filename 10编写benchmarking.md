# 编写benchmarking 

编写benchmarking分两种情况，如下：

* 对函数进行性能测试时需要的构造条件不会涉及到本pallet以外的其它pallet；
* 在对函数进行性能测试时需要先使用其它的pallet构造测试的先决条件。

第一种情况相对来说比较简单，这个也比较好找到例子。第二种情况则比较复杂，写起来也比较麻烦。不过在我们的开发中，大部分都是第一种情况。

针对这两种情况，我们分别来讲解如何编写代码。本节，主要讲第一种情况。

# 1 编写pallet业务代码

# 2 编写mock代码
编写mock，编写benchmarking时需要的mock和编写tests差不多，甚至是更简单。

# 3 编写benchmarking

# 4 添加到runtime中

# 5 编译&生成weights.rs文件

# 6 参考文档
https://docs.substrate.io/v3/runtime/benchmarking/

# 7 完整源码参考

https://github.com/anonymousGiga/learn-substrate-easy-source/tree/main/substrate-node-template/pallets/use-benchmarking
