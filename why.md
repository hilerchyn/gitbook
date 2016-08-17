### 为什么 Why

Go is a great technology stack for building scalable, web-based, back-end systems for web applications.

对于web应用来说，Go 拥有构建可伸缩的、基于web的后端系统很棒的技术栈。

When you think about building web applications and web APIs, or simply building HTTP servers in Go, does your mind go to the standard net/http package?
Then you have to deal with some common situations like dynamic routing (a.k.a parameterized), security and authentication, real-time communication and many other issues that net/http doesn't solve. 

当你考虑搭建web应用和web API，或者使用Go简单的搭建一个HTTP服务器，你有抱怨过 net/http 标准包吗？你得处理一些常见的情况，像动态路由(如 参数化)、安全和认证、实时通信和许多 net/http 没有解决的问题。

The net/http package is not complete enough to quickly build well-designed back-end web systems. When you realize this, you might be thinking along these lines:

net/http package 不能完全足够的去快速搭建设计良好的后端web系统。当你认识到这点，你也许正在思考以下几点：

- Ok, the net/http package doesn't suit me, but there are so many frameworks, which one will work for me?!
- 好吧，net/http 包不合适，但是有很多框架，对于我来说哪一个会适合呢？
- Each one of them tells me that it is the best. I don't know what to do!
- 他们每一个都说是最好的。我不知道该怎么办了！

##### 真相 The truth

I did some deep research and benchmarks with 'wrk' and 'ab' in order to choose which framework would suit me and my new project. The results, sadly, were really disappointing to me.

为了选出那个框架适合我和我的新项目，我用 "wrk" 和 "ab" 做了一些深入的研究和基准测试。结果，很让人受伤，太另我失望了。

I started wondering if golang wasn't as fast on the web as I had read... but, before I let Golang go and continued to develop with nodejs, I told myself:

我开始怀疑是不是golang在web上不如我阅读中了解到的那么快... 而且，在使用golang之前我一直用nodejs 开发， 我告诉自己：

> '**Makis, don't lose hope, give at least a chance to Golang. Try to build something totally new without basing it off the "slow" code you saw earlier; learn the secrets of this language and make *others* follow your steps!**'.

> '**Makis, 不要失去希望，至少给Golang一个机会。不用你以前看到的那些慢的代码，去建造一些新的东西；学习这个预言的神秘之处，并使 *别人* 跟随你的呃脚步!**'

These are the words I told myself that day [**13 March 2016**]. 

[**2016年3月13日**] 那天我这样告诉自己。

The same day, later the night, I was reading a book about Greek mythology. I saw an ancient goddess' name and was inspired immediately to give a name to this new web framework (which I had already started writing) - **Iris**.

同一天的晚上，我正在一本关于希腊神话的书。我看到一个远古时代的神的名字，被它激发出灵感来，立马给这个新的web框架起了个名字(我已经开始写了) - **Iris**

**Two months later**, I'm writing this intro. 

**两个月以后**, 我正在写这篇介绍。

 I'm still here [because Iris has succeed in being the fastest go web framework](https://github.com/kataras/iris#benchmarks)

![](comment1.png)
![](comment2.png)
![](comment3.png)
![](comment4.png)
![](comment5.png)
![](comment6.png)
![](comment7.png)
![](comment8.png)

![](comment10.png)

![](comment12.png)

![](comment13.png)

![](comment14.png)

![](comment17.png)


![](comment21.png)

![](comment22.png)

![](comment24.png)

![](comment23.png)