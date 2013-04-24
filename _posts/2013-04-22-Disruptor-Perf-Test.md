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
Disruptor：3.0.1  
#### 2. 测试方法：  
每种类型写操作100，000，000次，buffer_size 36536  
测试场景分别：单生产者单消费者，单生产者多消费者，多生产者单消费者，多生产者多消费者  
每个场景分别测试10次去平均值  

#### 3. 测试数据  
1.单生产者单消费者1P1C测试  
<table class="table table-bordered table-striped table-condensed">
	<tr>
		<td>方案</td>
		<td>Disruptor</td>
		<td>Array</td>
		<td>Linked</td>
		<td>Sync</td>
	</tr>
	<tr>
		<td>1P1C</td>
		<td>8355.4373</td>
		<td>449.445</td>
		<td>126.493</td>
		<td>529.5291</td>
	</tr>
</table> 

<img src="/assets/images/articles/disruptor/1p1c.png" />  
场景分析：  
Disruptor吞吐量比Queue高出20倍左右  
关键原因应该是Queue在读写时用到了锁，而Disruptor没有使用锁  

2.单生产者多消费者1P多C测试   
<table class="table table-bordered table-striped table-condensed">
	<tr>
		<td>方案</td>
		<td>Disruptor</td>
		<td>Array</td>
		<td>Linked</td>
		<td>Sync</td>
	</tr>
	<tr>
		<td>2C</td>
		<td>7687.2447</td>
		<td>169.0777</td>
		<td>212.732</td>
		<td>248.2842</td>
	</tr>
	<tr>
		<td>4C</td>
		<td>3295.4343</td>
		<td>24.5359</td>
		<td>21.6052</td>
		<td>3.4345</td>
	</tr>
</table> 

<img src="/assets/images/articles/disruptor/1pnc.png" />  
场景分析：
Disruptor的吞吐量仍然以超高的倍数领先Queue

3.多生产者单消费者  多P单C 测试  
<table class="table table-bordered table-striped table-condensed">
	<tr>
		<td>方案</td>
		<td>Disruptor</td>
		<td>Array</td>
		<td>Linked</td>
		<td>Sync</td>
	</tr>
	<tr>
		<td>2P</td>
		<td>1670.5315</td>
		<td>286.8795</td>
		<td>352.6871</td>
		<td>288.0944</td>
	</tr>
	<tr>
		<td>4P</td>
		<td>1386.071</td>
		<td>190.605</td>
		<td>303.3227</td>
		<td>200.7288 </td>
	</tr>
</table>
<img src="/assets/images/articles/disruptor/np1c.png" />  
场景分析：  
Disruptor仍然高出Queue 5-10倍吞吐量  
4.多生产者多消费者  多P多C 测试
待续  
#### 4. 分析结论  

