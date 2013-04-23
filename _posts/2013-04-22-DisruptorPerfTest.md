---
layout: blog
excerpt: Disruptor性能测试
---

###Disruptor与Queue的性能测试


#### 1. 测试环境：  
MacbookAir 2GHz Intel Core i7  
Mem：8G 1600 MHz DDR3  
OS：OS X 10.8.2  
JDK: 1.6.0_43  
#### 2. 测试方法：  
每种类型写操作100，000，000次，buffer_size 36536  
测试场景分别：单生产者单消费者，单生产者多消费者，多生产者单消费者，多生产者多消费者  
每个场景分别测试10次去平均值  

#### 3. 测试数据  
1.单生产者单消费者1P1C测试  
Disruptor	ArrayQueue	LinkedQueue	SyncQueue  
8355.4373	449.445	    126.493	    529.5291  

<img src="assets/images/articles/disruptor/1p1c.png" />
场景分析：  
Disruptor吞吐量比Queue高出20倍左右  
关键原因应该是Queue在读写时用到了锁，而Disruptor没有使用锁  

2.单生产者多消费者1P多C测试   
方案	Disruptor	ArrayQueue	LinkedQueue	SyncQueue  
2C	7687.2447	169.0777	212.732	    248.2842  
4C	3295.4343	24.5359	    21.6052	    3.4345  

<img src="assets/images/articles/disruptor/1pnc.png" />
场景分析：
Disruptor的吞吐量仍然以超高的倍数领先Queue

3.多生产者单消费者  多P单C 测试  

4.多生产者多消费者  多P多C 测试

#### 4. 分析结论  