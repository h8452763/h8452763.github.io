---
layout: post
title: 用JNI实现加密字符串
date: 2017-12-14 21:32:24.000000000 +09:00
---

#### 实现的功能
实现的功能就是将密钥存储在c文件中 ，用c语言的MD5进行加密，将加密的字符串通过JNI调用传送回去，可以实现网络传输的验证

实现起来很简单第一步就是搭建NDK的环境
[配置环境变量](http://www.jianshu.com/p/b55d562fabd7)

搭建好了之后可以新建一个官方提供的demo看看目录结构
[新建项目](http://blog.csdn.net/xiaoyu_93/article/details/53082088)
记得要关联NDK环境
![ndk环境](https://raw.githubusercontent.com/smshen/MarkdownPhotos/master/Res/test.jpg)
这样跑起来之后 可以看见NDK 的样式了具体目录结构参见第一个博客。
然后根据需求编写自己的代码 。
#######需求
实现将需要加密的字符串从java中传递到native层，在native层通过和后台定义好的密钥进行拼接 然后MD5 进行加密，将加密好的字符串从native层传递回java层，
然后打好so包 将so文件放在需要的项目中就行了

MD5的c语言加密是在网上找的，然后将转化之后的字符串转化成我们想要的格式 传输回来就行了附上项目[GitHub地址](https://github.com/h8452763/NDKmd5)



#### License

Great thanks to [Dale Anthony](https://github.com/daleanthony) and his [Uno](https://github.com/daleanthony/uno). Vno Jekyll is based on Uno, and contains a lot of modification on page layout, animation, font and some more things I can not remember. Vno Jekyll is followed with Uno and be licensed as [Creative Commons Attribution 4.0 International](http://creativecommons.org/licenses/by/4.0/). See the link for more information.
