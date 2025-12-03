# RDMA交换机-独有的RocE功能 本篇介绍全闪存存储(RoCE)配置指导
1.RoCE功能介绍
全闪存存储 RoCE 功能，基于 pfc 和 ecn 实现，pfc 可以逐跳提供基于优先级的流量控制，不影
响其他优先级的流量，作用于点对点；而 ecn 可以在设备发生拥塞时，对报文 IP 头中 ECN 域进
行标记，接收端收到后会向发送端发出降低发送速率的 CNP（Congestion Notification Packet，拥
塞通知报文）包，作用于端到端。

知识补充：
1.什么是RoCE

**RDMA技术**通过 **Infiniband** 和 **RoCE** 落地
为了实现RDMA技术的落地，需要在 **硬件** 和 **软件** 上进行改造
  
    1.1软件上：
  
      1.1.1需要定义新的技术规范，以取代传统的TCP/IP网络协议规范，于是制定了**RDMA verbs**，他是由“IB贸易协会（IBTA，Infiniband Trade Association）”组织制定的一组规范，**用于定义RDMA传输过程中应支持的特性和行为**    
      1.1.2在此基础上，“开放结构联盟”组织（OFA，Openfabrics Alliance）**基于Verbs的定义**，推动**实现了RDMA的应用编程接口**————>**RDMA Verbs API**：允许应用程序可以无视底层硬件和协议的差异，用于替代传统TCP/IP网络协议的套接字API
<img width="1515" height="714" alt="9faec1836c966d4a6ccc172241126417" src="https://github.com/user-attachments/assets/2f2d8e0d-57c5-4246-889d-155e2302e09e" />
  
    1.2硬件上：
  
      1.2.1服务器需要使用支持RDMA的网络适配器以取代传统的网络适配器，RDMA的网络适配器是RDMA技术的主要承载者，有多种不同的协议，eg：A.InfiniBand B.RoCE C.iWARP
           RNIC所支持的协议不同，其专用名称也不同，
           1>支持A+B协议的RDMA的网络适配器可被称为“主机通道适配器”（HCA，Host Channel Adapter）
           2>仅支持协议A的RDMA的网络适配器，被称为“IB适配器”（InfiniBand Adapter）、“IB网卡”、“IB卡”
           3>仅支持协议C的RDMA的网络适配器，被称为“可感知RDMA的网络接口控制器”（RNIC）
<img width="1789" height="640" alt="1ec50cade9102d0fda43215a6d0e4e73" src="https://github.com/user-attachments/assets/b6913b80-df2e-431c-8192-66afa722c0a7" />


2.PFC：

中文：优先级流控（Priority Flow Control）

大白话定义：给 RoCE 流量分配一个专属车道（队列），当这个车道快堵死时，给上游设备发 “暂停信号”，让上游先别发 RoCE 包，等车道有空位了再恢复 ——只停 RoCE 的包，不影响其他普通流量。

关键特点：
只针对**特定优先级的队列**（比如 RoCE 专属的队列 3），不影响其他队列的流量（比如浏览网页的流量）；
是 “硬暂停”：收到暂停信号就停，没信号再发，简单直接；
解决的是**已经堵死**的紧急情况，避免包丢失。

3.ECN：

中文：显式拥塞通知（Explicit Congestion Notification）

大白话定义：在 RoCE 包还没堵死之前，提前给包 “贴个拥堵标签”，告诉发送方 “前面快堵了，你慢点开”，让发送方主动减少发包速度 ——不暂停，只是减速，更灵活。

3. 关键特点
 “提前预警”：不等堵死就行动，比 PFC 更主动，不浪费带宽；
 “贴标签”，不丢包、不暂停，传输更流畅；
需要发送方配合：发送方看到标签后要主动减速（RoCE 协议本身支持这个逻辑，不用额外配置）。
