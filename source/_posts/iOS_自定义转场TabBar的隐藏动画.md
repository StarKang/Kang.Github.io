---
title: iOS_自定义转场TabBar的隐藏动画
date: 2018-03-07 15:45:56
tags: iOS开发
---
- 摘要
实现同iOS 11 App Store Today相似的转场效果.本文尝试解决转场(Transition)过程中TabBar的隐藏(向下滑出屏幕).

<!-- more -->

![tabbar_ani_slow.gif](http://upload-images.jianshu.io/upload_images/1908138-eea03cb294e62ccb.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



##### 遇到的问题
项目中使用的是系统自带的Storyboard上的TabBar,TabBar的隐藏使用的是`hidesBottomBarWhenPushed`方法.实现自定义转场的过程中,TabBar会跟随转场动画向左移出屏幕,但需求是向下滑出屏幕.如果自定义TabBar可能更容易实现,具体效果可以点击App Store上的"今日 App".
##### 解决的问题
仅在自定义转场中按需隐藏TabBar(push向下滑出屏幕,pop向上划入屏幕),不影响其他使用系统转场的页面.
###### 解决方案
- 使用方法
首先确定是否使用的自定义转场,如果定义了自定义转场的类,在要使用的页面实现`UINavigationControllerDelegate`的代理方法
```
-(id<UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                 animationControllerForOperation:(UINavigationControllerOperation)operation
                                              fromViewController:(UIViewController *)fromVC
                                                toViewController:(UIViewController *)toVC 
{
    //根据类型返回对应动画
    if (operation == UINavigationControllerOperationPush) {
        return _trans;
    }else {
        return nil;
    }   
}
```
在我的工程中,存在着重写的`TabBar`、`UITabBarController`、`UINavigationController`的类分别实现不同的需求,要实现这种新增加的效果,最好的方法应该是使用`Category`,所以我们定义`UITabBar`的类别`UITabBar+CustomTabbar`,实现动画(出处作者称为淡入淡出效果).
###### UITabBar+CustomTabbar.h
```
#import <UIKit/UIKit.h>

@interface UITabBar (HYCustomTabBar)<CAAction>

@end
```
###### UITabBar+CustomTabbar.m
```
#import "UITabBar+CustomTabbar.h"
 
@implementation UITabBar (CustomTabbar)
-(id<CAAction>)actionForLayer:(CALayer *)layer forKey:(NSString *)event{
    if ([event isEqualToString:@"position"]) {
        if(layer.position.x<0){
            //show tabbar
            CATransition *pushFromTop = [[CATransition alloc] init];
            pushFromTop.duration = 0.3;
            pushFromTop.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];
            pushFromTop.type = kCATransitionPush;
            pushFromTop.subtype = kCATransitionFromTop;
            return pushFromTop;
        }else if (layer.position.x>0&&(layer.position.y>layer.bounds.size.height)&&(layer.position.y<[UIScreen mainScreen].bounds.size.height)){
            //hide tabbar
            CATransition *pushFromBottom = [[CATransition alloc] init];
            pushFromBottom.duration = 0.3;
            pushFromBottom.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];
            pushFromBottom.type = kCATransitionPush;
            pushFromBottom.subtype = kCATransitionFromBottom;
            return pushFromBottom;
        }
    }
    return (id<CAAction>)[NSNull null];
}
 
-(void)runActionForKey:(NSString *)event object:(id)anObject arguments:(NSDictionary *)dict{
    
}
@end
```
其中
```
layer.position.y>layer.bounds.size.height
layer.position.y<[UIScreen mainScreen].bounds.size.height
```
是为了去除第一次加载视图的情况。
然后在自己重写的`UITabBarController`类中添加类别的引用
```
#import "TabVC.h"
#import "UITabBar+CustomTabbar.h"

@interface TabVC ()

@end
  ...
```
- 这样,在自定义转场过程中,就能实现自定义的TabBar隐藏动效,而且实现了很大程度的解耦.
###### 出处
贴出处[自定义hidesBottomBarWhenPushed动画](http://www.626code.com/2014/12/%E8%87%AA%E5%AE%9A%E4%B9%89hidesbottombarwhenpushed%E5%8A%A8%E7%94%BB/),如侵删

 

