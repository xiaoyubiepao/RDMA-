# RDMA交换机-独有的RocE功能 本篇介绍全闪存存储(RoCE)配置指导
1.RoCE功能介绍
全闪存存储 RoCE 功能，基于 pfc 和 ecn 实现，pfc 可以逐跳提供基于优先级的流量控制，不影
响其他优先级的流量，作用于点对点；而 ecn 可以在设备发生拥塞时，对报文 IP 头中 ECN 域进
行标记，接收端收到后会向发送端发出降低发送速率的 CNP（Congestion Notification Packet，拥
塞通知报文）包，作用于端到端。

2.什么是RoCE

**RDMA技术**通过 **Infiniband** 和 **RoCE** 落地
为了实现RDMA技术的落地，需要在 **硬件** 和 **软件** 上进行改造
  
    2.1软件上：
  
      2.1.1需要定义新的技术规范，以取代传统的TCP/IP网络协议规范，于是制定了**RDMA verbs**，他是由“IB贸易协会（IBTA，Infiniband Trade Association）”组织制定的一组规范，**用于定义RDMA传输过程中应支持的特性和行为**    
      2.1.2在此基础上，“开放结构联盟”组织（OFA，Openfabrics Alliance）**基于Verbs的定义**，推动**实现了RDMA的应用编程接口**————>**RDMA Verbs API**：允许应用程序可以无视底层硬件和协议的差异，用于替代传统TCP/IP网络协议的套接字API
<img width="1515" height="714" alt="9faec1836c966d4a6ccc172241126417" src="https://github.com/user-attachments/assets/2f2d8e0d-57c5-4246-889d-155e2302e09e" />
  
    2.2硬件上：
  
      2.2.1服务器需要使用支持RDMA的网络适配器以取代传统的网络适配器，RDMA的网络适配器是RDMA技术的主要承载者，有多种不同的协议，eg：A.InfiniBand B.RoCE C.iWARP
           RNIC所支持的协议不同，其专用名称也不同，
           1>支持A+B协议的RDMA的网络适配器可被称为“主机通道适配器”（HCA，Host Channel Adapter）
           2>仅支持协议A的RDMA的网络适配器，被称为“IB适配器”（InfiniBand Adapter）、“IB网卡”、“IB卡”
           3>仅支持协议C的RDMA的网络适配器，被称为“可感知RDMA的网络接口控制器”（RNIC）
<img width="1789" height="640" alt="1ec50cade9102d0fda43215a6d0e4e73" src="https://github.com/user-attachments/assets/b6913b80-df2e-431c-8192-66afa722c0a7" />
