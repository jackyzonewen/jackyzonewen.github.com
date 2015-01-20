---
layout: post
title: "Android动画"
date: 2015-01-20 17:08
comments: true
categories: 
tags: [android, animation]
---

####前言

闲来无事，整理一下Android的动画的学习和使用。

####简介

Android提供了三种动画:

1. 补间动画(Tween):通过对场景里的对象不断做图像变换(平移、缩放、旋转)产生动画效果，即是一种渐变动画，只是在绘制的过程对于矩阵做了变换，注意，只是修改了绘制的效果(缩放、旋转、平移和alpha透明度等)，但是它的实际属性值没有改变，比如你缩小了一个Button的大小，它的有效点击区域却不变，因为它的位置和大小属性没变动。
2. 帧动画(Drawable):连续展示一帧一帧的图片资源(存放在res/drawable)，只能设置帧与帧的间隔时间，像播放电影一样。
3. 属性动画(Property):Android3.0推出,[nineoldandroids][5]可以兼容到3.0以下，对于之前的补间的动画的一个提升，通过动态地改变对象的属性从而达到动画效果，同时支持对非视图对象进行动画。

####补间动画

#####原理

要知道原理，首先需要知道android的视图的层级结构，具体的可以查看相关教程，大概的层次结构如下:  
RootView -> DecorView -> LinearLayout(title+content) -> ··· -> activity_root -> ···  

RootView为视图层级树的顶层视图，RootView只有一个子视图DecorView，DecorView为整个Window界面的最顶层View，DecorView只有一个子视图为LinearLayout，代表整个Window界面，包含通知栏，标题栏，内容显示栏三块区域，通常在Activity中setContentView是设置的内容显示视图。  
ViewTree是以一种自上而下的方式进行遍历实现，Parent总是最先绘制的，其次才是Children，并且仍然遵循自上而下的方式。  

ViewRoot.java中的draw函数准备好Canvas后会调用mView.draw(canvas)，其中mView就是调用ViewRoot.setView时设置的DecorView。  
接着看一下View的绘制过程：  

1. 绘制背景；
2. 如果需要，保存画布（canvas）的层为淡入或淡出做准备；
3. 绘制View本身的内容，通过调用View.onDraw(canvas)函数实现，通过这个我们应该能看出来onDraw 函数重载的重要性，onDraw函数中绘制线条/圆/文字等功能会调用Canvas中对应的功能；
4. 绘制自己的孩子（通常也是一个view系统），通过dispatchDraw(canvas)实现，参看ViewGroup.Java中的代码可知，dispatchDraw->drawChild->child.draw(canvas)这样的调用过程被用来保证每个子View的draw函数都被调用，通过这种递归调用从而让整个View树中的所有View的内容都得到绘制。在调用每个子View的draw函数之前，需要绘制的View的绘制位置是在Canvas通过translate函数调用来进行切换的，窗口中的所有View是共用一个Canvas对象；
5. 如果需要，绘制淡入淡出相关的内容并恢复保存的画布所在的层（layer）；
6. 绘制修饰的内容（例如滚动条等）；

当一个ChildView要重画时，它会调用其成员函数invalidate()函数将通知其ParentView这个ChildView要重画，这个过程一直向上遍历到ViewRoot，当ViewRoot收到这个通知后就会调用上面提到的 ViewRoot中的draw函数从而完成绘制。  

ParentView根据ChildView在其内部的布局来调整Canvas，其中画布的属性之一就是定义和ChildView相关的坐标系，Android动画就是通过ParentView来不断调整ChildView的画布坐标系来实现的。  

    dispatchDraw() 
    { 
         .... 
         Animation a = ChildView.getAnimation() 
         Transformation tm = a.getTransformation(); 
         Use tm to set ChildView's Canvas; 
         Invalidate(); 
         .... 
    } 

可知某一个View的动画的绘制并不是由他自己完成的而是由它的父view完成。

#####动画类型

抽象类Animation中主要定义了动画的一些属性，比如开始时间、持续时间、是否重复播放等，这个类主要有两个重要的函数：getTransformation和applyTransformation，在getTransformation中Animation会根据动画的属性来产生一系列的差值点，然后将这些差值点传给applyTransformation，不同的动画重载这个函数，实现不同的动画效果，通过这个函数将根据这些点来生成不同的Transformation，Transformation中包含一个矩阵和alpha值，矩阵是用来做平移、旋转和缩放动画的，而alpha值是用来做透明度的动画，类的层次结构如下：

![](../images/animation.png)

#####如何使用？

使用代码创建如下类的实例构建动画:

- AlphaAnimation:渐变透明度动画效果
- ScaleAnimation:渐变尺寸伸缩动画效果
- TranslateAnimation:画面转换位置移动动画效果
- RotateAnimation:画面转移旋转动画效果
- AnimationSet:动画集合
- Rotate3dAnimation:3D动画  

例如：
 
    Animation anim = AnimationUtils.loadAnimation(mContext, R.anim.rotate); 
	//监听动画的状态（开始，结束）
	anim.setAnimationListener(new EffectAnimationListener());
	textWidget = (TextView)findViewById(R.id.text_widget);
	textWidget.setText("画面旋转动画效果");
	textWidget.startAnimation(anim);

在XML中定义动画，然后通过AnimationUtils加载:

+ alpha:渐变透明度动画效果
+ scale:渐变尺寸伸缩动画效果
+ translate:画面转换位置移动动画效果
+ rotate:画面转移旋转动画效果
+ set: 动画集合

例如：

	<?xml version="1.0" encoding="utf-8"?>  
	<set xmlns:android="http://schemas.android.com/apk/res/android"  
    	 android:fillAfter = "false"  
    	 android:zAdjustment="bottom">  
    	<rotate  
        	android:fromDegrees="0"  
        	android:toDegrees="360"  
        	android:pivotX="50%"  
        	android:pivotY="50%"  
        	android:duration="4000"/>
        <scale
			android:interpolator= “@android:anim/accelerate_decelerate_interpolator”
			android:fromXScale=”0.0″
			android:toXScale=”1.4″
			android:fromYScale=”0.0″
			android:toYScale=”1.4″
			android:pivotX=”50%”
			android:pivotY=”50%”
			android:fillAfter=”false”
			android:startOffset=“700”
			android:duration=”4000″
			android:repeatCount=”10″ />  
	</set>  

####帧动画

Android SDK提供了另外一个类AnimationDrawable来定义使用帧动画，本身并没有提供接口来监听动画的状态,需要自己处理。

例如在drable文件夹中实现帧动画：

	<?xml version="1.0" encoding="utf-8"?>  
	<animation-list  
	  xmlns:android="http://schemas.android.com/apk/res/android"  
	  android:oneshot="true">  
	       <item android:drawable="@drawable/p1" android:duration="1000"></item>  
	       <item android:drawable="@drawable/p2" android:duration="1000"></item>  
	       <item android:drawable="@drawable/p3" android:duration="1000"></item>  
	       <item android:drawable="@drawable/p4" android:duration="1000"></item>  
	       <item android:drawable="@drawable/p5" android:duration="1000"></item>  
	       <item android:drawable="@drawable/p6" android:duration="1000"></item>  
	</animation-list>  

代码中调用帧动画：

	AnimationDrawable anim = (AnimationDrawable)getResources().getDrawable(R.drawable.frame);
	textWidget = (TextView)findViewById(R.id.text_widget);
	textWidget.setText("背景渐变动画效果");
	textWidget.setBackgroundDrawable(anim);
	anim.start();
	
####属性动画

![](../images/animator.png)



####参考

- [官方SDK][7]
- [Android Animation学习笔记][1]
- [传统View动画与Property动画基础及比较][6]- 
- [Android属性动画完全解析][9]
- [Android属性动画深入分析：让你成为动画牛人][2]
- [Property Anim详解][3]
- [Android 属性动画 源码解析 深入了解其内部实现][4]
- [属性动画视频教程][8]

[1]:http://www.cnblogs.com/feisky/archive/2010/01/11/1644482.html
[2]:http://blog.csdn.net/singwhatiwanna/article/details/17841165
[3]:http://blog.csdn.net/xushuaic/article/details/40424379
[4]:http://blog.csdn.net/lmj623565791/article/details/42056859
[5]:http://nineoldandroids.com
[6]:http://blog.csdn.net/xushuaic/article/details/40322345
[7]:http://developer.android.com/guide/topics/resources/animation-resource.html
[8]:http://www.imooc.com/learn/263
[9]:http://blog.csdn.net/lmj623565791/article/details/38067475



