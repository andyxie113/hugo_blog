---
title: "RocketMQ出现system busy、broker busy原因分析与解决"                         
author: "雨吁"  
description : "RocketMQ使用者反馈在消息发送过程中偶尔会出现如下错误信息"    
date: 2018-09-18        
lastmod: 2018-09-26             

tags : [                                    
"rocketmq",
]
categories : [                              
"rocketmq",
]
keywords : [                                
"rocketmq",
]
---
RocketMQ使用者反馈在消息发送过程中偶尔会出现如下错误信息：

- [REJECTREQUEST]system busy, start flow control for a while
- [PC_SYNCHRONIZED]broker busy, start flow control for a while
- [PCBUSY_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: %sms, size of queue: %d
- too many requests and system thread pool busy, RejectedExecutionException
- [TIMEOUT_CLEAN_QUEUE]broker busy

#### 原因：

消息发送时抛出system busy、broker busy错误，其本质是系统的PageCache繁忙，通俗一点讲就是向PageCache追加消息时，单个消息发送占用的时间超过1s了。

1. osPageCacheBusyTimeOutMills + broker busy

   pagecache压力较大

   判断pagecache是否忙的依据就是在写入消息时，在向内存追加消息时加锁的时间，默认的判断标准是加锁时间超过1s，就认为是pagecache压力大，向客户端抛出相关的错误日志。

   **优化: 理论上可以把osPageCacheBusyTimeOutMills的值调大一点，但是不推荐，特别是对时效敏感的系统**

2. sendThreadPoolQueueCapacity + system busy

   发送线程池挤压的拒绝策略

   简单说所有到达Broker的请求会被转入到一个线程继续，这个线程的长度由sendThreadPoolQueueCapacity决定，默认长度为10000，超过就报system busy

   **优化**：理论上可以增加该参数防止报异常，但是不建议，治标不治本

3. brokerFastFailureEnable + broker busy

   Broker端快速失败策略

   brokerFastFailureEnable默认为true，表示开启快速失败策略

   如果brokerFastFailureEnable=true,当如果发现Broker服务器的PageCache繁忙，如果发现sendThreadPoolQueue队列中不为空，表示还有排队的发送请求在排队等待执行，则直接结束等待，返回broker busy，

#### 优化方向：

##### 方向一：

1. 当Broker服务器自身比较忙的时候，快速失败，并且在接下来的一段时间内会规避该Broker，这样该Broker恢复提供了时间保证，Broker本身的架构是支持分布式水平扩容的，增加Topic的队列数，降低单台Broker服务器的负载，从而避免出现PageCache。

2. 与之扩容对应的，也可以通过对原有Broker进行升配，例如增加内存、把机械盘换成SSD，但这种情况，通常需要重启Broekr服务器，没有扩容来的方便

##### 方向二：

在broker配置文件中将transientStorePoolEnable设置为true。

- 依据： 启用“读写”分离，消息发送时消息先追加到DirectByteBuffer(堆外内存)中，然后在异步刷盘机制下，会将DirectByteBuffer中的内容提交到PageCache，然后刷写到磁盘。消息拉取时，直接从PageCache中拉取，实现了读写分离，减轻了PageCaceh的压力，能从根本上解决该问题。

- 缺点： 会增加数据丢失的可能性，如果Broker JVM进程异常退出，提交到PageCache中的消息是不会丢失的，但存在堆外内存(DirectByteBuffer)中但还未提交到PageCache中的这部分消息，将会丢失。但通常情况下，RocketMQ进程退出的可能性不大。
