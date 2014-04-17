---
layout: post
title: "iOS屏幕旋转"
date: 2014-04-09 21:07
comments: true
categories: iOS
tags: [Orientation]
---

##### 屏幕旋转背后做了什么？

先来看一段简单的代码：

    // iOS5以前支持
    - (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation
    {
        return YES;
    }
    
    // iOS6以后支持
    - (BOOL)shouldAutorotate
    {
        return YES;
    }
    
    - (NSUInteger)supportedInterfaceOrientations
    {
        return UIInterfaceOrientationMaskAll;
    }
    
    - (void)willRotateToInterfaceOrientation:(UIInterfaceOrientation)toInterfaceOrientation duration:(NSTimeInterval)duration
    {
        NSLog(@"UIViewController willRotateToInterfaceOrientation : %ld", toInterfaceOrientation);
    }
    
    - (void)didRotateFromInterfaceOrientation:(UIInterfaceOrientation)fromInterfaceOrientation
    {
        NSLog(@"UIViewController didRotateFromInterfaceOrientation, view ：%@", self.view);
    }

可以看到代码支持四个方向的旋转，并且在旋转之后打印了相关的信息如下：
    
{% img /images/log.jpeg %} 

设备的初始方向是UIInterfaceOrientationPortrait的，然后顺时针依次经过UIInterfaceOrientationLandscapeLeft，UIInterfaceOrientationPortraitUpsideDown，UIInterfaceOrientationLandscapeRight，最后再回到UIInterfaceOrientationPortrait方向，在旋转的过程中，frame没有变化，Transform一直在变化，因此我们可以怀疑屏幕旋转是通过变化Transform实现的，不用怀疑，其实就是通过view的transform属性实现的。

##### 什么是Transform？

Transform(变化矩阵)是一种3×3的矩阵，如下图所示：

{% img /images/juzhen.jpeg %} 

通过这个矩阵我们可以对一个坐标系统进行缩放，平移，旋转以及这两者的任意组着操作。而且矩阵的操作不具备交换律，即矩阵的操作的顺序不同会导致不同的结果。UIView有个transform的属性，通过设置该属性，我们可以实现调整该view在其superView中的大小和位置。

矩阵实现坐标变化背后的数学知识：  

{% img /images/gongshi.jpeg %} 

　　设x，y分别代表在原坐标系统中的位置，x'，y'代表通过矩阵变化以后在新的系统中的位置。其中式1就是矩阵变化的公式，对式1进行展开以后就可以得到式2。从式2我们可以清楚的看到（x，y）到（x'，y'）的变化关系。

　　1）当c，b，tx，ty都为零时，x' = ax，y' = dy；即a，d就分别代表代表x，y方向上放大的比例；当a，d都为1时，x' = x，y' = y；这个时候这个矩阵也就是传说中的CGAffineTransformIdentity(标准矩阵)。

　　2）当a，d为1，c，b为零的时候，x' = x + tx，y' = y + ty；即tx，ty分别代表x，y方向上的平移距离。

　　3）前面两种情况就可以实现缩放和平移了，那么旋转如何表示呢？

　　假设不做平移和缩放操作，那么从原坐标系中的一点(x，y)旋转α°以后到了新的坐标系中的一点(x'，y')，那么旋转矩阵如下：

{% img /images/jiaodu.jpeg %} 

展开以后就是x' = xcosα - ysinα，y' = xsinα + ycosα；

回过头来看看前面设备旋转时的输出日志，当设备位于Portrait方向的时候由于矩阵是标准矩阵，所以没有进行打印。当转到UIInterfaceOrientationLandscapeLeft方向的时候，我们的设备是顺时针转了90°(逆时针为正，顺时针为负)，这个时候矩阵应该是（cos-90°，sin-90°，-sin-90°，cos-90°，tx，ty），由于未进行平移操作所以tx，ty都为0，刚好可以跟我们控制台输出："UIViewController didRotateFromInterfaceOrientation, view ：<UIView: 0x10922cf40; frame = (0 0; 320 568); transform = [0, -1, 1, 0, 0, 0]; autoresize = RM+BM; layer = <CALayer: 0x10924cd60>>"一致。观察其他两个方向的输出，发现结果均和分析一致。

由此可以发现屏幕旋转其实就是通过view的矩阵变化实现，当设备监测到旋转的时候，会通知当前程序，当前程序再通知程序中的window，window会通知它的rootViewController的，rootViewController对其view的transform进行设置，最终完成旋转。  
　　
如果我们直接将一个view添加到window上，系统将不会帮助我们完成旋操作，这个时候我们就需要自己设置该view的transform来实现旋转了。这种情况虽然比较少，但是也存在的，例如现在很多App做的利用状态栏进行消息提示的功能就是利用自己创建window并且自己设置transform来完成旋转支持。  

加速计是整个IOS屏幕旋转的基础，依赖加速计，设备才可以判断出当前的设备方向，IOS系统共定义了以下七种设备方向：  
　　
    
    typedef NS_ENUM(NSInteger, UIDeviceOrientation) {  
        UIDeviceOrientationUnknown,  
        UIDeviceOrientationPortrait,            // Device oriented vertically, home button on the bottom  
        UIDeviceOrientationPortraitUpsideDown,  // Device oriented vertically, home button on the top  
        UIDeviceOrientationLandscapeLeft,       // Device oriented horizontally, home button on the right  
        UIDeviceOrientationLandscapeRight,      // Device oriented horizontally, home button on the left  
        UIDeviceOrientationFaceUp,              // Device oriented flat, face up  
        UIDeviceOrientationFaceDown             // Device oriented flat, face down  
    };
    
以及如下四种界面方向：  

    typedef NS_ENUM(NSInteger, UIInterfaceOrientation) {  
        UIInterfaceOrientationPortrait           = UIDeviceOrientationPortrait,  
        UIInterfaceOrientationPortraitUpsideDown = UIDeviceOrientationPortraitUpsideDown,  
        UIInterfaceOrientationLandscapeLeft      = UIDeviceOrientationLandscapeRight,  
        UIInterfaceOrientationLandscapeRight     = UIDeviceOrientationLandscapeLeft  
    };

![](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Art/view_orientations_2x.png)
    
当加速计检测到方向变化的时候，会发出[UIDeviceOrientationDidChangeNotification][1]通知，这样任何关心方向变化的view都可以通过注册该通知，在设备方向变化的时候做出相应的响应，在屏幕旋转的时候，UIKit帮助我们做了很多事情，方便我们完成屏幕旋转。  

UIKit的相应屏幕旋转的流程如下：

1. 设备旋转的时候，UIKit接收到旋转事件。
2. UIKit通过AppDelegate通知当前程序的window。
3. Window会知会它的rootViewController，判断该view controller所支持的旋转方向，完成旋转。
4. 如果存在弹出的view controller的话，系统则会根据弹出的view controller，来判断是否要进行旋转。

在响应设备旋转时，我们可以通过UIViewController的方法实现更细粒度的控制，当view controller接收到window传来的方向变化的时候，流程如下:  

1. 首先判断当前viewController是否支持旋转到目标方向，如果支持的话进入流程2，否则此次旋转流程直接结束。
2. 调用 willRotateToInterfaceOrientation:duration: 方法，通知view controller将要旋转到目标方向。如果该viewController是一个container view controller的话，它会继续调用其content view controller的该方法。这个时候我们也可以暂时将一些view隐藏掉，等旋转结束以后在现实出来。
3. window调整显示的view controller的bounds，由于view controller的bounds发生变化，将会触发 viewWillLayoutSubviews 方法。这个时候self.interfaceOrientation和statusBarOrientation方向还是原来的方向。
4. 接着当前view controller的 willAnimateRotationToInterfaceOrientation:duration: 方法将会被调用。系统将会把该方法中执行的所有属性变化放到动animation block中。
5. 执行方向旋转的动画。
6. 最后调用 didRotateFromInterfaceOrientation: 方法，通知view controller旋转动画执行完毕。这个时候我们可以将第二部隐藏的view再显示出来。

整个响应过程如下图所示：

![](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Art/rotation_onestep_2x.png)

iOS横竖屏切换的方案目前有以下几种实现的方法：  

1. 通过AutoResizeMask或者AutoLayout来实现界面的适配。比较简单，不够灵活。  
2. 通过代码，在横竖屏切换的时候，重新调整控件的frame。调试比较麻烦，最灵活。  
3. 横竖屏采用不同的界面，当屏幕切换的时候，显示对应的视图。  

屏幕旋转相关知识点：  

+ iOS6使用supportedInterfaceOrientations 和 shouldAutorotate 2个方法来代替shouldAutorotateToInterfaceOrientation(deprecated)。注意：为了向后兼容iOS 4 and 5，还是需要在你的app里保留shouldAutorotateToInterfaceOrientation。  

+ iOS5和以前SDK, 如果没有重写shouldAutorotateToInterfaceOrientation，那么对于iphone来讲，默认是只支持portrait，不能旋转,iPad默认支持所有方向。  

+ iOS6之后如果没有重写shouldAutorotate和supportedInterfaceOrientations,默认 iphone则是"可以旋转，支持非upside down的方向"， iPad默认支持所有方向。  

+ 在iOS 4 and 5，都是由具体的view controller来决定对应的view的orientation设置；而在iOS 6,则是由top-most controller来决定view的orientation设置。  

+ 当我们的view controller隐藏的时候，设备方向也可能发生变化。例如view Controller A弹出一个全屏的view controller B的时候，由于A完全不可见，所以就接收不到屏幕旋转消息。这个时候如果屏幕方向发生变化，再dismiss B的时候，A的方向就会不正确。我们可以通过在view controller A的viewWillAppear中更新方向来修正这个问题。

+ 屏幕旋转时的一些建议
	* 在旋转过程中，暂时界面操作的响应。
	* 旋转前后，尽量当前显示的位置不变。
	* 对于view层级比较复杂的时候，为了提高效率在旋转开始前使用截图替换当前的view层级，旋转结束后再将原view层级替换回来。
	* 在旋转后最好强制reload tableview，保证在方向变化以后，新的row能够充满全屏。例如对于有些照片展示界面，竖屏只显示一列，但是横屏的时候显示列表界面，这个时候一个界面就会显示更多的元素，此时reload内容就是很有必要的。  

举个例子：app的rootViewController是navigation controller "nav", 在”nav"里的stack依次是：main view -> first view > second view，而main view里有一个button会present modal view "modal view".  
那么for ios 4 and 5，在ipad里，如果你要上述view都仅支持横屏orientation，你需要在上面的main view, first view, second view, model view里都添加下面代码：

	- (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation {  
		return (interfaceOrientation == UIInterfaceOrientationLandscapeLeft || interfaceOrientation==UIInterfaceOrientationLandscapeRight);  
	}  

对于iOS6, 由于是由top-most controller来设置orientation，因此你在main view, first view, second view里添加下面的代码是没有任何效果的，而应该是在nav controller里添加下列代码。而modal view则不是在nav container里，因此你也需要在modal view里也添加下列代码。

    -(NSUInteger)supportedInterfaceOrientations{  
        return UIInterfaceOrientationMaskLandscape;  
    }  
      
    - (BOOL)shouldAutorotate  
    {  
        return YES;  
    }

你需要自定义一个UINavigationController的子类for "nav controller"，这样才可以添加上述代码。和navigation controller类似，tab controller里的各个view的orientation设置应该放在tab controller里。  

对于ios6的top-most controller决定orientation设置，导致这样一个问题：在 top-most controller里的views无法拥有不相同的orientation设置。例如：对于 iphone, 在nav controller里，你有main view, first view 和 second view，前2个都只能打竖，而second view是用来播放video，可以打横打竖。那么在ios 4 and 5里可以通过在main view 和 first view的shouldAutorotateToInterfaceOrientation里设置只能打竖，而在second view的shouldAutorotateToInterfaceOrientation设置打竖打横即可。而在ios 6里则无法实现这种效果，因为在main view, first view和 second view的orientation设置是无效的，只能够在nav controller里设置。那么你可能想着用下列代码在nav controller里控制哪个view打竖，哪个view打横。

    -(NSUInteger)supportedInterfaceOrientations{  
        if([[self topViewController] isKindOfClass:[SubSubView class]])  
            return UIInterfaceOrientationMaskAllButUpsideDown;  
        else  
            return UIInterfaceOrientationMaskPortrait;  
    }

或者在top-most controller中直接获取顶部的ViewController支持的方向，需要在下面层次的ViewController中都设置支持的方向：  

    -(NSUInteger)supportedInterfaceOrientations{  
        return self.topViewController.supportedInterfaceOrientations;
    }

这样可以使得在main view 和 first view里无法打横，而second view横竖都行。但问题来了，如果在second view时打横，然后返回到 first view，那么first view是打横显示的！  

目前想到的解决方法只能是把second view脱离nav controller，以modal view方式来显示。这样就可以在modal view里设置打横打竖，而在nav controller里设置只打竖。  

如果你的app的所有view的orientation的设置是统一的，那么你可以简单的在plist file里设置即可，不用添加上面的代码。而如果你添加了上面的代码，就会覆盖plist里orientation的设置。    

参考：  

- [苹果官网][1]
- [smileEvday][3]
- [jaywon][4]

[1]:https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/RespondingtoDeviceOrientationChanges/RespondingtoDeviceOrientationChanges.html#//apple_ref/doc/uid/TP40007457-CH7-SW1
[2]:https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIDevice_Class/Reference/UIDevice.html#//apple_ref/c/data/UIDeviceOrientationDidChangeNotification
[3]:http://www.cnblogs.com/smileEvday/archive/2013/04/24/3041407.html
[4]:http://blog.csdn.net/jaywon/article/details/8208991