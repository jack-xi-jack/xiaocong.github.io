---
title: consul 入门服务注册中心
date: 2025-03-08 19:08:06
---

<h1 id="8519e085"><font style="background-color:rgba(255, 255, 255, 0);">基础概念</font></h1>
---

<h2 id="5e372583"><font style="background-color:rgba(255, 255, 255, 0);">什么是注册中心</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">随着微服务理论发展的成熟，越来越多互联网公司采用微服务架构来支持业务发展。各个微服务之间都需要通过注册中心来实现自动化的注册和发现。  
</font>![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1738900841329-403a2378-9696-4410-a98b-791e28e3ccf0.jpeg)<font style="background-color:rgba(255, 255, 255, 0);">  
</font><font style="background-color:rgba(255, 255, 255, 0);">注册中心主要有三种角色：</font>

+ **<font style="background-color:rgba(255, 255, 255, 0);">服务提供者（RPC Server）</font>**<font style="background-color:rgba(255, 255, 255, 0);">：在启动时，向 Registry 注册自身服务，并向 Registry 定期发送心跳汇报存活状态。</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">服务消费者（RPC Client）</font>**<font style="background-color:rgba(255, 255, 255, 0);">：在启动时，向 Registry 订阅服务，把 Registry 返回的服务节点列表缓存在本地内存中，并与 RPC Sever 建立连接。</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">服务注册中心（Registry）</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于保存 RPC Server 的注册信息，当 RPC Server 节点发生变更时，Registry 会同步变更，RPC Client 感知后会刷新本地 内存中缓存的服务节点列表。</font>

<font style="background-color:rgba(255, 255, 255, 0);">最后，RPC Client 从本地缓存的服务节点列表中，基于负载均衡算法选择一台 RPC Sever 发起调用。  
</font><font style="background-color:rgba(255, 255, 255, 0);">目前常见的注册中心有Consul、ETCD、Zookeeper、Eureka、Nacos等。</font>

<h2 id="9cfc6012"><font style="background-color:rgba(255, 255, 255, 0);">什么是Consul</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">Consul是HashiCorp公司推出的开源工具，Consul由Go语言开发，部署起来非常容易，只需要极少的可执行程序和配置文件，具有绿色、轻量级的特点。Consul是分布式的、高可用的、 可横向扩展的用于实现分布式系统的服务发现与配置。</font>

<h2 id="22750ec8"><font style="background-color:rgba(255, 255, 255, 0);">Consul特点</font></h2>
+ <font style="background-color:rgba(255, 255, 255, 0);">服务发现（Service Discovery）：Consul提供了通过DNS或者HTTP接口的方式来注册服务和发现服务。一些外部的服务通过Consul很容易的找到它所依赖的服务。</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">健康检查（Health Checking）：Consul的Client可以提供任意数量的健康检查，既可以与给定的服务相关联(“webserver是否返回200 OK”)，也可以与本地节点相关联(“内存利用率是否低于90%”)。操作员可以使用这些信息来监视集群的健康状况，服务发现组件可以使用这些信息将流量从不健康的主机路由出去。</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">Key/Value存储：应用程序可以根据自己的需要使用Consul提供的Key/Value存储。 Consul提供了简单易用的HTTP接口，结合其他工具可以实现动态配置、功能标记、领袖选举等等功能。</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">安全服务通信：Consul可以为服务生成和分发TLS证书，以建立相互的TLS连接。意图可用于定义允许哪些服务通信。服务分割可以很容易地进行管理，其目的是可以实时更改的，而不是使用复杂的网络拓扑和静态防火墙规则。</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">多数据中心：Consul支持开箱即用的多数据中心. 这意味着用户不需要担心需要建立额外的抽象层让业务扩展到多个区域。</font>

<h2 id="73539e88"><font style="background-color:rgba(255, 255, 255, 0);">consul组件</font></h2>
+ <font style="background-color:rgba(255, 255, 255, 0);">Agent：是在 Consul 集群的每个成员上长期运行的守护进程，通过命令 consul agent 启动运行。由于所有节点都必须运行一个 Agent，因此 Agent 可以分为 client 或 Server。所有的 Agent 都可以运行DNS或HTTP接口，并负责运行监测和保持服务同步</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">Client：是将所有RPC转发到 Server 的 Agent。Client 是相对无状态的，Client 唯一执行的后台活动是加入 LAN gossip 池。这只有最小的资源开销，且只消耗少量的网络带宽</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">Server：是一个有一组扩展功能的 Agent，这些功能包括参与 Raft 选举、维护集群状态、响应RPC查询、与其他数据中心交互 WAN gossip 和转发查询给 leader 或远程的数据中心</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">Datacenter：是一个私有的、低延迟和高带宽的网络环境。这不包括通过公网的通信，但就目的而言，单个 EC2 中的多个可用区域被视为数据中心的一部分</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">Consensus：一致性。Consul 使用 Consensus 协议（具体由 Raft 算法实现）来提供一致性（由 CAP 定义），表明 leader 选举和事务的顺序达成一致</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">Gossip：Consul 使用 Gossip 协议来管理成员资格并向集群广播消息。Serf 提供了完整的 Gossip 协议，可用于多种目的，而 Consul 建立在 Serf 之上。Gossip 涉及节点到节点的随机通信，主要是通过UDP。Gossip 协议也被称为 Epidemic 协议（流行病协议）</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">LAN Gossip：指包含所有位于同一局域网或数据中心的节点的 LAN gossip 池</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">WAN Gossip：指仅包含 Server 的 WAN gossip 池。这些 Server 主要分布在不同的数据中心，通常通过Internet或者广域网进行通信</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">RPC：远程过程调用。一种 请求/响应 机制，允许 Client 向 Server 发起请求</font>

<h2 id="641185e8"><font style="background-color:rgba(255, 255, 255, 0);">Consul 架构图</font></h2>
![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1738900841304-0599a640-bce7-44a1-8834-3b42cbe90ad9.jpeg)<font style="background-color:rgba(255, 255, 255, 0);">  
</font><font style="background-color:rgba(255, 255, 255, 0);">每个为Consul提供服务的节点都会运行一个Consul Agent进程。运行代理不需要发现其他服务或获取/设置密钥/值数据。Agent负责对节点上的服务以及节点本身进行健康检查。  
</font><font style="background-color:rgba(255, 255, 255, 0);">Consul Agent 分为两种模式， Server 和 Client模式，一般部署模型是 Server + Client的模式（当然也可以纯Server）, Server 具有Client的全部功能， 但是由于Server负责存储数据，并且强一致性模型的缘故， Server数是有限的（3-5个Server节点，Client可以无限扩展的）。  
</font><font style="background-color:rgba(255, 255, 255, 0);">Agent与一个或多个Consul Server对话。Consul Server是</font>**<font style="background-color:rgba(255, 255, 255, 0);">存储</font>**<font style="background-color:rgba(255, 255, 255, 0);">和</font>**<font style="background-color:rgba(255, 255, 255, 0);">复制数据</font>**<font style="background-color:rgba(255, 255, 255, 0);">的地方。Server本身会选出一个Leader。虽然Consul可以用一台Server来运作，但建议使用3到5台，以避免故障情况导致数据丢失。建议每个数据中心采用Consul服务器集群。  
</font><font style="background-color:rgba(255, 255, 255, 0);">Server Agent维护着一个目录（Catalog），这个目录（Catalog）是由Agent提交的信息汇总形成的。目录维护着集群的高层视图，包括哪些服务可用，哪些节点运行这些服务，健康信息等。  
</font><font style="background-color:rgba(255, 255, 255, 0);">需要发现其他服务或节点的基础结构组件可以查询任何Consul Server或任何Consul Agent。Agent将查询自动转发到Server。  
</font><font style="background-color:rgba(255, 255, 255, 0);">Agent会自动将查询转发给Server Agent。 每个数据中心都运行一个Consul Server集群。当有跨数据中心的服务发现或配置请求时，本地Consul Server将请求转发到远程数据中心并返回结果。</font>

<h2 id="236bb442"><font style="background-color:rgba(255, 255, 255, 0);">Consul的使用场景</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">Consul的应用场景包括服务发现、服务隔离、服务配置：</font>

+ <font style="background-color:rgba(255, 255, 255, 0);">服务发现场景中consul作为注册中心，服务地址被注册到consul中以后，可以使用consul提供的dns、http接口查询，consul支持health check。</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">服务隔离场景中consul支持以服务为单位设置访问策略，能同时支持经典的平台和新兴的平台，支持tls证书分发，service-to-service加密。</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">服务配置场景中consul提供key-value数据存储功能，并且能将变动迅速地通知出去，借助Consul可以实现配置共享，需要读取配置的服务可以从Consul中读取到准确的配置信息。</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">Consul可以帮助系统管理者更清晰的了解复杂系统内部的系统架构，运维人员可以将Consul看成一种监控软件，也可以看成一种资产（资源）管理系统。</font>

<h1 id="039d392c"><font style="background-color:rgba(255, 255, 255, 0);">安装部署</font></h1>
---

<font style="background-color:rgba(255, 255, 255, 0);">参考文档：</font>[<font style="background-color:rgba(255, 255, 255, 0);">https://developer.hashicorp.com/consul/downloads?host=www.consul.io</font>](https://developer.hashicorp.com/consul/downloads?host=www.consul.io)

<h2 id="62a5d2da"><font style="background-color:rgba(255, 255, 255, 0);">yum部署</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">安装软件包</font>

```plain
[root@tiaoban ~]# yum install -y yum-utils
[root@tiaoban ~]# yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
[root@tiaoban ~]# yum -y install consul
```

<font style="background-color:rgba(255, 255, 255, 0);">启动服务</font>

```plain
[root@tiaoban ~]# systemctl start consul
[root@tiaoban ~]# systemctl enable consul
```

<h2 id="4443f491"><font style="background-color:rgba(255, 255, 255, 0);">二进制部署</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">下载</font>

```plain
[root@tiaoban ~]# mkdir consul
[root@tiaoban ~]# cd consul/
[root@tiaoban consul]# wget https://releases.hashicorp.com/consul/1.13.2/consul_1.13.2_linux_amd64.zip
[root@tiaoban consul]# ls
consul_1.13.2_darwin_amd64.zip
```

<font style="background-color:rgba(255, 255, 255, 0);">解压安装</font>

```plain
[root@tiaoban consul]# unzip consul_1.13.2_linux_amd64.zip 
Archive:  consul_1.13.2_linux_amd64.zip
  inflating: consul                  
[root@tiaoban consul]# ls
consul  consul_1.13.2_linux_amd64.zip
[root@tiaoban consul]# mv consul /usr/local/bin/
[root@tiaoban consul]# consul version
Consul v1.13.2
Revision 0e046bbb
Build Date 2022-09-20T20:30:07Z
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
```

<font style="background-color:rgba(255, 255, 255, 0);">启动服务测试  
</font><font style="background-color:rgba(255, 255, 255, 0);">使用单节点模式启动测试</font>

```plain
[root@tiaoban consul]# consul agent -dev -ui -client 0.0.0.0
```

![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1738900841427-2c5517e2-2c9f-4ae3-9048-de5811f58ea6.jpeg)<font style="background-color:rgba(255, 255, 255, 0);">  
</font><font style="background-color:rgba(255, 255, 255, 0);">添加启动脚本</font>

```plain
[root@tiaoban ~]# cat /usr/lib/systemd/system/consul.service 
[Unit]
Description=consul
After=network.target
    
[Service]
ExecStart=/usr/local/consul/start.sh
KillSignal=SIGTERM
    
[Install]
WantedBy=multi-user.target
[root@tiaoban ~]# mkdir -p /usr/local/consul/
[root@tiaoban ~]# cat /usr/local/consul/start.sh
#!/bin/bash
/usr/local/bin/consul agent -dev -ui -client 0.0.0.0
[root@tiaoban ~]# chmod u+x /usr/local/consul/start.sh
```

<font style="background-color:rgba(255, 255, 255, 0);">启动服务</font>

```plain
[root@tiaoban ~]# systemctl daemon-reload 
[root@tiaoban ~]# systemctl start consul
[root@tiaoban ~]# systemctl enable consul
```

<h1 id="b19ba4ef"><font style="background-color:rgba(255, 255, 255, 0);">服务启动</font></h1>
---

<font style="background-color:rgba(255, 255, 255, 0);">以下操作以yum方式安装consul配置为例，配置文件路径</font>`<font style="background-color:rgba(255, 255, 255, 0);">/etc/consul.d/consul.hcl</font>`

<h2 id="8c7054cf"><font style="background-color:rgba(255, 255, 255, 0);">单节点模式</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">修改配置文件</font>

```plain
# 指明数据中心的名字
datacenter = "my-dc-1"
# 存储状态的数据目录
data_dir = "/opt/consul"
# web ui
ui_config{
  enabled = true
}
# 节点是个Server
server = true
# 绑定的一个地址，用于节点之间通信的地址
bind_addr = "192.168.10.100"
# 期望提供的Server节点数目
bootstrap_expect=1
# Client接口绑定到的地址
client_addr = "0.0.0.0"
```

<font style="background-color:rgba(255, 255, 255, 0);">启动服务，查看集群成员</font>

```plain
[root@tiaoban consul.d]# systemctl start consul
[root@tiaoban consul.d]# consul members 
Node     Address              Status  Type    Build   Protocol  DC       Partition  Segment
tiaoban  192.168.10.100:8301  alive   server  1.13.2  2         my-dc-1  default    <all>
```

<h2 id="6955953a"><font style="background-color:rgba(255, 255, 255, 0);">集群模式</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">主机规划</font>

| **主机名** | **IP** | **角色** |
| --- | --- | --- |
| k8s-master | 192.168.10.10 | server |
| k8s-work1 | 192.168.10.11 | server |
| k8s-work2 | 192.168.10.12 | server |
| tiaoban | 192.168.10.100 | clinet |


<font style="background-color:rgba(255, 255, 255, 0);">server端配置</font>

```plain
# 指明数据中心的名字
datacenter = "my-dc-1"
# 存储状态的数据目录
data_dir = "/opt/consul"
# web ui
ui_config{
  enabled = true
}
# 节点是个Server
server = true
# 绑定的一个地址，用于节点之间通信的地址。此处以192.168.10.10为例，其他两个更换ip即可。
bind_addr = "192.168.10.10"
# 期望提供的Server节点数目
bootstrap_expect=3
# Client接口绑定到的地址
client_addr = "0.0.0.0"
```

<font style="background-color:rgba(255, 255, 255, 0);">client配置</font>

```plain
# 指明数据中心的名字
datacenter = "my-dc-1"
# 存储状态的数据目录
data_dir = "/opt/consul"
# web ui
ui_config{
  enabled = true
}
# 节点是个Server
server = false
# 绑定的一个地址，用于节点之间通信的地址
bind_addr = "192.168.10.100"
# Client接口绑定到的地址
client_addr = "0.0.0.0"
```

<font style="background-color:rgba(255, 255, 255, 0);">启动服务并加入集群</font>

```plain
# 所有机器执行
[root@k8s-master consul.d]# systemctl restart consul
# 除了k8s-master以外的其他机器执行
[root@k8s-tiaoban consul.d]# consul join 192.168.10.10
Successfully joined cluster by contacting 1 nodes.
```

<h1 id="4b194b09"><font style="background-color:rgba(255, 255, 255, 0);">Consul 常用CLI</font></h1>
---

<h2 id="eb75b9f5"><font style="background-color:rgba(255, 255, 255, 0);">查看服务列表</font></h2>
```plain
[root@k8s-master ~]# consul catalog services
consul
traefik
traefik-tcp
```

<h2 id="75acbd2e"><font style="background-color:rgba(255, 255, 255, 0);">注销服务</font></h2>
```plain
consul services deregister -id=:service-id
// :service-id 为用户服务Id
```

<h2 id="1b4e985d"><font style="background-color:rgba(255, 255, 255, 0);">查看节点成员</font></h2>
```plain
[root@k8s-master ~]# consul members
Node        Address              Status  Type    Build   Protocol  DC       Partition  Segment
k8s-master  192.168.10.10:8301   alive   server  1.13.3  2         my-dc-1  default    <all>
k8s-work1   192.168.10.11:8301   alive   server  1.13.3  2         my-dc-1  default    <all>
k8s-work2   192.168.10.12:8301   alive   server  1.13.3  2         my-dc-1  default    <all>
tiaoban     192.168.10.100:8301  alive   client  1.13.2  2         my-dc-1  default    <default>
```

<h2 id="1459fae8"><font style="background-color:rgba(255, 255, 255, 0);">查看server节点信息</font></h2>
```plain
[root@k8s-master ~]# consul operator raft list-peers
Node        ID                                    Address             State     Voter  RaftProtocol
k8s-work1   322a522a-dcf5-3727-052e-0f2d65406f8d  192.168.10.11:8300  follower  true   3
k8s-master  5db3c09a-f94d-b53f-6c9f-694beb64c1aa  192.168.10.10:8300  leader    true   3
k8s-work2   d0e4a609-f029-cd91-1d61-63c3403b82b3  192.168.10.12:8300  follower  false  3
```

<h2 id="81ea3784"><font style="background-color:rgba(255, 255, 255, 0);">Consul常用HTTP API</font></h2>
<h2 id="3b842616"><font style="background-color:rgba(255, 255, 255, 0);">服务查询</font></h2>
+ <font style="background-color:rgba(255, 255, 255, 0);">/agent/services：该端点返回在本地代理程序中注册的所有服务；</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">/agent/service/{service_id}：返回在本地代理上注册的单个服务实例的完整服务定义；</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">/agent/health/service/name/{service_name} ：通过名称检索注册的服务状态（设置了健康检查的服务）；</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">/agent/health/service/id/{service_id}：通过id检索注册的服务状态（设置了健康检查的服务）；</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">/health/service/{service_name}?passing：通过健康检查的服务(包含未设置check的service)</font>

<h2 id="f3bad82b"><font style="background-color:rgba(255, 255, 255, 0);">服务注册</font></h2>
+ <font style="background-color:rgba(255, 255, 255, 0);">/agent/service/register：注册服务；</font>

<h2 id="18616023"><font style="background-color:rgba(255, 255, 255, 0);">服务删除</font></h2>
+ <font style="background-color:rgba(255, 255, 255, 0);">/agent/service/deregister/{service_id}：注销服务；</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">/agent/service/maintenance/{service_id}：该端点将给定的服务置于“维护模式”，在维护模式下，该服务将被标记为不可用，并且不会出现在DNS或API查询中；</font>

<h1 id="f3bad82b-1"><font style="background-color:rgba(255, 255, 255, 0);">服务注册</font></h1>
---

<h2 id="15c64aa0"><font style="background-color:rgba(255, 255, 255, 0);">配置文件</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">新增json配置文件</font>

```plain
[root@k8s-master consul.d]# cat /etc/consul.d/kube-apiserver.json 
{
    "service": {
        "name": "traefik-tcp",
        "tags": [
            "k8s","traefik"
        ],
        "address": "192.168.10.10",
        "port": 9100
    }
}
```

<font style="background-color:rgba(255, 255, 255, 0);">重启服务</font>

```plain
[root@k8s-master consul.d]# systemctl restart consul
```

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<font style="background-color:rgba(255, 255, 255, 0);">web界面查看  
</font>![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1738900841445-ed3d6655-6fda-49ac-88cb-19c119d5de0f.jpeg)

<h2 id="http-api"><font style="background-color:rgba(255, 255, 255, 0);">HTTP API</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">注册一个name为traefik的服务</font>

```plain
[root@k8s-master consul.d]# curl -X PUT -d '{"name": "traefik","address": "192.168.10.10","port": 80,"tags": ["k8s"]}' http://192.168.10.10:8500/v1/agent/service/register
```

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<font style="background-color:rgba(255, 255, 255, 0);">注册带健康检查的服务</font>

```plain
curl -X PUT -d '{"name": "traefik-metrics","address": "192.168.10.10","port": 80,"tags": ["k8s"],"checks":[{"http":"http://192.168.10.10:9100/metrics","interval":"5s"}]}' http://192.168.10.10:8500/v1/agent/service/register
```

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<font style="background-color:rgba(255, 255, 255, 0);">打开管理页面查看已注册的服务  
</font>![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1738900841454-78a08404-76de-44b8-8f10-689d9517273f.jpeg)

<h2 id="sdk"><font style="background-color:rgba(255, 255, 255, 0);">SDK</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">除了采用配置文件或者HTTP API方式注册服务外，consul也支持使用sdk包注册查询服务，目前主流的开发语法均已支持，详情参考文档：  
</font>[<font style="background-color:rgba(255, 255, 255, 0);">https://developer.hashicorp.com/consul/api-docs/libraries-and-sdks</font>](https://developer.hashicorp.com/consul/api-docs/libraries-and-sdks)

<h1 id="3b842616-1"><font style="background-color:rgba(255, 255, 255, 0);">服务查询</font></h1>
---

<h2 id="http-api-1"><font style="background-color:rgba(255, 255, 255, 0);">HTTP API</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">查看服务列表</font>

```plain
[root@k8s-master ~]# curl -s http://192.168.10.10:8500/v1/catalog/services | jq
{
  "consul": [],
  "traefik": [
    "k8s"
  ],
  "traefik-tcp": [
    "k8s",
    "traefik"
  ]
}
```

<font style="background-color:rgba(255, 255, 255, 0);">查看服务详细信息</font>

```plain
[root@k8s-master ~]# curl -s http://192.168.10.10:8500/v1/catalog/service/traefik | jq
[
  {
    "ID": "eda0ef7f-ca12-bbca-c84e-f88cc3664cd1",
    "Node": "tiaoban",
    "Address": "192.168.10.100",
    "Datacenter": "my-dc-1",
    "TaggedAddresses": {
      "lan": "192.168.10.100",
      "lan_ipv4": "192.168.10.100",
      "wan": "192.168.10.100",
      "wan_ipv4": "192.168.10.100"
    },
    "NodeMeta": {
      "consul-network-segment": ""
    },
    "ServiceKind": "",
    "ServiceID": "traefik",
    "ServiceName": "traefik",
    "ServiceTags": [
      "k8s"
    ],
    "ServiceAddress": "192.168.10.10",
    "ServiceTaggedAddresses": {
      "lan_ipv4": {
        "Address": "192.168.10.10",
        "Port": 80
      },
      "wan_ipv4": {
        "Address": "192.168.10.10",
        "Port": 80
      }
    },
    "ServiceWeights": {
      "Passing": 1,
      "Warning": 1
    },
    "ServiceMeta": {},
    "ServicePort": 80,
    "ServiceSocketPath": "",
    "ServiceEnableTagOverride": false,
    "ServiceProxy": {
      "Mode": "",
      "MeshGateway": {},
      "Expose": {}
    },
    "ServiceConnect": {},
    "CreateIndex": 12,
    "ModifyIndex": 12
  }
]
```

<font style="background-color:rgba(255, 255, 255, 0);">健康检查</font>

```plain
[root@k8s-master ~]# curl -s http://192.168.10.10:8500/v1/health/service/traefik?passing | jq
[
  {
    "Node": {
      "ID": "eda0ef7f-ca12-bbca-c84e-f88cc3664cd1",
      "Node": "tiaoban",
      "Address": "192.168.10.100",
      "Datacenter": "my-dc-1",
      "TaggedAddresses": {
        "lan": "192.168.10.100",
        "lan_ipv4": "192.168.10.100",
        "wan": "192.168.10.100",
        "wan_ipv4": "192.168.10.100"
      },
      "Meta": {
        "consul-network-segment": ""
      },
      "CreateIndex": 9,
      "ModifyIndex": 11
    },
    "Service": {
      "ID": "traefik",
      "Service": "traefik",
      "Tags": [
        "k8s"
      ],
      "Address": "192.168.10.10",
      "TaggedAddresses": {
        "lan_ipv4": {
          "Address": "192.168.10.10",
          "Port": 80
        },
        "wan_ipv4": {
          "Address": "192.168.10.10",
          "Port": 80
        }
      },
      "Meta": null,
      "Port": 80,
      "Weights": {
        "Passing": 1,
        "Warning": 1
      },
      "EnableTagOverride": false,
      "Proxy": {
        "Mode": "",
        "MeshGateway": {},
        "Expose": {}
      },
      "Connect": {},
      "PeerName": "",
      "CreateIndex": 12,
      "ModifyIndex": 12
    },
    "Checks": [
      {
        "Node": "tiaoban",
        "CheckID": "serfHealth",
        "Name": "Serf Health Status",
        "Status": "passing",
        "Notes": "",
        "Output": "Agent alive and reachable",
        "ServiceID": "",
        "ServiceName": "",
        "ServiceTags": [],
        "Type": "",
        "Interval": "",
        "Timeout": "",
        "ExposedPort": 0,
        "Definition": {},
        "CreateIndex": 9,
        "ModifyIndex": 9
      }
    ]
  }
]
```

<font style="background-color:rgba(255, 255, 255, 0);">删除注册的服务</font>

```plain
[root@k8s-master consul.d]# curl -X PUT http://192.168.10.10:8500/v1/agent/service/deregister/traefik
```

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<h2 id="74c3df45"><font style="background-color:rgba(255, 255, 255, 0);">DNS解析</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">DNS接口是Consul中主要的查询接口之一，另一个是HTTP接口， HTTP接口查询请查阅,Consul默认在8600端口监听DNS查询。  
</font><font style="background-color:rgba(255, 255, 255, 0);">参考文档：</font>[<font style="background-color:rgba(255, 255, 255, 0);">https://developer.hashicorp.com/consul/docs/discovery/dns</font>](https://developer.hashicorp.com/consul/docs/discovery/dns)<font style="background-color:rgba(255, 255, 255, 0);">  
</font><font style="background-color:rgba(255, 255, 255, 0);">要使用DNS接口， 有几种方法可以实现：  
</font><font style="background-color:rgba(255, 255, 255, 0);">一是使用指定的DNS解析库， 然后指向Consul；  
</font><font style="background-color:rgba(255, 255, 255, 0);">二是把Consul设置为节点的DNS服务器, 并且提供recursors配置项， 这样非Consul的查询也能被解析；  
</font><font style="background-color:rgba(255, 255, 255, 0);">最后一种方法是从已有的DNS服务器上把所有consul.为域名的请求转发到consul agent上。  
</font>**<font style="background-color:rgba(255, 255, 255, 0);">节点查找</font>**<font style="background-color:rgba(255, 255, 255, 0);">  
</font><font style="background-color:rgba(255, 255, 255, 0);">查找节点的地址信息，查找格式：<node>.node[.datacenter].<domain>。如果datacenter不指定，默认为当前集群查询。</font>

```plain
[root@k8s-master ~]# dig @192.168.10.10 -p 8600 tiaoban.node.consul ANY

; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> @192.168.10.10 -p 8600 tiaoban.node.consul ANY
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25684
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;tiaoban.node.consul.           IN      ANY

;; ANSWER SECTION:
tiaoban.node.consul.    0       IN      A       192.168.10.100
tiaoban.node.consul.    0       IN      TXT     "consul-network-segment="

;; Query time: 14 msec
;; SERVER: 192.168.10.10#8600(192.168.10.10)
;; WHEN: 一 10月 31 10:51:10 CST 2022
;; MSG SIZE  rcvd: 100
```

**<font style="background-color:rgba(255, 255, 255, 0);">服务查找</font>**<font style="background-color:rgba(255, 255, 255, 0);">  
</font><font style="background-color:rgba(255, 255, 255, 0);">查询服务提供者。服务查询支持两种查找方法：标准和严格RFC 2782。  
</font><font style="background-color:rgba(255, 255, 255, 0);">标准查找格式：[tag.]<service>.service[.datacenter].<domain>。Tag是可选的，而且与节点查找一样，数据中心也是可选。如果没有提供Tag，就不会有过滤，如果没有数据中心，就会选择默认的数据中心。</font>

```plain
[root@k8s-master ~]# dig @192.168.10.10 -p 8600 traefik.service.consul SRV

; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> @192.168.10.10 -p 8600 traefik.service.consul SRV
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28200
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 3
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;traefik.service.consul.                IN      SRV

;; ANSWER SECTION:
traefik.service.consul. 0       IN      SRV     1 1 80 c0a80a0a.addr.my-dc-1.consul.

;; ADDITIONAL SECTION:
c0a80a0a.addr.my-dc-1.consul. 0 IN      A       192.168.10.10
tiaoban.node.my-dc-1.consul. 0  IN      TXT     "consul-network-segment="

;; Query time: 13 msec
;; SERVER: 192.168.10.10#8600(192.168.10.10)
;; WHEN: 一 10月 31 10:53:30 CST 2022
;; MSG SIZE  rcvd: 164
```

<font style="background-color:rgba(255, 255, 255, 0);">RFC2782 查找 格式：</font>_<font style="background-color:rgba(255, 255, 255, 0);"><service>.</font>_<font style="background-color:rgba(255, 255, 255, 0);"><protocol>.service[.datacenter][.domain]根据RFC 2782， SRV请求都应该在service和protocol前使用(_)作为前缀。避免发生DNS冲突。Protocol可以是service任何一个tag，如果service没有tag，使用tcp作为protocol。如果一旦设置了tcp，那么查询时将不会执行任何标签过滤。</font>

```plain
# 查询tag为k8s的注册服务traefik信息。
[root@k8s-master ~]# dig @192.168.10.10 -p 8600 _traefik._k8s.service.consul SRV

; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> @192.168.10.10 -p 8600 _traefik._k8s.service.consul SRV
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43270
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 3
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;_traefik._k8s.service.consul.  IN      SRV

;; ANSWER SECTION:
_traefik._k8s.service.consul. 0 IN      SRV     1 1 80 c0a80a0a.addr.my-dc-1.consul.

;; ADDITIONAL SECTION:
c0a80a0a.addr.my-dc-1.consul. 0 IN      A       192.168.10.10
tiaoban.node.my-dc-1.consul. 0  IN      TXT     "consul-network-segment="

;; Query time: 1 msec
;; SERVER: 192.168.10.10#8600(192.168.10.10)
;; WHEN: 一 10月 31 10:57:05 CST 2022
;; MSG SIZE  rcvd: 170
```

<h1 id="bf9a8ea1"><font style="background-color:rgba(255, 255, 255, 0);">配置kv</font></h1>
---

<font style="background-color:rgba(255, 255, 255, 0);">consul kv 是consul的核心功能，并随consul agent一起安装。consul kv允许用户存储索引对象，尽管其主要用途是存储配置参数和元数据。  
</font><font style="background-color:rgba(255, 255, 255, 0);">consul kv 数据存储在server上，可以由任何agent（client或server）访问。consul允许在所有server之间自动复制数据，如果发生故障，拥有一定数量的server将减少数据丢失的风险。  
</font><font style="background-color:rgba(255, 255, 255, 0);">数据存储位于server上的数据目录中，为确保在完全中断的情况下不会丢失数据，可以使用 consul snapshot 命令备份数据。还可以通过 consul kv 子命令、HTTP API 和 Consul UI 访问kv存储。</font>

<h2 id="dbb0d3bb"><font style="background-color:rgba(255, 255, 255, 0);">命令行</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">命令行创建kv</font>

```plain
[root@k8s-master consul.d]# consul kv put key1 value1
Success! Data written to: key1
```

<font style="background-color:rgba(255, 255, 255, 0);">查看kv</font>

```plain
# 查看所有key value数据
[root@k8s-master consul.d]# consul kv get --recurse 
key1:value1
key2:value2
# 查看指定key value数据
[root@k8s-master consul.d]# consul kv get key2
value2
```

<font style="background-color:rgba(255, 255, 255, 0);">更新kv</font>

```plain
[root@k8s-master consul.d]# consul kv put key2 v2
Success! Data written to: key2
```

<font style="background-color:rgba(255, 255, 255, 0);">删除kv</font>

```plain
[root@k8s-master consul.d]# consul kv delete key2
Success! Deleted key: key2
```

<h2 id="web"><font style="background-color:rgba(255, 255, 255, 0);">web</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">web界面创建kv  
</font>![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1738900841944-ff641b3f-de5b-4382-b961-1c30bf857d1b.jpeg)<font style="background-color:rgba(255, 255, 255, 0);">  
</font><font style="background-color:rgba(255, 255, 255, 0);">web界面查看kv  
</font>![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1738900842227-005c67d9-9bd2-4a02-9f36-e4e9aa288c32.jpeg)

<h2 id="http-api-2"><font style="background-color:rgba(255, 255, 255, 0);">HTTP API</font></h2>
```plain
# 设置kv
[root@k8s-master consul.d]# curl -X PUT -d 'value3' http://192.168.10.10:8500/v1/kv/key3
true
# 查看kv
[root@k8s-master consul.d]# curl -s http://192.168.10.10:8500/v1/kv/key3 | jq
[
  {
    "LockIndex": 0,
    "Key": "key3",
    "Flags": 0,
    "Value": "dmFsdWUz",
    "CreateIndex": 611,
    "ModifyIndex": 611
  }
]
# base64解码查看内容
[root@k8s-master consul.d]# echo "dmFsdWUz" | base64 -d
value3
# 删除kv
[root@k8s-master consul.d]# curl -X DELETE http://192.168.10.10:8500/v1/kv/key3
```

<h1 id="f58e1c18"><font style="background-color:rgba(255, 255, 255, 0);">访问控制</font></h1>
---

<font style="background-color:rgba(255, 255, 255, 0);">通过ACLS 来确保安全的访问 UI, API, CLI, servie 通信，Agent通信。如果想要确保数据中心安全，就需要配置ACLS。ACL核心原理是，将规则分组为策略， 然后一个或多个策略于令牌关联。</font>

<h2 id="8facd89c"><font style="background-color:rgba(255, 255, 255, 0);">启用ACL</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">修改consul配置文件，新增如下内容</font>

```plain
# acl访问控制
acl = {
  enabled = true
  default_policy = "deny" # 默认拒绝所有操作
  enable_token_persistence = true # 持久化到磁盘，重启时重新加载
}
```

<font style="background-color:rgba(255, 255, 255, 0);">重启consul服务</font>

```plain
[root@k8s-master consul.d]# systemctl restart consul
```

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<font style="background-color:rgba(255, 255, 255, 0);">生成token</font>

```plain
[root@k8s-master consul.d]# consul acl bootstrap
AccessorID:       16a37577-f243-94fb-8770-35489870025c
SecretID:         54a3e3fd-ea07-85a8-67e3-33107a958977
Description:      Bootstrap Token (Global Management)
Local:            false
Create Time:      2022-10-27 22:56:07.654230262 +0800 CST
Policies:
   00000000-0000-0000-0000-000000000001 - global-management
```

<h2 id="046e223e"><font style="background-color:rgba(255, 255, 255, 0);">访问验证</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">web页面访问验证  
</font>![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1738900842237-eb42da7b-f34d-499d-9861-dd3bb7290224.jpeg)<font style="background-color:rgba(255, 255, 255, 0);">  
</font><font style="background-color:rgba(255, 255, 255, 0);">api接口访问验证</font>

```plain
# 直接获取kv提示权限拒绝
[root@k8s-master consul.d]# curl -s http://192.168.10.10:8500/v1/kv/key3 
rpc error making call: Permission denied: token with AccessorID '00000000-0000-0000-0000-000000000002' lacks permission 'key:read' on "key3"[root@k8s-master consul.d]# 
# 请求头添加token访问
[root@k8s-master consul.d]# curl -s -H "X-Consul-Token:54a3e3fd-ea07-85a8-67e3-33107a958977" http://192.168.10.10:8500/v1/kv/key3 | jq
[
  {
    "LockIndex": 0,
    "Key": "key3",
    "Flags": 0,
    "Value": "dmFsdWUz",
    "CreateIndex": 611,
    "ModifyIndex": 611
  }
]
```

