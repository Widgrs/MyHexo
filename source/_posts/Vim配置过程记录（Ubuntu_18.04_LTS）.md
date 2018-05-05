---
layout:	
title: Vim配置过程记录（Ubuntu 18.04 LTS）	
date: 2018-04-29 18:54
updated: 2018-05-05 20:32
comments: true
tags:
- 开发工具
- Vim
categories:
- 开发工具
- Vim	
permalink: 
---

[TOC]

# 1. 安装系统基础软件

本文针对的是在一个全新的Ubuntu18.04操作系统上安装Vim，所以需要先安装一些系统需要的最基础的软件。

<!-- more -->

```shell
sudo apt update
sudo apt install openssh-server
sudo apt install git
# 配置Git
git config --global user.name "Widgrs"
git config --global user.email "jichliu@163.com"
# 查看Git全局设置
git config --global -l
# 查看是否有SSH Keys
ls -al ~/.ssh
# 如果没有id_rsa文件，则生成一对密钥对
# 如果自己有常用的密钥对，则可以略过下一步，直接将私钥拷贝到~/.ssh目录下再更改文件权限即可
ssh-keygen -t rsa -b 4096 -C "Widgrs's Key"
# 更改id_rsa文件权限，如果提示权限太高则将权限改为600
chmod 644 id_rsa
# 将id_rsa.pub文件中的内容复制到GitHub SSH Keys设置中

# 测试是否能成功连接到GitHub
ssh -T git@github.com
# 如果成功，返回结果为：
# Hi Widgrs! You've successfully authenticated, but GitHub does not provide shell access.
# 如果提示Failed to add the host to the list of known hosts (/home/ubuntu/.ssh/known_hosts)
# 则在.ssh目录下新建一个known_hosts文件并将其权限更改为766

# 安装几个基础软件
sudo apt install lrzsz tree cmake checkinstall dos2unix curl python-pip
```

# 2. 源码安装Vim

```shell
# 安装Vim相关依赖库的头文件，如果不需要python3、ruby、perl、gtk等也可以不安装
sudo apt install python-dev python3-dev ruby-dev liblua5.3-dev libperl-dev libncurses5-dev libx11-dev libgnome2-dev libgnomeui-dev libxt-dev libgtk2.0-dev

# 下载最新版本的Vim源码
git clone git@github.com:vim/vim.git ~/Download/Vim
# 卸载系统自带的老版本Vim（如果有的话）
sudo apt remove vim vim-runtime gvim

# 编译Vim源代码并安装
cd ~/Download/Vim/
# Python3的头文件目录 /usr/lib/python3.6/config-3.6m-x86_64-linux-gnu 需要根据实际进行修改
./configure --with-features=huge --enable-multibyte --enable-python3interp --enable-rubyinterp --enable-luainterp --enable-perlinterp --with-python3-config-dir=/usr/lib/python3.6/config-3.6m-x86_64-linux-gnu/ --enable-gui=gnome2 --enable-cscope --prefix=/usr/local
make
# 使用root权限安装，否则会提示“无法创建普通文件'/usr/local/bin/vim': 权限不够”
# 使用checkinstall打包安装方便卸载，卸载使用命令sudo dpkg -r packagename
sudo checkinstall
```

安装完成后在Vim中执行`:echo has('python3')` ，若输出 1 则表示构建出的 Vim 已支持 python3，反之输出 0 则表示不支持。

# 3. 安装LLVM + Clang和Go开发环境

## 3.1 安装LLVM + Clang

Ubuntu 18.04官方源中Clang的默认版本是6.0，已经是最新版本了（2018年4月29日），所以直接使用APT安装即可。

```shell
sudo apt install llvm clang libc++-dev libc++abi-dev
```

创建一个验证文件test.cc，输入以下内容（注意最后要多留一行空行），然后执行编译命令 `clang++ -std=c++11 -stdlib=libc++ -Werror -Weverything -Wno-disabled-macro-expansion -Wno-float-equal -Wno-c++98-compat -Wno-c++98-compat-pedantic -Wno-global-constructors -Wno-exit-time-destructors -Wno-missing-prototypes -Wno-padded -Wno-old-style-cast -lc++ -lc++abi test.cc` 并运行生成的可执行文件，看是否输出正确结果。

```c++
#include <iostream>
#include <string>

class MyClass
{
public:
  std::string s ="Hello, world\n"; // Non-static data member initializer
};

int main()
{
  std::cout << MyClass().s;
  return 0;
}

```

## 3.2 安装Golang

Ubuntu 18.04官方源中Golang的默认版本是1.10，也已经是最新版本了（2018年4月29日），所以直接使用APT安装即可。

```shell
sudo apt install golang-go
```

创建一个验证文件test.go并输入以下内容，然后运行go run test.go查看是否输出正确结果。

```go
package main
import "fmt"

func main() {
    fmt.Println("Hello World!")
}
```

由于是使用APT安装，可执行程序go在/usr/bin目录下，所以不需要再设置GOROOT，只需要设置GOPATH：在~/.bashrc文件中添加如下两行，并执行source ~/.bashrc使其生效。

```shell
# $HOME/Code/Golang目录即为Go的代码目录，可以根据实际情况修改。按照约定，其下有src、bin、pkg三个目录，src用于存放源代码，bin用于存放编译生成的可执行文件，pkg用于存放编译生成的中间文件
# 执行 go build project_name 即可编译项目
# 执行 go install project_name 即可编译项目并将生成的可执行文件拷贝到 bin 目录下
# 将 $GOPATH/bin 目录也添加进系统路径，这样就可以直接执行go生成的可执行文件
export GOPATH=$HOME/Code/Golang
export PATH=$PATH:$GOPATH/bin
```

# 4. 安装插件

## 4.1 基础设置

参见备份的`.vimrc`文件

## 4.2 安装vim-plug插件

下载 [vim-plug](https://github.com/junegunn/vim-plug) 插件到 ~/.vim/autoload 目录：

```shell
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

在 .vimrc 文件中添加 call plug#begin() 和 call plug#end() 段并在其中添加插件信息，然后打开Vim运行`:PlugInstall`命令安装插件。插件会被默认下载到`~/.vim/plugged`目录，也可以自己制定目录。

## 4.3 安装vim-gutentags插件

由于ctags已经多年不维护了，所以使用其替代品[Universal Ctags](https://ctags.io/)，因为仓库里没有该软件，所以使用源码安装：

```shell
git clone git@github.com:universal-ctags/ctags.git ~/Download/ctags
cd ~/Download/ctags
# 如果提示 autoreconf: not found 需要安装 autoconf
# sudo apt-get install autoconf
./autogen.sh
# 为了避免ctags名称冲突，安装时将其名称设为exctags
./configure --program-prefix=ex --prefix=/usr/local
make
# 使用 sudo dpkg -r exctags 卸载
sudo checkinstall
```

[vim-gutentags](https://github.com/ludovicchabant/vim-gutentags) 是一个异步自动生成tags的插件，需要在后端调用ctags软件，其可以从文件当前路径向上递归查找是否有 `.git`, `.svn`, `.project` 等标志性文件（可以自定义）来确定当前文档所属的工程目录，并可以增量更新工程对应的tags文件。

## 4.4 安装YouCompleteMe插件

[YouCompleteMe](https://github.com/Valloric/YouCompleteMe) 是一个功能强大的补全插件，安装过程如下：

编译YouCompleteMe共享库：从LLVM官网下载合适版本的预编译二进制文件到Download目录并解压或者使用APT方式安装LLVM + Clang（第二步中已完成），然后执行如下操作后会在 ~/.vim/bundle/YouCompleteMe/third_party/ycmd 中生成 libclang.so.6、libclang.so.6.0、ycm_core.so、PYTHON_USED_DURING_BUILDING等文件。

```shell
# 单独手动安装C/C++补全
# 如果不安装boost库文件则会报错
sudo apt install libboost-all-dev
cd ~/Download
mkdir ycm_build
cd ycm_build
# clang+llvm-5.0.1-x86_64-linux-gnu-ubuntu-16.04目录根据实际情况修改
cmake -G "Unix Makefiles" -DUSE_SYSTEM_BOOST=ON -DPATH_TO_LLVM_ROOT=~/Download/clang+llvm-5.0.1-x86_64-linux-gnu-ubuntu-16.04 . ~/.vim/plugged/YouCompleteMe/third_party/ycmd/cpp
cmake --build . --target ycm_core

# 自动安装C/C++补全和Golang补全(已配置好Golang环境)
cd ~/.vim/plugged/YouCompleteMe
./install.py --clang-completer --system-libclang --go-completer
```

YouCompleteMe会在当前路径以及上层路径寻找.ycm_extra_conf.py，也可以在.vimrc文件中指定.ycm_extra_conf.py的路径。将备份的.ycm_extra_conf.py文件放置于~/.vim/plugged/YouCompleteMe/third_party/config/目录下并修改.vimrc配置文件即可。

## 4.5 安装vim-go插件

[vim-go](https://github.com/fatih/vim-go) 插件是一个用于支持Go语言开发的插件，首先在 .vimrc 文件中 增加一行 `Plug 'fatih/vim-go'`然后打开Vim执行`:PlugInstall`命令安装 vim-go 插件，然后执行`:GoInstallBinaries`命令自动安装相关的工具到`$GOPATH/bin`目录下。

注意：由于墙的原因，有些工具会下载失败，可以在 ~/.vim/plugged/vim-go/plugin/go.vim 文件的 s:packages 字段中找到需要从 golang.org 网站下载的包名，自己去 [Golang中国](https://www.golangtc.com/download/package) 网站下载，将下载的文件解压缩到`$GOPATH/src`目录下，然后执行如下命令安装：

```
go install golang.org/x/tools/cmd/goimports
go install golang.org/x/tools/cmd/gorename
go install golang.org/x/tools/cmd/godoc
go install golang.org/x/tools/cmd/guru
。。。等
```

# 5. 从备份文件安装

从备份文件安装首先按照步骤1、2、3安装相关软件，然后将备份文件解压至当前用户的主目录，并编译安装Universal Ctags（步骤4.3）、YouCompleteMe（步骤4.4）和GO相关工具（步骤4.5）即可。

# 6. 参考资料

- [Building Vim from source](https://github.com/Valloric/YouCompleteMe/wiki/Building-Vim-from-source)
- [所需即所获：像 IDE 一样使用 vim](https://github.com/yangyangwithgnu/use_vim_as_ide)
- [如何在 Linux 下利用 Vim 搭建 C/C++ 开发环境?](https://www.zhihu.com/question/47691414)
- [Vim 8 下 C/C++ 开发环境搭建](http://www.skywind.me/blog/archives/2084)


- [LLVM Download Page](http://releases.llvm.org/download.html)
- [Valloric/ycmd](https://github.com/Valloric/ycmd)
- [一步一步带你安装史上最难安装的 vim 插件 —— YouCompleteMe](https://www.jianshu.com/p/d908ce81017a?nomobile=yes)
- [Ubuntu 16.04 64位安装YouCompleteMe](http://www.cnblogs.com/Harley-Quinn/p/6418070.html)
- [The Go Programming Language](https://golang.google.cn/)

