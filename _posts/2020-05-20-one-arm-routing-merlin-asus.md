---
title: 单臂模式使用华硕路由器
author: zlbruce
layout: post
categories:
  - 笔记
tags:
  - merlin
  - asus
  - vlan
---
## 背景
在规划家里网络的时候，打算将路由器放到客厅，因为这个位置比较居中，能够比较好的将wifi覆盖到各个方向，而我的弱电箱在入户走廊，因此一般的方式就是，从光猫（入户走廊）出来的网线接到路由器（客厅）的 wan 口上，然后路由器 lan 口出来的网线接回交换机（入户走廊）上，再由交换机接到其他房间。因此弱电箱和客厅应该有两根网线，预想如图：

![plan]({{ site.url }}/images/2020-05/home-networking-plan.png)

结果预留网线的时候弱电箱到客厅的网线只预埋了一根，于是上述的预想就泡汤了，那么接下来有大概以下几种方案：
* 重新开槽，很显然不太现实；
* 将路由器放回弱电箱，这会导致wifi信号可能覆盖不全；
* 使用光猫拨号，用光猫做路由，客厅的路由器只提供wifi功能。不过我以前已经对光猫做路由有阴影了，也基本不考虑了；
* 采用 vlan trunk 的方式，把进出路由器的流量隔离起来，这也是本文所采用的方案。

从原始的预想图中可以看出，与路由器 wan 口相连的设备为光猫，与路由器 lan 口相连的设备为交换机，所以我们可以简单划分两个 vlan，暂且就先叫做 vlan:wan 和 vlan:lan 吧。

vlan trunk 就是在发送数据包的时候在包中带上 vlan tag，标明该数据包所属的 vlan，接收的交换机识别 tag 之后，移除该 tag，并发往正确的 vlan 中。一般采用的标准为 [IEEE 802.1Q][1]。

从以上描述可以看出，除了路由器需要支持 vlan trunk 之外，与他相连的设备也需要支持 vlan trunk，所以需要选购一台支持的交换机，我这里使用的是 CISCO SG250-08 交换机，具体的网络图如下：

![vlan]({{ site.url }}/images/2020-05/home-networking.png)

## RT-AC88U 的设置
基本的想法确定之后，接下来就看看如何配置。首先，我的路由器型号为 RT-AC88U，我们先看看路由器的架构：

![asus]({{ site.url }}/images/2020-05/asus.png)

可以简单的认为整个路由器由两部分组成，switch 和 linux 系统，路由器对外的网口都是插到的这个 switch，switch 上 8 号网口与 linux 系统的 eth0 网口相连，在路由器上使用`robocfg show`命令查看：
{% highlight bash %}
VLANs: BCM5301x enabled mac_check mac_hash
   1: vlan1: 0 1 2 3 5 7 8t
   2: vlan2: 4 8u
{% endhighlight %}

可以看到这个 switch 本身就划分了两个 vlan：vlan1 和 vlan2，而这个 8 号网口与 linux 的 eth0 口其实就用了 vlan trunk 通信，vlan1 的数据从 8 口收发的时候是带上了 tag 的，而 vlan2 的数据从 8 口收发的时候是不带 tag 的。另外 4 口其实对应的是路由器面板上 wan 口，所以其实路由器本身划分的两个 vlan，vlan1 和 vlan2 就是对应我们之前暂定的 vlan:lan 和vlan:wan。

接着我们通过`ip`命令发现 lan 口的 ip 是配置得到了 br0 网口上，通过`brctl show`查看

{% highlight bash %}
# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.244bfe4200c8       yes             vlan1
                                                        eth1
                                                        eth2
{% endhighlight %}

其中 eth1 和 eth2 分别对应的是 2.4G 和 5G 的无线网口，很显然应该是 lan 口，因此再一次说明了 vlan1 代表之前的 vlan:lan。
很显然我们沿用路由器的设定会比较简单，即 vlan1 带 tag，vlan2 不带 tag，路由器上的设置只需要如下语句：

{% highlight bash %}
robocfg vlan 1 ports "0 1 2 3 4t 5 7 8t"
{% endhighlight %}

其实就是增加了`4t`，让vlan1的数据也能在 4 口上转发，并且带上其 tag。

我的路由器刷了梅林固件，因此将上面语句直接放到了`/jffs/scripts/services-start`脚本中，保证重启后也能正确的设置 vlan。

## 交换机设置
按照以上思路，交换机的设置也就比较清楚了，总共 8 个网口：
* 将 1-6 口划为 vlan1，即 lan 口，用于与其他房间的连接；
* 7 口为 vlan2，即 wan 口，用于与光猫连接；
* 8 口为 trunk 口，用于与路由器连接。

同样的，trunk 口的配置也按照 vlan1 带 tag，vlan2 不带 tag 的方式进行配置即可，如图所示：

![switch]({{ site.url }}/images/2020-05/home-networking-switch.png)

## 总结
至此，整个的配置就完全结束了。我们简单做下分析：

### 会不会影响带宽？
从我自己的分析来看，应该不影响：
1. 局域网访问，如果是有线与有线相互访问，其实是有交换机在进行转发，并不会通过 vlan trunk 口，肯定是不影响的。
2. 局域网访问，如果是无线与有线相互访问，最初我认为会有影响，毕竟两个网口退化为一个网口了，但是当我发现路由器结构后，发现其内部用的就是 vlan trunk，因此路由器内部本身就退化为一个网口了，所以我个人认为不影响。
3. 局域网访问，无线与无线互通，也不会通过 vlan trunk 口，肯定也不影响。
4. 有线上网，首先自家的宽带才 300M，1G 网口就算损失一点 vlan tag 的带宽，也绰绰有余，另外加上上述第 2 点，我认为是不会有影响的。
5. 无线上网，也不会通过 vlan trunk 口，肯定不会有影响。

### vlan2 不带 tag 的好处？
在我最初的设想图中，vlan:lan 和 vlan:wan 都应该是带 tag 的，为了最小改动，才将 vlan2 设置为不带 tag。不过，现在思考后发现，其实不带 tag 有一个巨大的好处：假设路由器坏掉，临时换用其他路由器时，可以直接将网线接到新的路由器上，就能直接拨号了。虽然少了 vlan1，但是只会影响有线的网络，无线仍然可以正常使用。

## RT-AX86U 的设置
**2021-05-05 更新：**使用了快两年的 RT-AC88U 由于电源开关有点问题，京东直接免费更换了一个 RT-AX86U，不得不夸一下京东的售后。这里更新一下 RT-AX86U 的配置。

更换路由器后是可以直接拨号并上网的，同时无线也能正常使用，所以可以用笔记本去登录路由器。登录之后发现没有`robocfg`命令，说明其内部结构应该与 RT-AC88U 不同。

通过`ip`命令可以看到 lan 口 ip 仍然配置在了 br0 上，所以还是用`brctl show`查看一下：

{% highlight bash %}
# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.fc3497390200       yes             eth1
                                                        eth2
                                                        eth3
                                                        eth4
                                                        eth5
                                                        eth6
                                                        eth7
{% endhighlight %}

很显然 linux 接管了所有的网口，不再有类似 switch 的东西了。我们现在的问题是局域网的有线网络还用不了，也就是 vlan1 还没有加入到这个 br0，通过排除法，剩余的网口 eth0 就是我们之前定义的 vlan trunk 口，所以只需要在上面创建一个 vlan1，并把他加入到 br0 中即可：

{% highlight bash %}
ip link add link eth0 name eth0.1 type vlan id 1
ip link set eth0.1 up
brctl addif br0 eth0.1
{% endhighlight %}

同样，将上面语句放到`/jffs/scripts/services-start`保证开机能够执行即可。

既然路由器内部不再是 vlan trunk 的方式了，因此之前总结的无线与有线的互通，在这台路由器上应该是有影响的了。

 [1]: https://en.wikipedia.org/wiki/IEEE_802.1Q "IEEE 802.1Q"
