---
layout: post
title: "iOS中锚点anchorPoint理解"
date: 2014-05-10 14:10
comments: true
categories: iOS
tags: 
---

#### 引言

最近项目中使用到UIBezierPath和CAShapeLayer，发现锚点anchorPoint的概念有点模糊，来捋一捋。

#### 概念

UIView内部管理这一个CALayer，这厮才是真正显示的东西。首先要清楚UIView和CALayer它们几何层级关系的表示，我们通常的UIView的通过frame来描述视图在父视图中的位置，其实frame是根据另外两个属性bounds和center计算出来的。而CAlayer也有这几个属性，另外增加了一个anchorPoint属性，同UIView一样，frame属性也是计算出来的，首先记住一个公式，如果脑子转不过来，运算一下：

	// UIView的frame计算公式
	frame.origin.x = center.x - 0.5 * bounds.size.width;
	frame.origin.y = center.y - 0.5 *bounds.size.height;
	或者
	center.x = frame.origin.x + 0.5 * bounds.size.width;
	center.y = frame.origin.y + 0.5 *bounds.size.height;
	
	// CALayer的frame的计算公式
	frame.origin.x = position.x - anchorPoint.x * bounds.size.width;
	frame.origin.y = position.y - anchorPoint.x * bounds.size.height;
	或者
	position.x = frame.origin.x + anchorPoint.x * bounds.size.width；  
	position.y = frame.origin.y + anchorPoint.y * bounds.size.height；


结论就是：frame.origin由position和anchorPoint共同决定，anchorPoint和position之间互不影响。当你设置图层的frame属性的时候，position点的位置（也就是position坐标）根据锚点（anchorPoint）的值来确定；而当你设置图层的position属性的时候，bounds的位置（也就是frame的orgin坐标）会根据锚点(anchorPoint)来确定。

在实际情况中，可能还有这样一种需求，我需要修改anchorPoint，但又不想要移动layer也就是不想修改frame.origin，那么根据前面的公式，就需要position做相应地修改。简单地推导，可以得到下面的公式：

	positionNew.x = positionOld.x + (anchorPointNew.x - anchorPointOld.x)  * bounds.size.width  
	positionNew.y = positionOld.y + (anchorPointNew.y - anchorPointOld.y)  * bounds.size.height
	
但是在实际使用没必要这么麻烦。修改anchorPoint而不想移动layer，在修改anchorPoint后再重新设置一遍frame就可以达到目的，这时position就会自动进行相应的改变。写成函数就是下面这样的：

	- (void) setAnchorPoint:(CGPoint)anchorpoint forView:(UIView *)view{
  		CGRect oldFrame = view.frame;
  		view.layer.anchorPoint = anchorpoint;
  		view.frame = oldFrame;
	}

#### 锚点anchoroPoint

可以理解成变换(平移，缩放，旋转)的支点，锚点也是一个CGPoint，用于描述支点的位置相对视图bounds的比例值，比如anchorPoint = (0.5, 0.5)，锚点位于中心；在左手坐标系(iOS)中，那么anchorPoint（0，1）位于右上角，如下图:

![][1]

![][3]



参考：  

- [彻底理解position与anchorPoint][2]



[1]: http://wonderffee.github.io/images/anchorPointAndPosition/layer_coords_anchorpoint_position_2x.png
[2]: http://wonderffee.github.io/blog/2013/10/13/understand-anchorpoint-and-position/
[3]: http://wonderffee.github.io/images/anchorPointAndPosition/anchorpoint2.jpg