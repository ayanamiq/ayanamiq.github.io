---
title: network
tags: 网络
date: 2018-12-09 00:48:47
---

# get和post
- get传输的数据量很小，post较大。
- get安全性教差，post安全性较高。
- get产生一个TCP数据包,post产生两个TCP数据包。
- get地址栏明文传输数据，post通过http header传输。
- get是从服务器上获取数据，post是向服务器传送数据。
[参考地址](https://www.cnblogs.com/logsharing/p/8448446.html)

# patch请求
在HTTP原本的定义中，用于数据上传的方法只有post和put，后来鉴于post和put语义和功能上的不足，又加入了patch方法，post和put方法的差异是显而易见的，而put和patch方法就比较相似，但用法却完全不同。
PATCH方法是新引入的，是对PUT方法的补充，用来对已知资源进行局部更新。patch方法用来更新局部资源，这句话我们该如何理解？
假设我们有一个UserInfo，里面有userId， userName， userGender等10个字段。可你的编辑功能因为需求，在某个特别的页面里只能修改userName，这时候的更新怎么做？
人们通常(为徒省事)把一个包含了修改后userName的完整userInfo对象传给后端，做完整更新。但仔细想想，这种做法感觉有点二，而且真心浪费带宽(纯技术上讲，你不关心带宽那是你土豪)。
于是patch诞生，只传一个userName到指定资源去，表示该请求是一个局部更新，后端仅更新接收到的字段。
而put虽然也是更新资源，但要求前端提供的一定是一个完整的资源对象，理论上说，如果你用了put，但却没有提供完整的UserInfo，那么缺了的那些字段应该被清空
补充一下，PATCH 与 PUT 属性上的一个重要区别还在于：PUT 是幂等的，而 PATCH 不是幂等的。
幂等是一个数学和计算机学概念，在计算机范畴内表示一个操作执行任意次对系统的影响跟一次是相同。
如：POST 方法不是幂等的，若反复执行多次对应的每一次都会创建一个新资源。如果请求超时，则需要回答这一问题：资源是否已经在服务端创建了？能否再重试一次或检查资源列表？而对于幂等方法不存在这一个问题，我们可以放心地多次请求。
最后再补充一句，restful只是标准，标准的意思是如果在大家都依此行事的话，沟通成本会很低，开发效率就高。但并非强制(也没人强制得了)，所以你说在你的程序里把方法名从put改成patch没有任何影响，那是自然，因为你的后端程序并没有按照标准对两个方法做不同处理，它的表现自然是一样的。