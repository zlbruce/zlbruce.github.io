---
title: 减小 Virtualbox VM 磁盘大小
author: zlbruce
layout: post
permalink: /2014/11/03/shrink-virtualbox-vm-disk/
categories:
  - 笔记
tags:
  - Virtualbox
---
我希望把我的虚拟机导出成 OVA 文件，给其他同事使用。然而在导出之前，曾经编译过我们公司的代码，导致产生了十几 G 的文件，虽然都删除了，但是虚拟机的磁盘文件已经增长到 15G，而在虚拟机里面看到的磁盘使用量其实只有 1G，这样直接导出的话不仅费时费力，还不好传输。可以通过下面是方法减小磁盘大小：

### 转换格式

只有 VDI 的格式才能够通过 Virtualbox 的命令进行缩小，所以如果不是 VDI 格式的文件，先转换为 VDI 格式，如果已经是 VDI 格式就不需要这一步了。

    VboxManage clonehd original-disk.vmdk new-disk.vdi --format VDI

说是转换，其实是拷贝，其中，original-disk.vmdk 为现有的磁盘文件，new-disk.vdi 为新的磁盘文件。  
转换好了之后，设置虚拟机配置，让其使用新的磁盘文件。

### zerofree

[zerofree][1] 可以把磁盘中不使用的块用 0 来填充，这样磁盘能够被更好的压缩，在 Debian/Ubuntu 中可以直接使用 apt 安装

    apt-get install zerofree

然后重启虚拟机到维护模式，一般是启动菜单的第二个选项。启动到维护模式之后先将根目录挂载为只读：

    mount -n -o remount,ro /

如果挂载失败，可以先把所有在运行的服务停掉，比如 dhclient 等。  
然后使用 zerofree 对分区进行处理

    zerofree /dev/sda1

当然，你要替换成你自己的分区，等待处理完成之后，关闭虚拟机

    poweroff

### 减小磁盘文件

现在是时候对磁盘文件进行缩减了，使用 Virtualbox 提供的命令即可：

    VBoxManage modifyhd new-disk.vdi --compact

执行成功之后，再看一下磁盘文件，从之前的 15G 减小到了 1.5G，效果是还是很明显的，最后导出的虚拟机 OVA 文件只有 300 多兆。

 [1]: http://intgat.tigress.co.uk/rmy/uml/index.html "zerofree"