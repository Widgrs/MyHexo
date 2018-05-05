---
layout:	
title: 像IDE一样使用Vim（Ubuntu 16.04 LTS）	
date: 2018-02-20 16:08
updated: 2018-04-22 18:29
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

本文针对的是在一个全新的Ubuntu16.04操作系统上安装Vim，所以需要先安装一些系统需要的最基础的软件。

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

sudo apt install lrzsz
sudo apt install tree
sudo apt install cmake
```

# 2. 源码安装Vim

```shell
# 安装Vim相关依赖库的头文件，如果不需要python3、ruby、perl、gtk等也可以不安装
# 本人Ubuntu 16.04.3桌面版64位装上libgtk2.0-dev（或者libgtk-3-dev？？？鬼知道是哪一个）后会出现显卡故障、无法加载图形桌面的情况，所以这两个我都没装，反正编译也不会生成gvim，鬼知道为什么
sudo apt install python-dev
sudo apt install python3-dev
sudo apt install ruby-dev
sudo apt install liblua5.3-dev
sudo apt install libperl-dev
sudo apt install libx11-dev
sudo apt install libncurses5-dev
# sudo apt install libgtk2.0-dev
# sudo apt install libgtk-3-dev
# 为了方便，也可以用下条语句一次安装多个软件
# sudo apt install python-dev python3-dev ruby-dev liblua5.3-dev libperl-dev libx11-dev libncurses5-dev libgtk2.0-dev libgtk-3-dev

# 下载最新版本的Vim源码
git clone git@github.com:vim/vim.git ~/Download/Vim
# 卸载系统自带的老版本Vim（如果有的话）
sudo apt purge vim vim-common vim-runtime vim-tiny

# 编译Vim源代码并安装
cd ~/Download/Vim/
# Python的头文件目录 /usr/lib/python2.7/config-x86_64-linux-gnu 需要根据实际进行修改
./configure --with-features=huge --enable-pythoninterp --enable-rubyinterp --enable-luainterp --enable-perlinterp --with-python-config-dir=/usr/lib/python2.7/config-x86_64-linux-gnu/ --enable-gui=gtk2 --enable-cscope --prefix=/usr/local
make
# 使用root权限安装，否则会提示“无法创建普通文件'/usr/bin/vim': 权限不够”
sudo make install
```

安装完成后在Vim中执行`:echo has('python')` ，若输出 1 则表示构建出的 Vim 已支持 python，反之输出 0 则表示不支持。

# 3. 安装LLVM + Clang

## 3.1 使用预编译包安装LLVM + Clang 

Ubuntu 16.04官方源中Clang的默认版本是3.8，版本较旧了，所以采用LLVM官方提供的预编译包安装Clang。首先去 [LLVM Download Page](http://releases.llvm.org/download.html) 页面下载所需版本的预编译包，可以使用wget命令下载，也可以直接使用浏览器或迅雷下载下来之后再传到虚拟机中，然后在预编译包文件所在目录执行如下命令：

```shell
# 具体文件名称可以根据实际情况修改
cd ~/Download
wget http://releases.llvm.org/5.0.1/clang+llvm-5.0.1-x86_64-linux-gnu-ubuntu-16.04.tar.xz
xz -d clang+llvm-5.0.1-x86_64-linux-gnu-ubuntu-16.04.tar.xz
tar -xvf clang+llvm-5.0.1-x86_64-linux-gnu-ubuntu-16.04.tar
sudo cp -r clang+llvm-5.0.1-x86_64-linux-gnu-ubuntu-16.04/* /usr/local
# 更新动态库信息
sudo ldconfig
# 运行 clang --version 和 clang++ --version 命令检查是否成功安装
```

创建一个验证文件test.cc，输入以下内容，然后执行编译命令 `clang++ -std=c++11 -stdlib=libc++ -Werror -Weverything -Wno-disabled-macro-expansion -Wno-float-equal -Wno-c++98-compat -Wno-c++98-compat-pedantic -Wno-global-constructors -Wno-exit-time-destructors -Wno-missing-prototypes -Wno-padded -Wno-old-style-cast -lc++ -lc++abi test.cc` 并运行生成的可执行文件，看是否输出正确结果。

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

如果运行时提示 `error while loading shared libraries: libc++.so.1: cannot open shared object file: No such file or directory` ，是因为相关动态链接库被放在`/usr/local/lib`目录中，而系统默认在`/lib`和`/usr/lib`目录中查找动态链接库，所以需要将`/usr/local/lib`添加到`/etc/ld.so.conf`文件中。打开`/etc/ld.so.conf`文件，发现里面只有一行内容，即`include /etc/ld.so.conf.d/*.conf`，所以可以在`/etc/ld.so.conf.d`目录下新建一个`usr-libs.conf`文件，然后将`/usr/local/lib`添加到`usr-libs.conf`文件中并执行`sudo ldconfig`或者重启系统即可。

Ubuntu 16.04系统之所以不需要手动添加一个conf文件，是因为系统默认自带的 libc.conf 文件中包含了 /usr/local/lib。

## 3.2 使用源代码安装LLVM + Clang

从 [LLVM Download Page](http://releases.llvm.org/download.html#6.0.0) 页面下载需要的 LLVM、Clang、LLDB、libc++、libc++ ABI、complier-rt、clang-tools-extra 等源代码文件，按照 [Getting Started with the LLVM System](https://llvm.org/docs/GettingStarted.html) 页面介绍的目录结构将相应的文件夹放到正确的位置，然后创建一个build目录用于编译。

```shell
# llvm 所有源代码位于 ~/Download/LLVM+Clang/llvm 目录下
cd ~/Download/LLVM+Clang
mkdir build
cd build
# 如果想clang/clang++自动使用libc++库，那么在编译clang时就需要指定DCLANG_DEFAULT_CXX_STDLIB参数值  为libc++，否则在链接的时候自动使用gcc/g++的libstdc++库
# 默认安装位置为 /usr/local，也可以使用 DCMAKE_INSTALL_PREFIX 参数指定安装位置
cmake -G "Unix Makefiles" -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DCLANG_DEFAULT_CXX_STDLIB=libc++ -DCMAKE_BUILD_TYPE="Release" -DCMAKE_INSTALL_PREFIX=/data/home/liujingchao/Program/llvm ../llvm
# 使用四核编译加快编译速度
make -j4
make install

# 在 .bashrc 文件中添加动态链接库信息
LLVM_LIB_PATH=/data/home/liujingchao/Program/llvm/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$LLVM_LIB_PATH
```

# 4. 安装插件

首先安装 vundle 插件：

```shell
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

将备份的 .vimrc 文件拷贝到用户目录下，然后打开 Vim 执行 :PluginInstall 自动安装指定的插件及其帮助文档。

安装tagbar插件需要的exuberant-ctags和ctrlsf.vim插件需要的ack-grep。

```shell
sudo apt install exuberant-ctags
sudo apt install ack-grep
```

为了让indexer插件自动生成ctags标签并引入，需要创建~/.indexer_files文件，并设定项目名称和项目代码目录，格式如下所示：

```shell
[VimTest]
/home/dylan/Code/VimTest
```

新版 UltiSnips 并未自带预定义的代码模板，所以需要将备份好的模板文件cpp.snippets放置于~/.vim/bundle/ultisnips/mysnippets/目录下。

编译YouCompleteMe共享库：从LLVM官网下载合适版本的预编译二进制文件到Download目录并解压（第二步中已完成），然后执行如下操作后会在 ~/.vim/bundle/YouCompleteMe/third_party/ycmd 中生成 libclang.so.5、libclang.so.5.0、ycm_core.so、PYTHON_USED_DURING_BUILDING等文件。

```shell
# 单独手动安装C/C++补全
# 如果不安装boost库文件则会报错
sudo apt install libboost-all-dev
cd ~/Download
mkdir ycm_build
cd ycm_build
# clang+llvm-5.0.1-x86_64-linux-gnu-ubuntu-16.04目录根据实际情况修改
cmake -G "Unix Makefiles" -DUSE_SYSTEM_BOOST=ON -DPATH_TO_LLVM_ROOT=~/Download/clang+llvm-5.0.1-x86_64-linux-gnu-ubuntu-16.04 . ~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp
cmake --build . --target ycm_core

# 自动安装C/C++补全和Golang补全(已配置好Golang环境)
cd ~/.vim/bundle/YouCompleteMe
./install.py --clang-completer --system-libclang --go-completer
```

YouCompleteMe会在当前路径以及上层路径寻找.ycm_extra_conf.py，也可以在.vimrc文件中指定.ycm_extra_conf.py的路径。将备份的.ycm_extra_conf.py文件放置于~/.vim/bundle/YouCompleteMe/third_party/config/目录下并修改.vimrc配置文件即可。

# 5. 从备份文件安装

```shell
sudo apt update
sudo apt install openssh-server git lrzsz tree cmake python-dev python3-dev ruby-dev liblua5.3-dev libperl-dev libx11-dev libncurses5-dev exuberant-ctags ack-grep

# 配置 Git
git config --global user.name "Widgrs"
git config --global user.email "jichliu@163.com"
git config --global -l
ssh -T git@github.com

# 源码安装 Vim
sudo apt purge vim vim-common vim-runtime vim-tiny
git clone git@github.com:vim/vim.git ~/Download/Vim
cd ~/Download/Vim/
# Python 路径根据实际情况修改
./configure --with-features=huge --enable-pythoninterp --enable-rubyinterp --enable-luainterp --enable-perlinterp --with-python-config-dir=/usr/lib/python2.7/config-x86_64-linux-gnu/ --enable-gui=gtk2 --enable-cscope --prefix=/usr/local
make
sudo make install
# 安装完成后在Vim中执行:echo has('python') ，若输出 1 则表示构建出的 Vim 已支持 python，反之输出 0 则表示不支持

# 使用预编译包安装 LLVM + Clang
cd ~/Download
wget http://releases.llvm.org/6.0.0/clang+llvm-6.0.0-x86_64-linux-gnu-ubuntu-16.04.tar.xz
xz -d clang+llvm-6.0.0-x86_64-linux-gnu-ubuntu-16.04.tar.xz
tar -xvf clang+llvm-6.0.0-x86_64-linux-gnu-ubuntu-16.04.tar
sudo cp -r clang+llvm-6.0.0-x86_64-linux-gnu-ubuntu-16.04/* /usr/local
# 更新动态库信息
sudo ldconfig
# 运行 clang --version 和 clang++ --version 命令检查是否成功安装

# 将备份文件解压至当前用户主目录下并重新编译安装YouCompleteMe即可
cd ~/.vim/bundle/YouCompleteMe
./install.py --clang-completer --system-libclang --go-completer
```

# 6. 参考资料

- [所需即所获：像 IDE 一样使用 vim](https://github.com/yangyangwithgnu/use_vim_as_ide)


- [LLVM Download Page](http://releases.llvm.org/download.html)
- [Getting Started with the LLVM System](https://llvm.org/docs/GettingStarted.html)
- [Valloric/ycmd](https://github.com/Valloric/ycmd)
- [一步一步带你安装史上最难安装的 vim 插件 —— YouCompleteMe](https://www.jianshu.com/p/d908ce81017a?nomobile=yes)
- [Ubuntu 16.04 64位安装YouCompleteMe](http://www.cnblogs.com/Harley-Quinn/p/6418070.html)
- [The Go Programming Language](https://golang.google.cn/)
- [CentOS7.3使用CMake编译安装最新的LLVM和Clang4.0.1](https://typecodes.com/linux/cmakellvmclang4.html)


