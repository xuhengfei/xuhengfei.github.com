---
layout: blog
category: tech
title: 对源码生成程序的一些偏见
excerpt: 公司内部有人投入开发源码生成程序，吐槽一下
---

公司部门中有人在投入精力开发一个源码生成工具，为了提高生产效率，快速产出。  
个人对源码生成程序有些偏见，于是就整理了一下思路，写了一堆缺点  

所有的源码生成方案都存在以下的几个难点缺点：
###1.源码生成程序做到灵活健壮很难  
这是一个技术上的问题，写出一个完善的源码生成程序，需要投入大量的精力，尝试各种用例测试，考虑各种特殊情况。  并且产出的结果需要有良好的品质。  
我预测实际情况应该会这样：  
写出一个简单版的源码生成程序，用户使用后发现有问题->反馈->改进->发现问题->反馈....  
将会进入如此一个循环，在循环中不断改进。  
这里将会存在一个很严重的问题： 我们做这个源码生成工具，是为了做这个工具呢？还是为了提高工作效率呢？  
当我们把大量的时间放在了反馈和等待中时，还不如手工写一把源码来的更简单呢。  

###2.学习源码生成程序是一种新的成本  
手写源码，是所有程序员必备的基础知识，可认为没有学习成本  
使用了源码生成程序后，所有人都需要学习源码生成程序的使用方法和注意事项。  
一份源码直接手写，可能需要5分钟，学习源码生成程序的使用，估计要超过30分钟。  
而且很可能由于难点1的存在，导致这个源码的生成程序不断在改变，需要不断的学习。  

###3.生成的源码不可修改  
非常常见的需求，一个接口进行了升级，添加了一个新的数据字段  
原本你看到源码后，3分钟时间便可以修改完成了。  
但是因为这个是自动生成的源码，你需要去寻找生成源码的源文件，修改源文件，再使用源文件来生成源码。  
这个流程就远远不止3分钟了。  
而且很可能这份源码当初并不是你写的，你需要找相关人员来咨询，了解源文件的存放位置，修改的注意事项等等。中间加了沟通环节，效率将会大大降低。  

###4.生成的源码没有可扩展性  
很多时候我们需要对源码进行一下修改或者扩展。  
比如原始字段是String类型的True和False，为了让程序更加优美，我们希望在代码中转换成BOOL类型的YES和NO。  
这种类型case发生在自动生成的源码中是无解的。  
我们要么忍受他的不合理之处，要么用辗转的方式来解决这个问题。  


总结一下我的观点  
高品质的代码必然是需要手工写出来的。不能指望通过源码生成工具来产生高品质的代码。  
源码生成程序适用于解决重复而又没有技术含量的劳动。  
但是当你遇到经常需要进行重复而又没有技术含量的劳动时，第一反应不应当是做一个源码生成工具。而应当考虑为什么会出现需要重复而又没有技术含量的劳动。  
因为大部分时候，出现这种情况本身就是不合理的。  
比如需求的反复变动，推倒重来。 这种情况我们需要做的是和需求方讨论为什么需求如此不稳定，能否充分讨论验证后再开工。  
只有当确认这种重复而没有技术含量的劳动是合理的，才能再来考虑是否需要做一个源码生成工具。  
毕竟做出这个源码生成工具是需要付出精力的，而且是由代价的。上面说到的缺点就是代价。  
盲目的投入开发源码生成工具，很可能让整个开发的效率更低下，适得其反。  

