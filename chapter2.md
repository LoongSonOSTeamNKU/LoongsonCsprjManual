# 完成操作系统移植

[TOC]

## 一. 编译及烧写PMON

龙芯官方发布的开发板系统采用“开放源码的Pmon”作为系统BIOS。Pmon是一个兼有BIOS和boot loader功能的开放源码软件。
PMON的源代码主要有两种，分别是
[pmon-loongson3](http://www.loongnix.org/cgit/pmon-loongson3/)
[pmon-2HSoc](http://www.loongnix.org/cgit/pmon-2HSoc/)

目前暂不清楚使用哪一种版本的PMON

### 编译PMON

而PMON的[编译方法](http://www.loongnix.org/index.php/PMON%E7%BC%96%E8%AF%91%E6%96%B9%E6%B3%95)为：

#### 1. 编译环境

- PMON的编译环境是交叉编译，即在X86机器上进行编译，编译出MIPS版本的PMON二进制。
- 系统要求为X86 32位linux操作系统。实验测试Ubuntu，Debian，Fedora，CentOS都可以使用。
- 系统中需要先安装一些开发包：flex，bison，xutils-dev。
- Fedora系统使用yum install 命令安装，Ubuntu、Debian系统使用apt-get install命令安装。

#### 2. 安装编译器

- 在X86 Linux机器上，编译器下载[gcc-4.4.0](http://ftp.loongnix.org/toolchain/gcc/release/CROSS_COMPILE/gcc-4.4.0-pmon.tgz)
- 保存在任意位置。

``` bash
# mkdir  -p  /usr/local/comp/mips-elf/
# tar -zxvf gcc-4.4.0-pmon.tgz -C /usr/local/comp/mips-elf/
```

- 设置如下环境变量：

``` bash
vi ~/.bash_profile
```

- 在文件末尾添加下面三行：

``` bash
export LD_LIBRARY_PATH=/usr/local/comp/mips-elf/gcc-4.4.0-pmon/lib:
export CROSS_COMPILE=mipsel-linux-
export PATH=/usr/local/comp/mips-elf/gcc-4.4.0-pmon/bin/:$PATH
```

#### 3. 编译PMON

- 下载PMON源代码，

``` bash
git clone  http://www.loongnix.org/cgit/pmon-loongson3/
cd  pmon-loongson3/
```

- 下一步很重要，根据要编译平台的不同，进入不同的子目录。

例如：如果要编译3A780E单路的PMON，则进入zloader.3a780e子目录；如果要编译3A双路的PMON，则进入zloader.3aserver。 其它还有很多种平台的子目录。

- 下面以编译3A780E单路的PMON为例：

``` bash
cd zloader.3a780e
make cfg
make tgt=rom
```

- 编译正常结束后，当前目录下有一个gzrom.bin文件，这就是PMON的二进制

#### 4. 在机器上更新PMON

- 龙芯PMON支持在线更新功能，即在本机上启动PMON，在命令行上更新上面编译出来的二进制
- 在线更新命令为:

``` PMON
PMON> load -r -f 0xbfc00000  URL
```

- 其中URL指向PMON二进制所在的位置，龙芯支持通过U盘和网络两种形式。
- 由于不同的开发板的具体命令可能不同，请参考相应的开发板手册。
- 以龙芯3A780E单路为例，请下载 [《龙芯 3A+RS780E 单路开发板技术规格书》](http://www.loongson.cn/uploadfile/devsysmanual/LS3A-RS780E-s-develop_board_user_manual.pdf)，在“第4.1.2.3 PMON 的更新”中有详细描述。

#### 5. 附加说明

- 如果在编译时提示“缺少 pmoncfg文件”，通过以下步骤解决：

``` bash
cd tools/pmoncfg
make
# 此步骤将生成pmoncfg文件
# cp pmoncfg /usr/bin  （以root身份执行此命令）
```

- 再次编译，应该就可以正常通过了。

### 烧写PMON

1. 通过串口线将FPGA连接至主机
2. 按照`Flash`的烧录方法二通过串口软件`ECOM`或`SecureCRT`将编译好的`PMON`的二进制文件，烧录至`flash`中，其中波特率选为`230400`，连接正常后，根据提示，键盘输入`x`表示开始`xmodem`传输串口软件使用`xmodem`模式传输`binary`文件。等待传输完成,编程过程中，不需要拔下 `flash` 芯片。
    （其中 `ECOM`和`SecureCRT` 软件位于`lab\_environment\_v1.00\uart\_soft`，可直接运行与windows环境下，Linux环境下使用 minicom工具）

## 二、下载 完备 soc 生成的 bit 流文件

