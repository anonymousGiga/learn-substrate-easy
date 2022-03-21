# substrate构建一条链的体验

但凡我们要开始学习某个区块链系统，常常做的第一件事情就是把这个区块链系统的代码拉下来，然后编译后起个节点来跑一下。substrate官方教程里面的[第一课](https://docs.substrate.io/tutorials/v3/create-your-first-substrate-chain/)名称叫做创建我们的第一条链，实际上我觉得应该叫做启动substrate默认模板链的节点更贴切，因为这个教程里面实际上就是把一个用substrate已经开发好的模板链的代码拉下来，然后编译一下，然后再启动起来。这个过程实际上和我们拉一个比特币的代码，然后编译下然后再启动
，并没有太大的不同。不过即使是这样，我们还是要罗嗦一下，快速的把这个过程走一边。

## 1 substrate开发环境
### 1.1 准备环境
编译substrate模板主要需要一些预编译包和Rust开发环境，安装的命令如下：
```
# 1.安装预编译包
sudo apt update && sudo apt install -y git clang curl libssl-dev llvm libudev-dev

# 2.安装Rust编译环境
curl https://sh.rustup.rs -sSf | sh
source ~/.cargo/env
rustup default stable
rustup update
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```
执行完上述命令后，可以用如下命令进行查看：
```
rustc --version
rustup show
```


## 2 快速构建一条链

## 3 
