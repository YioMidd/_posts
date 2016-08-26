---
title: 同一台设备如何使用两个GitHub帐号？
date: 2016-08-26 18:26:19
tags:
---
   <div align=center>
   ![Paste_Image.png](http://s5.51cto.com/wyfs02/M02/76/C5/wKioL1ZcE9LyccTbAAC_FRvn6-U778.jpg)</div>
 
**假设你跟我一样有在同台电脑配置两个Git帐号的需求，但同时也不太清楚怎么操作，也许本文章可以帮助到你。**

> 文章所讲的是在Mac系统下，Windows下的朋友也可以参考，整体流程都一样，只不过某些文件的路径不同系统下有所不同而已。

<!-- more -->

## 前言
这几天突然想要积极点、活跃点的混迹GitHub -. - 但又看了下之前的git帐号，昵称起的太不合心意了，并且这个帐号已经添加到公司的git teams里面，想去修改昵称怕又搞出什么幺蛾子，一种申请一个新的私人帐号想法油然而生。
于是乎，趁着下班空闲时间，开干！

## 配置SSH Key
到这里，假设你手上已有两个Git帐号，一个原先设备使用着，另一个是新申请的，这里以我自己的两个帐号为例子来进行讲解

``canny09@qq.com  这是旧的Git帐号``

``listen_kb@163.com  这是新的Git帐号``

你肯定知道，使用这个命令 ``ssh-keygen -t rsa -C “你的邮箱”``来生成``SSH Key``，但假设你之前是通过百度按着教程步骤来的，通常教程里都会说，输入这个命令之后，连敲3次回车就好了，出现下面这样，就表示``SSH Key``成功了
   <div align=center>
   ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/935058-ba95c44569172de1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>
   
   但这次你用次命令该命令来生成``SSH Key``的时候，在``Enter file in which to save the key (/Users/kiben/.ssh/id_rsa):``这里按第一个回车的时候，不要按回车，因为按了回车，就会生成默认的``id_rsa``跟``id_rsa.pub``两个文件，也因此会覆盖你原先旧帐号在``/Users/你的电脑用户名/.ssh/``文件夹的两个同名文件，也导致你原先帐号的``SSH Key``失效。 因此，在Terminal输入你新帐号的SSH Key文件的名称，就像这样：
   <div align=center>
   ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/935058-327ba0bbdfabea7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>
   剩下的也是敲两个回车就行了，我们不给改文件设置密码，成功之后，你会在上面提到的``.ssh``文件下看到``id_rsa_yiomidd``跟``id_rsa_yiomidd.pub``两个文件，如果你找不到``.ssh``文件夹，可以用以下命令：
   
   ```
   显示 Mac 隐藏文件的命令
   defaults write com.apple.finder AppleShowAllFiles -bool true
   
   然后再输入一句：重启Finder
   killall Finder
   
   隐藏 Mac 隐藏文件的命令
   defaults write com.apple.finder AppleShowAllFiles -bool false
   ```
   
   接着就是在Terminal使用``vim id_rsa_yiomidd.pub``命令打开公钥文件，然后复制里面的所有字符，接着到GitHub 帐号设置那里添加SSH Key，这步骤你肯定也很熟练了，我就不多废话了。
   
## 配置config文件
回到Terminal，接着使用``vi config``创建``config``文件，然后在文件里复制以下内容：

```
#这里是原先使用的帐号(canny09@qq.com)
Host github.com
HostName github.com 
User git 
IdentityFile ~/.ssh/id_rsa

#这里是新的帐号(listen_kb@163.com)
Host YioMidd
HostName github.com 
User git 
IdentityFile ~/.ssh/id_rsa_yiomidd
```

解释下每一行的作用

``Host``设置别名，原先使用的帐号，你使用默认的就好；新的帐号就需要重新命名，名称随便你，好记就行；

``HostName`` 默认就是github.com 不需要去改动

``User`` 用户，也是默认就好

``IdentityFile`` 这个就是你帐号对应的公钥文件了，路径是这样固定的，唯一要改的就是末尾的文件名``id_rsa_yiomidd``改成你自己的

然后保存退出

> 这里要注意并记住，使用旧帐号的时候，之前怎么用还是怎样；在使用新帐号的时候，比如在克隆仓库的时候，本来是这样的
> ``git clone git@github.com:YioMidd/projectName.git``，但现在要改为``git clone YioMidd:YioMidd/projectName.git``

到这里，你可能就觉得已经完成了，可以愉快的使用新帐号了，不过你会发现，使用的时候，不成功，Terminal报错，原因是，你这台设备现在还是使用着原先的旧帐号，所以你要切换帐号。**这也是比较烦的**

控制台``cd ..``会到根目录，``vi .gitconfig``文件，修改里面的``Name``跟``Email``

   <div align=center>
   ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/935058-16d9adbff8979257.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) </div> 
   对应你的git帐号昵称跟注册的邮箱，然后保存退出就行了。
   当然，如果你对git 命令比较熟悉的话，你也可以使用以下命令来进行切换：
   
   ```
   git config --global user.name "你的昵称"
   git config --global user.email "你的邮箱地址"
   ```
   最后还有一种方法，也是最方便的。如果你跟我一样，使用SourceTree来进行Git仓库操作的，在偏好设置里设置一下就好啦
      <div align=center>
   ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/935058-8bab06fcf9870e2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>
 
 到此，就可以愉快地切换工作跟私人的Git帐号了。
 
 
> 感谢阅读，如有纰漏或错误，请批评指正！！
