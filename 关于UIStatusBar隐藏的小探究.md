---
title: 关于UIStatusBar隐藏的小探究
date: 2016-08-20 20:13:18
tags:
---
   <div align=center>
   ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/935058-280d99388787e823.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>

最近听说滴滴的折扣很给力，对于之前一直用微信叫车的我，就下了个客户端，随便玩了一下，注意到状态栏那小东西的变化，就去探究了一下，也就有了这篇文章...

> 本人小菜鸟一只，文章也只是当作自己的笔记，并没有复杂高深的内容，大神请忽略飘过~ 哈哈

<!-- more -->
#### UIStatusBar
对于StatusBar，由于其特殊性，苹果对其并未像其他类一样，暴露给我们单独的``h``文件，包含各个``property``以及相关API，就只是在``UIApplication``类提供相关几个属性跟API，所以我们平时对其的操作不外乎更改``Style``或者``Hidden``属性；当然，还有横竖屏转换的问题，这里我就不另作详述了。

#### 隐藏的方式
这里要稍微讲一下，在iOS 6的时候，我们通过修改项目``plist``文件中的``View controller-based status bar appearance`` 的布尔值为**No**（默认为Yes），然后使用以下API来实现对statusBar的隐藏

``` objectivec
- (void)setStatusBarHidden:(BOOL)hidden withAnimation:(UIStatusBarAnimation)animation NS_DEPRECATED_IOS(3_2, 9_0, "Use -[UIViewController prefersStatusBarHidden]")
```

通常我们也是这么干的，通过配置一个Bool值参数传入方法，来实现在需要的时候对statusBar做隐藏或显示，同时也可配置是否伴随动画，类似这样：

``` objectivec
- (void)configeStatusBarHidden:(BOOL)hidden {
    if (hidden) {
        [[UIApplication sharedApplication] setStatusBarHidden:hidden withAnimation:UIStatusBarAnimationSlide];
    }else {
        [[UIApplication sharedApplication] setStatusBarHidden:hidden withAnimation:UIStatusBarAnimationSlide];
    }
}
```
   <div align=center>
   ![article1.gif](http://upload-images.jianshu.io/upload_images/935058-f0a06fa48f75539c.gif?imageMogr2/auto-orient/strip)</div>

这种修改``plist``文件的方式在iOS7、8中也照样可用，不过在iOS9中就报了警告，详情可[戳这里](https://github.com/ChenYilong/iOS9AdaptationTips)，因此，我们应该把``plist``文件中的那个value值改为Yes。并且从上面的API可以看到，iOS9之后，这个API也就``DEPRECATED``，官方也建议我们用``-[UIViewController prefersStatusBarHidden]``来代替，可能有童鞋会说，``DEPRECATED``也可以用呀，就像项目原先支持iOS7+ 使用``UIAlertView``，后来改为支持ios8+ ，无非就多个警告而已，也没事呀. 但这里会些许不同

> 当``plist``文件的value值改为Yes的时候，上面使用的``- (void)setStatusBarHidden:(BOOL)hidden withAnimation:(UIStatusBarAnimation)animation`` 将不起任何作用，反之，``- (BOOL)prefersStatusBarHidden``也会失效；
> 使用新的API来实现隐藏的功能需求

``` objectivec
- (BOOL)prefersStatusBarHidden {
    UIApplication *app = [UIApplication sharedApplication];
    return !app.isStatusBarHidden;
}
- (UIStatusBarAnimation)preferredStatusBarUpdateAnimation {
    return UIStatusBarAnimationSlide;
}
```


但使用两个API虽然可以实现隐藏功能，但我们即使设置了动画，也没看到任何动画效果，也只是闪的一下就没了
   <div align=center>
   ![article2.gif](http://upload-images.jianshu.io/upload_images/935058-39c98b241e57f24e.gif?imageMogr2/auto-orient/strip)</div>

那要怎样才有动画效果呢？ 其实只要你够细心，在看API文档的时候就会发现，在上面那两个API的下面，有这么个方法，
``` objectivec
// This should be called whenever the return values for the view controller's status bar attributes have changed. If it is called from within an animation block, the changes will be animated along with the rest of the animation block.
- (void)setNeedsStatusBarAppearanceUpdate NS_AVAILABLE_IOS(7_0) __TVOS_PROHIBITED;
```

相关的注释说的非常清晰，我们可以把这个方法放在animation block里面来达到动画效果，事不宜迟，赶紧试试
``` objectivec
[UIView animateWithDuration:0.33 animations:^{
        [self setNeedsStatusBarAppearanceUpdate];
    }];
```
   <div align=center>
   ![article3.gif](http://upload-images.jianshu.io/upload_images/935058-fc5603c26a7cfb8d.gif?imageMogr2/auto-orient/strip)</div>

这样，滴滴的那个效果也就可以初步实现了，看下效果
``` objectivec
- (IBAction)jump:(UIButton *)sender {
    sender.tag = !sender.tag;
    sender.tag ? [self showLeftView] : [self hideLeftView];
}
- (void)showLeftView {
    [UIView animateWithDuration:0.33 animations:^{
        self.leftView.transform = CGAffineTransformTranslate(self.leftView.transform, 200, 0);
        [self setNeedsStatusBarAppearanceUpdate];
    }];
}
- (void)hideLeftView {
    [UIView animateWithDuration:0.33 animations:^{
        self.leftView.transform = CGAffineTransformTranslate(self.leftView.transform, -200, 0);
    } completion:^(BOOL finished) {
        [UIView animateWithDuration:0.33 animations:^{
            [self setNeedsStatusBarAppearanceUpdate];
        }];
    }];
｝
```
   <div align=center>
   ![article4.gif](http://upload-images.jianshu.io/upload_images/935058-75b1afffb08684b2.gif?imageMogr2/auto-orient/strip)</div>

OK啦，可以收工了！！👏👏👏 
 
不不不，还没完呢，还有话要说：

**文章中所截图的demo，最外层是有套了一个NavigationController的，然后通过在*viewDidLoad*设置隐藏navigationBar，因为仔细看发现滴滴应该也是隐藏了navigationBar的。其实这种需求还是比较少见的，而且也不适用于在Push界面的时候等；对了，如果你没隐藏navigationBar，会出现这样的情况:**

   <div align=center>
   ![article6.gif](http://upload-images.jianshu.io/upload_images/935058-861575a876ab841c.gif?imageMogr2/auto-orient/strip)</div>



最后，感谢您的阅读，若文章有描述错误，望指正，非常感谢！
