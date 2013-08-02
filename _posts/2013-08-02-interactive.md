---
layout: blog
title: iOS Interactive框架
category: tech
excerpt: 基于HTTP的网络层框架，设计优雅，令人陶醉
---

Interactive是一个iOS上基础的网络层框架

框架本身提供了5个核心协议，以及基础的一套实现方案

框架提供一个巨大的扩展与增强空间，使用者可以在系统的实现方案上进行扩展，自己实现核心协议。

特性： 

***Simple***：使用Interactive来进行Http请求处理将会变得非常简单

***Mock***：Interactive内置支持数据的mock，可直接mock业务数据对象，而非仅仅是http返回的String

***Reusable and Transferable*** ：Interactive将请求和返回包装成一个Api，方便进行重用与移植

***Plugin***：内置插件机制，可对请求进行前置后置处理，对调用者透明

***Expansibility***：极强扩展性，实现核心协议即可构建出属于自己的小框架

Interactive有4个核心协议，其相互关系如下图所示：

<img src="/assets/images/interaceive/protocols.png"/>

XHFApi (下面简称：Api)

```objective-c
@protocol XHFApi <NSObject>
@required
//provide a request base on ASIHTTPRequest  
-(ASIHTTPRequest*) getHttpRequest;
//provide a parser which need to implement XHFResponseParser
-(id<XHFResponseParser>) getResponseParser;

@optional
//mock Api response,not just string ,but business object
-(id)mock;
@end
```
Api 主要有2个方法，分别是：构建Http请求，构建Http响应的解析器  
还有一个mock方法，实现此方法即可对这个请求进行mock行为。  
注意mock返回的类型是id，不是String，这里返回的id类型应当是解析rsponse后的业务对象  

XHFExecutor (下面简称：Executor)

```objective-c
@protocol XHFExecutor <NSObject>
@required
//execute a Api with Sync Mode
-(id)execute:(id<XHFApi>)api exception:(NSException *__autoreleasing*)exception;
//execute a Api with Async Mode and callback in MainThread
-(id<XHFHandler>)execute:(id<XHFApi>)api completeOnMainThread:(CompleteCallback)callback;
@end
```
Executor就是一个执行器，负责将一个Api发送出去，并且将Http返回的数据进行解析，最终返回解析完成的业务对象

有了上面2个协议的介绍，我们可以给出一段伪代码，方便理解用法

```objective-c
SomeApi *api=[[SomeApi alloc]initWithUrl:@"http://www.google.com"];
SimpleExecutor *executor=[[SimpleExecutor alloc]init];
__autoreleasing NSException *exception=nil;
id result=[executor execute:api exception:&exception];
if(exception!=nil){
      //Business Code
}
```
我们的使用方法，基本分为2步。1：构造一个Api  2.执行这个Api  

XHFHandler (下面简称：Handler)

```objective-c
@protocol XHFHandler <NSObject>

-(BOOL)isCanceled;

-(BOOL)isDone;

-(void)cancel;

@end
```
Handler是Executor在执行异步请求时返回的一个句柄，
持有了这个Handler，就可以对这个请求进行取消操作。
同时也可以追踪这个请求的一些状态信息。

XHFExecutorPlugin (下面简称：Plugin)

```objective-c
@protocol XHFExecutorPlugin <NSObject>
@optional
//this will be invoke before the request send
-(id)preProcess:(ASIHTTPRequest *)request context:(XHFExecutorContext *)context api:(id<XHFApi>)api;
//this will be invoke after the response has received
-(id)postProcess:(ASIHTTPRequest *)request context:(XHFExecutorContext *)context api:(id<XHFApi>)api;

@end
```
Plugin是对Executor的扩展增强，我们可以在execute前后对Api或者请求进行处理。
典型的用法就是：前置处理可以用来做缓存，后置处理可以用来做重登陆处理

***

以上是4个核心协议的介绍

Interactive对上述的协议做了一个简单通用的实现

XHFExecutor -> XHFSimpleExecutor  
XHFHandler  -> XHFSimpleHandler  

***

基于以上Interactive框架，你可以做很多扩展

比如可以自己实现一个ImageExecutor，专门用来加载图片，内部包含图片缓存等功能

也可以自己去实现一些Plugin插件，可以对所有的Http请求进行过滤行为处理

也可以去实现和继承Api协议，将公共的请求设置成几个基础Api类，然后以子类的方式构建各种业务Api

如果你有能力，也可以完全不用我们提供的XHFSimpleExecutor，和XHFSimpleHandler，完全自己实现一套


github地址：<a href="https://github.com/xuhengfei/iOS-Interactive" >https://github.com/xuhengfei/iOS-Interactive</a>



