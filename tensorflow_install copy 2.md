---
title: "Install"
author: "Kai Zheng"
tags: ["AI"]
date: 2021-03-10T11:15:31+08:00
draft: true
---

新的GPU服务器环境配置--Ubuntu18.04+CUDA10.0+cuDNN7.6.5+TensorFlow2.0安装笔记
@QI ZHANG · DEC 30, 2019 · 7 MIN READ

第一次安装CUDA的过程简直抓狂，中间出现了很多次莫名其妙的bug，踩了很多坑。比如装好了CUDA重启后进不去桌面系统了，直接黑屏、比如鼠标键盘都不work了、再比如装好了却安装不了TensorFlow-GPU……看了一圈网上的安装教程，发现还是官方指南真香了~ 新年第一篇，分享一下我的Ubuntu 18.04 + CUDA 10.0 + cuDNN 7.6.5 + TensorFlow 2.0 安装笔记，希望可以帮助大家少踩坑。 整个安装流程大致是：安装显卡驱动 -> 安装CUDA -> 安装cuDNN -> 安装tensorflow-gpu并测试。

<!--more-->


#1. Ubuntu安装和更新
全新的ubuntu18.04系统，先进行一些基本的安装和更新。具体的系统安装过程省略。

sudo apt-get update # 更新源
sudo apt-get upgrade # 更新已安装的包
sudo apt-get install vim
#2. 安装显卡驱动
2.1 禁用Nouveau驱动
注意：使用runfile安装需要手动禁用系统自带的Nouveau驱动

lsmod | grep nouveau # 要确保这条命令无输出
vim /etc/modprobe.d/blacklist-nouveau.conf

# 添加下面两行：
#######################################################
blacklist nouveau
options nouveau modeset=0
#######################################################

# 保存后重启：
sudo update-initramfs -u
sudo reboot

# 再次输入以下命令，无输出就表示设置成功了
lsmod | grep nouveau
2.2 安装合适的显卡驱动
# 先清空现有的显卡驱动及依赖并重启
sudo apt-get remove --purge nvidia* 
sudo apt autoremove                 
sudo reboot                         
# 添加ppa源并安装最新的驱动
sudo add-apt-repository ppa:graphics-drivers/ppa 
sudo apt update
ubuntu-drivers devices                          
sudo apt install nvidia-driver-440
# 为了防止自动更新驱动导致的兼容性问题，我们还可以锁定驱动版本:
sudo apt-mark hold nvidia-driver-440 
# nvidia-driver-440 set on hold.
并在【软件和更新】菜单中的附加驱动列表中，找到刚刚安装的nvidia-driver-440，选定即可。

输入sudo reboot重启后输入nvidia-smi，显示下图信息，这样表示显卡驱动已经ready：

image-20191230144403606

lsmod | grep nvidia # 看到下面的输出则为安装成功，如果无输出，表示有问题
image-20191231111357213

也可以手动去官网下载对应的安装程序安装显卡

# 动态监测显卡
watch -n 1 nvidia-smi # 1表示每1秒刷新一次
watch -n 0.01 nvidia-smi # 也可改成0.01s刷新一次
# 也可以用gpustat
pip install gpustat
gpustat -i 1 -P
3. 安装CUDA
百度百科：CUDA（Compute Unified Device Architecture），是显卡厂商NVIDIA推出的运算平台。 CUDA是一种由NVIDIA推出的通用并行计算架构，该架构使GPU能够解决复杂的计算问题。

Linux系统下有两种方案安装CUDA：一种是Package Manager Installation(.deb)，另一种是Runfile Installation(.run)。本文采取的是第一种（也是官方推荐的方式）。

CUDA对于系统环境有严格的依赖，比如对于CUDA10.0有如下的要求。其他的版本可查看对应的Online Documentation。



#3.1 安装前的准备
在安装CUDA之前需要先确定环境是ready的，以免出现乱七八糟的bug无从下手。直接引用官网的说明：

Some actions must be taken before the CUDA Toolkit and Driver can be installed on Linux:

Verify the system has a CUDA-capable GPU.
Verify the system is running a supported version of Linux.
Verify the system has gcc installed.
Verify the system has the correct kernel headers and development packages installed.
Download the NVIDIA CUDA Toolkit.
Handle conflicting installation methods.

