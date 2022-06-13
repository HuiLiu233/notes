# SGX Environment Install

**参考：**

https://www.daimajiaoliu.com/daima/a6afda856e28c03

https://flxdu.cn/2022/04/08/Intel-R-SGX%E7%8E%AF%E5%A2%83%E5%9C%A8Ubuntu-18-04%E7%9A%84%E5%AE%89%E8%A3%85/

**主要步骤：**

1. 硬件 
2. SGX Driver
3. SGX SDK
4. SGX PSW

## 准备

1. 修改 ubuntu 源为阿里源；

2. 电脑BIOS中启用 Intel SGX。

   Thinkpad 中 BIOS 界面为 F1 进入，可修改为 Enable。

   同时，要关闭 Secure BOOT.

   **ref:** https://blog.csdn.net/weixin_39937412/article/details/109981315?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-6-109981315-null-null.pc_agg_new_rank&utm_term=bios+sgx%E8%AE%BE%E7%BD%AE&spm=1000.2123.3001.4430

3. sgx 支持情况查询命令：

   ```bash
   sudo apt install cpuid
   cpuid -1 | grep -i sgx
   ```

   期望结果：

   ```bash
   SGX: Software Guard Extensions supported = true
   SGX_LC: SGX launch config supported = false
   ```

## 安装 SGX Driver

> sudo apt-get install make
>
> dpkg-query -s linux-headers-$(uname -r)

如： `Status: install ok installed` 则没问题，否则运行：`sudo apt-get install linux-headers-$(uname -r)`

然后:

```bash
git clone https://github.com/intel/linux-sgx-driver.git
cd linux-sgx-driver && make
sudo mkdir -p "/lib/modules/"`uname -r`"/kernel/drivers/intel/sgx"    
sudo cp isgx.ko "/lib/modules/"`uname -r`"/kernel/drivers/intel/sgx"    
sudo sh -c "cat /etc/modules | grep -Fxq isgx || echo isgx >> /etc/modules"    
sudo /sbin/depmod
sudo /sbin/modprobe isgx
```

查询是否安装成功：

```bas
lsmod | grep isgx
```



## 安装 SGX SDK

```bash
# 安装工具
sudo apt-get install build-essential python ocaml ocamlbuild automake autoconf libtool wget python libssl-dev git cmake perl libcurl4-openssl-dev protobuf-compiler libprotobuf-dev debhelper reprepro unzip
# 克隆仓库
git clone https://github.com/intel/linux-sgx.git
# 预编译，需代理
cd linux-sgx && make preparation
sudo cp ~/linux-sgx/external/toolset/ubuntu18.04/* /usr/local/bin
# 检查是否拷贝成功
which ar as ld objcopy objdump ranlib
# 编译 SGX SDK
make sdk
make sdk_install_pkg
```

安装工具：

```bash
sudo apt-get install build-essential python
```

在 linux-sgx 中找到`./linux/installer/bin/sgx_linux_x64_sdk_xxx.bin`

运行 `./sgx_linux_x64_sdk_xxx.bin` ，在它询问是否安装在当前文件夹时选择 **<u>no</u>**，并安装在 `/opt/intel/` 文件夹下。

根据提示添加环境变量：

```bash
source /opt/intel/sgxsdk/environment
```



## 安装 PSW

```bash
> echo 'deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu focal main' | sudo tee /etc/atp/sources.list.d/intel-sgx.list
> wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | sudo apt-key add
> sudo apt-get update
```

```bash
sudo apt-get install libsgx-epid libsgx-quote-ex libsgx-dcap-ql
```



## 启动 aesmd 服务

```bash
sudo systemctl start aesmd
cat /var/log/syslog | grep -i aesm
sudo systemctl enable aesmd
```



## 测试

sample code里 `make`  然后 `./app`





























