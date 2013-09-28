---
layout: blog
title: iOS中的线性布局
category: tech
excerpt: XHFLinearView 的使用
--- 

做iOS开发有几个月了，在开发的过程中，发现一个比较奇怪的问题，就是iOS里面几乎没有什么布局方式。
相比之下Android有LinearLayout，FrameLayout，AbsoluteLayout,RelativeLayout,TableLayout 等等。
因此一开始还是不太适应，老是想找一下有没有一些布局方式可以拿来用用，提高一下效率。

做了一段时间后，开始感受到iOS为什么没有这么多布局了。

iOS可以说只有一个相对布局，所有的View都是直接计算出相对parent的坐标，然后写死。
把坐标写死，一开始总是感觉是不是太死板了，万一有个变动，会不会导致界面错乱呢。
实际经验表明，其实不会的。
原因很简单，iOS就那么几个尺寸，而且apple对开发者的设计非常友好，对开发者而言，我们几乎只需要面对320x480一个尺寸就可以了。

正是因为我们面对的手机只有一个尺寸，使用相对布局变成了最简单的布局方式。我们不需要为了适配各种尺寸而设计出各种各样的布局方式。
从这个角度来说，Android搞出这个多布局方式，恰恰是为了适配各种各样的屏幕尺寸。这些布局方式不仅降低了程序员的工作效率，将代码从简单变成复杂，
复杂的布局层次关系和尺寸的动态计算也降低了界面绘制的性能。

了解到这些后，最开始的疑惑自然就解开了。
那既然iOS下使用相对布局就是最简单的方式，还有必要存在其他的布局方式吗？
我觉得这并不是对立的。
使用相对布局是最原生的，直观的，最容易理解的。但有些时候为了让代码更加简洁可读，让程序更加优雅。使用一些布局方式来维护界面仍然是非常好的选择。

工作中便遇到一例。
我们需要做一个比较复杂的页面，页面中有好多模块，依次往下放置。这些界面有时还需要变动大小，改变内容等等。
使用相对布局，我们计算每一个模块相对左上角的坐标。而且因为有时模块会变动大小，当变动时我们需要重新计算所有模块相对坐标的位置。
虽然这不是技术上的难题，但却是让代码变得很糟糕。糟糕的代码可读性和可和维护性都比较差，也更容易出bug。

根据之前的经验，我们都采用TableView来处理。
TableView解决了动态计算相对坐标的问题。(TableView自身来处理，我们不再需要关心)
但却带来了代码结构混乱的问题。
为了构建一个TableViewCell，我们需要在heightForRowAtIndexPath 里声明一个模块的高度，在cellForRowAtIndexPath里构建这个模块的内容。
而且同一个heightForRowAtIndexPath里面需要包含所有模块的高度的声明，同一个cellForRowAtIndexPath里面需要构建所有模块的内容。
一个模块原本应该是独立的，代码却分散到2个地方。
模块之间原本是解耦的，却被要求揉成一团。

其实这个地方最合理的方式是使用线性布局容器。线性布局虽然没有了像在Android里面的重要度，但是在iOS部分场景下还是有一些用处的。

因为iOS并没有好的线性布局容易，我便自己写了一个，取名  XHFLinearView ，github地址：<a target="_blank" href="http://github.com/xuhengfei/XHFLinearView">http://github.com/xuhengfei/XHFLinearView</a>

这个XHFLinearView实现了基本的线性布局要求，同时也支持一些变化的动画过度，比如插入，删除，变更大小等等。

给出代码使用示例：

```objective-c
    XHFLinearView *linearView=[[XHFLinearView alloc]initWithFrame:self.view.bounds];
    [self.view addSubview:linearView];
    //init linear view content views
    [linearView.dataSource addObject:XHFLinearViewUnitMake(someView, XHFMarginMake(0, 0, 0, 0))];
    //force layout linear view
    [linearView needLayout];
    //insert a view with animation
    [linearView insertItem:someView margin:XHFMarginMake(0, 0, 0, 0) atIndex:0 withAnimation:UITableViewRowAnimationFade];
    //replace a view with animation
    [linearView replaceItem:someView withNewItem:newView withAnimation:UITableViewRowAnimationFade];
    //resize a view with animation
    someView.frame=xxx;
    [linearView needLayoutForItem:someView];
    //remove a view with animation
    [linearView removeItemByIndex:0 withAnimation:UITableViewRowAnimationFade];
    
```

<img src="http://xuhengfei.com/assets/images/articles/2013-09-28-linearview.png" width="240"/>

更多内容还请下载<a target="_blank" href="http://github.com/xuhengfei/XHFLinearView">源码</a>自己体验一下
