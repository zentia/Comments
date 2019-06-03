---
title: 抗锯齿技术
mathjax: true
date: 2019-05-29 14:23:37
tags:
    - 计算机图形学
    - 后处理
categories: 计算机图形学
---

SMAAD技术的全称为"Enhanced Subpixel Morphological Antialiasing"，与FXAA一样同为后期处理抗锯齿技术，可以提供基于过滤算法的通用图像抗锯齿解决方案。SMAA技术使用了更好的几何形体和对角形体检测处理机制，通过图形边缘局部特征对比来识别图像的锯齿特征，并通过重建子像素的方式实现抗锯齿功能，效果与传统的4x MSAA相近而且可以根据游戏的需要进行定制。

{% asset_img 1.gif %}

通过”以模糊换取精确“的手段来消除显示屏上的锯齿，而这个过程我们称其为采样(Sampling)，也就是针对某一点的像素，通过让它带有周围像素的特性，因此在足够的分辨率下，这一点便不再顽固般地锐利，也达到消除锯齿的目的。

# 超级采样抗锯齿(Super Sampling Anti-Aliasing,SSAA)
通过名字就知道，SSAA最大的特点来自采样过程，以常见的SSAA*4为例，在面对一张最后需要以1920*1080像素渲染的画面时，SSAA会首先渲染一张尺寸位3840*2160像素的缓存，再在这种长宽都乘以2的画面上进行采样，采样的精度和效果当然是最理想的，但是你也可以想象，这种只为追求理想情况的手段对于硬件资源的消耗非常大，成本也非常高。更重要的是，即使在原理上SSAA拥有最理想的精度，但是最现实的情况万般变化中的游戏世界，SSAA并不能永远保证采样效果是最讨好眼睛的，换句话说性价比会随着新技术的现身而面临越来越大的挑战，这其中就包括SSAA本身的一种灵活的变体：多重采样抗锯齿技术(Multi-Sampling Anti-Aliasing,MSAA)。

MSAA的原理和SSAA一致，都是通过将图形拉伸至更高倍率之下的缓存之后再精细的图像上进行采样，但是前者真正聪明的地方在于，再开始狮子大开口之前MSAA存在一个判断的过程，换句话说MSAA仅仅针对画面中边缘部分进行放大处理，这么一来对于硬件的负担着实大大减轻，正式因为如此，MSAA早已是最流行的抗锯齿方案之一，对于任何一款现代游戏来说，不支持MSAA几乎是不可想象的。

{% asset_img 1.png %}
{% asset_img 2.png %}

无论是SSAA还是MSAA，他们运作都集中在非常考前的位置，比如光栅化阶段，因此对于硬件开销有较大的呼声，但是从2012年的GeFore 600系列以来，NVIDIA就已经开始推广一款全新的抗锯齿方案，它们最大的特点就是出发点非常靠前，作为一款后处理抗锯齿它的硬件需求非常低，