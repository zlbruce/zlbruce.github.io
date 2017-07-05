---
title: OpenStack LBaaS 笔记
author: zlbruce
layout: post
permalink: /2014/12/22/openstack-lbaas-note/
categories:
  - 笔记
tags:
  - LBaaS
  - Neutron
  - OpenStack
  - 负载均衡
---
近日一段时间在折腾 OpenStack，由于组件太多，就先从简单的看起，而我的工作又和负载均衡相关，因此就看了负载均衡的部分。

LBaaS 的 API 在 [OpenStack Networking 组件的 API][1] 中进行了定义，可以通过该 API 来创建 VIP, pool, member 以及 health monitor。其接口的定义的功能还是非常简单的，同时也意味着它只能完成最基本的四层负载，并不支持高级的七层负载功能。

## 配置方法

在 OpenStack 中实现 Networking 组件的项目叫 Neutron，其配置 LBaaS 的方法如下：

  * 开启 lbaas：修改 Neutron 配置文件中的 `service_plugins`，加入 lbaas，如：

    {% highlight ini %}
    service_plugins = router,lbaas
    {% endhighlight %}

  * 指定负载均衡具体的实现：设置 Neutron 配置文件中的 `service_provider` ，Neutron 默认使用 HAProxy，其配置如下：

    {% highlight ini %}  
    service_provider=LOADBALANCER:Haproxy:neutron.services.loadbalancer.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
    {% endhighlight %}

  其格式如下：
    
    <service_type>:<name>:<driver>[:default]
    
  `service_type` 表示是哪种服务的实现，可以是以下之一：LOADBALANCER, FIREWALL, VPN, ROUTER。  
  `name` 为该实现指定一个名字。  
  `driver` 具体的实现，指向一个类名。  
  `default` 可选项，表明是该服务类型的默认实现。  
  
  * 进行各个不同负载均衡实现的配置，比如使用 HAProxy 的，可能还需要在各个 agent 中进行配置 
    
## 具体实现
    
在现有的框架下要实现一个 `service_provider` 是非常简单的，只需要编写一个继承 `LoadBalancerAbstractDriver` 的类，并实现其中的接口，最后修改 `service_provider` 指向这个类即可。所有对 LBaaS API 的调用最终都会调用到这些接口上，其接口如下：
    
{% highlight python %}
def create_vip
def update_vip
def delete_vip
def create_pool
def update_pool
def delete_pool
def stats
def create_member
def update_member
def delete_member
def update_pool_health_monitor
def create_pool_health_monitor
def delete_pool_health_monitor
{% endhighlight %}
    
不过这样虽然简单，但是不够灵活，因为这个是运行在 neutron-server 中的，而 neutron-server 可能并不一定在 OpenStack 的网络环境中，另外如果有多个不同的网络和网络节点，处理起来也比较麻烦。
    
### Agent 方案
    
所以 OpenStack 提供了一套使用 Agent 的方法，neutron-server 中只是将 API 的请求通过 RPC 调用发送到某个 Agent 去处理，而 Agent 一般就处在 OpenStack 的网络环境中，因此就解决了上述的问题。
    
默认的 HAProxy 的实现中，他的 `service_provider` 是继承的 `AgentDriverBase` （当然这个 `AgentDriverBase` 是继承的 `LoadBalancerAbstractDriver`）。这个 `AgentDriverBase` 就会选择一个合适的 Agent 出来，然后再调用对应的 RPC 接口。
    
要实现这种类型的自定义 `service_provider` 也是非常简单的：
    
  * 在 neutron-server 端，自定义一个类继承至 `AgentDriverBase`，指定 `device_driver` 的名字，例如：

    {% highlight python %}
    class MyTestLBDriver(agent_driver_base.AgentDriverBase):
      device_driver = 'MyTestAG'
    {% endhighlight %}

  这样，neutron-server 就会去选择一个名称为 &#8216;MyTestAG&#8217; 的 Agent 出来，并将对应请求发送过去。
        
  * 在需要运行的 Agent 端，继承 `AgentDeviceDriver` 类，并实现其方法，注意，需要在 `get_name` 方法中返回该 Agent 的名字，本例中应为 &#8220;MyTestAG&#8221;。  
  最后，在 `lbaas-agent.ini` 配置文件中将 `device_driver` 指定为上述实现的类即可。 
        
最后，由于我自己对 Python 并不熟悉，所以如有错误之处，谢谢指出。

 [1]: http://developer.openstack.org/api-ref-networking-v2.html "Networking API v2.0"