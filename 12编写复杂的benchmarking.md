# 编写复杂的benchmarking
上节我们讲了编写benchmarking的基本步骤，也基本上可以满足大部分编写benchmarking的需求。但是在某些情况下，我们还有些稍微复杂的benchmarking的编写，会变得不一样。这里的不一样主要是当你的调度函数比较复杂，一些使用条件是需要外部的pallet来构建的情况，就例如substrate官方frame中的session和offences pallet。

最关键的就是config，要求config兼容多个pallet。下面我们就以一个完整的例子说明。

# 1 准备一个简单的pallet
我们首先准备一个简单的pallet，其源码如下：
```
//todo
```

# 2 编写mock runtime
编写mock runtime，源码如下：
```
//todo
```

# 3 编写benchmakring
编写benchmarking的代码如下：
```
//todo
```

# 4 在runtime中添加
todo

# 5 执行命令，生成权重

# 6 源码地址

# 7 参考文档

