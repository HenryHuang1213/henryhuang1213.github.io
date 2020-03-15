---
layout: post
title: '快速创建Linux下Docker环境的三种方式'
subtitle: 'Vagrant + VirtualBox  VS  docker-machine'
date: 2020.3.14
author: HenryHuang
cover: 'https://raw.githubusercontent.com/HenryHuang1213/picture_warehouse/master/vagrant.png'
tags:
    - vagrant
    - virtualbox
    - docker
---

这里给大家介绍三种方法。

***我们分别用操作时间（完成步骤所需时间）和系统可延展性（虚拟机内操作其他安装包）两个标准来衡量。***

## [一、通过VirtualBox或者Vmware等虚拟化软件直接创建Linux虚拟机后，在虚拟机内安装Docker环境。](#jump1) 
#### 【操作时间：较长，系统可延展性：强】

## [二、通过Vagrant + VirtualBox快速创建Linux虚拟机+Docker环境（Docker Host）。](#jump2) 
#### 【操作时间：较短，系统可延展性：强】

## [三、通过docker-machine快速打键Docker host。](#jump3) 
#### 【操作时间：较短，系统可延展性：弱】

-----

Mac 与 Windows 在此处操作几乎相同，除有显著区别的地方之外，不专门分开介绍。


## <span id="jump1">方法一：</span>

## 通过VirtualBox或者Vmware等虚拟化软件直接创建Linux虚拟机后，在虚拟机内安装Docker环境

1. 直接去官网下载VirtualBox：[VirtualBox官网下载](https://www.virtualbox.org/wiki/Downloads)

2. 由于直接从国外官网或其他镜像站下载镜像太慢，国内网络建议直接从国内镜像站下载需要的系统镜像：[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/)

3. 本地安装虚拟机并进入。根据Docker文档步骤进行内部配置。下面给出主流Linux虚拟机的Docker配置文档链接：

   - [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
   - [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
   - [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
   - [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

   步骤较多，不过比较简单。直接跟着步骤复制官网代码输入就行。配置稍微需要一点时间。

-----

## <span id="jump2">方法二：</span>

## 通过Vagrant + VirtualBox快速创建Linux虚拟机+Docker环境（Docker Host）

1. 直接去官网下载VirtualBox：[VirtualBox官网下载](https://www.virtualbox.org/wiki/Downloads)

2. 下载Vagrant：[Vagrant官网下载](https://www.vagrantup.com/downloads.html)

3. 新建空白文件夹。在该文件夹目录下打开命令行，执行命令:

   ```
   vagrant init <Linux虚拟机名 \ .box包名>
   ```

   比如我需要创建一个Centos7的虚拟机，我就输入

   ```
   vagrant init centos/7
   ```

4. 这时候当前文件目录下会出现一个 Vagrantfile 文件。 **配置 Vagrantfile 文件。**

   使用编辑器打开，在结尾处：

   ```
     # config.vm.provision "shell", inline: <<-SHELL
     #   apt-get update
     #   apt-get install -y apache2
     # SHELL
   end
   ```

   取消注释（删除 # 号）。并加入Docker文档配置代码。根据自己不同的Linux虚拟机点击相应链接查看文档：

     - [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
     - [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
     - [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
     - [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

   我这里使用centos7，于是将结尾代码改为：

   ```
     config.vm.provision "shell", inline: <<-SHELL
       apt-get update
       sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
       sudo yum install -y yum-utils device-mapper-persistent-data lvm2
       sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
       sudo yum install docker-ce docker-ce-cli containerd.io
       sudo systemctl start docker
     SHELL
   end
   ```

5. 运行Vagrant。此处有两种方法：

   1. 直接运行：

      ```
      vagrant up
      ```

      vagrant会自动去抓取需要的.box镜像文件，速度会比较慢，时间会稍微有点长。

   2. 通过浏览器下载，本地添加box方式解决。

      详细步骤请移步我的另一篇文章，里面有详尽介绍 ——> [Vagrant 安装VirtualBox等虚拟机镜像.box下载缓慢问题](http://henryhuang1213.com/2020/03/12/Vagrant+VirtualBox%E5%BF%AB%E9%80%9F%E6%90%AD%E5%BB%BA%E8%99%9A%E6%8B%9F%E6%9C%BA.html)

6. 此时，可以打开virtuabox会发现已经成功创建一台新的虚拟机。进入虚拟机，查看docker是否完成安装。

   当前文件目录下：

   ```
   vagrant ssh
   ```

   进入虚拟机后，检查docker是否完成安装：

   ```
   docker version
   ```

-----

## <span id="jump3">方法三：</span>

## 通过docker-machine快速打键Docker host。

1. 前序步骤：

   ***Mac系统下如果已经安装docker，则直接已经安装docker-machine，可直接跳转步骤3***[已安装docker-machine](#jump_step3) 

   ### Windows10系统下

   #### 如果开启Hyper-V，需要停止该功能（本地系统使用docker，则需要开启此功能）。

   Win10系统左下角搜索栏，直接输入Hyper-V，点击“设置”栏下“**启用或关闭Windows功能**”。取消“Hyper-V”文件夹前面的勾中状态（此处需要重启电脑生效）。

   同时，需要BIOS开起硬件虚拟化支持。

   #### 检查是否已经开启虚拟化功能。
   
      - 打开“任务管理器”
      - 选择“性能”
      - 查看CPU界面右下角“**虚拟化：**”是否显示“已开启”
      - 如果没有，则：
       > 1、重启电脑，在主板显示画面，快速寻找进入BIOS的按键。根据品牌不同，可能是F2、Del或其他键。
         2、进入BIOS后，寻找进入“System Configuration”。
         3、找到“Virtualization Technology”，按Enter回车键。
         4、选择“Enabled”，按Enter回车键。
         5、然后保存重启即可。

2. 下载docker-machine：

   打开docker-machine官网下载页面：[docker-machine](https://github.com/docker/machine/releases/)

   下拉选择自己当前系统的版本进行下载。

   此处演示Windows10，即点击下载 [docker-machine-Windows-x86_64.exe](https://github.com/docker/machine/releases/download/v0.16.2/docker-machine-Windows-x86_64.exe) (可直接点击进行下载)

   下载完成后：
      1. 在C盘，“Program Files”文件夹下新建文件夹“docker-machine”
      2. 将下载文件复制到该文件夹下
      3. 重命名为“docker-machine.exe”   
      ```
      mkdir "C:\Program Files\docker-machine"
      copy <下载文件的目录>\docker-machine-Windows-x86_64.exe "C:\Program Files\docker-machine"
      cd "C:\Program Files\docker-machine"
      rename docker-machine-Windows-x86_64.exe docker-machine.exe
      ```
      4. 将“C:\Program Files\docker-machine”添加到系统环境变量里
         - 左下角搜索框输入“PATH”
         - 点击编辑系统环境变量
         - 双击“系统变量” 下的“path”
         - 点击新建，输入“C:\Program Files\docker-machine”
         - 点击确定退出

3. <span id="jump_step3">测试是否安装完成。</span>

   打开cmd或者Windows Power Shell, 键入 docker-machine 回车, 即可成功看到docker-machine相关指令。

   此时需要同时保证自己电脑已经安装虚拟机软件，如 VirtualBox。[VirtualBox官网下载](https://www.virtualbox.org/wiki/Downloads)

4. docker-machine安装简易Docker host：

   ***[快捷步骤](#jump_fast)***

   ```
   docker-machine create <虚拟机名>
   ```
   比如，我将虚拟机名取为demo1：
   ```
   docker-machine create demo1
   ```

   此时便自动通过virtualbox创建一台已安装Docker的虚拟机。

   这个步骤将会首先下载一个，boot2docker.iso的镜像文件，可能会比较慢，我们可使用另一种方案节约时间。

   ***<span id="jump_fast">通过直接下载的方式解决。</span>***

   直接到GitHub的book2docker页面下载，此处我们选择最新发布的v19.03.5：[book2docker](https://github.com/boot2docker/boot2docker/releases/tag/v19.03.5)

   下载完成后，将文件放到“C:\Users\<用户名>\.docker\.machine\cache\”文件目录下
   ```
   copy <下载文件的目录>\boot2docker.iso "C:\Users\<用户名>\.docker\.machine\cache\"
   ```

   然后运行创建虚拟机：

   ```
   docker-machine create <虚拟机名>
   ```

   完成后，进入所创建的虚拟机：

   ```
   docker-machine ssh <虚拟机名>
   ```

   测试docker是否已经安装完成：
 
   ```
   docker version
   ```

   可看到Docker client & Server的版本号及相关数据，即是完成安装。

-----

## 对比三种方式，本人更推荐第二种方式。

第一种方式相对麻烦一些，下载完整系统镜像，进入后手动配置需要时间也较长。

第三种方式，虽然可以快速搭建docker host，但是弊端也很明显，这台小型虚拟机内部是无法安装其他软件的。如果我们还有延伸相关其他需求，希望在该虚拟机内实现，就比较麻烦。
