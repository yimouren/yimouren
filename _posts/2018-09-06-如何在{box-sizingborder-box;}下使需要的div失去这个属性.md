---
layout:     post
title:      如何在*{box-sizing:border-box;}下使需要的div失去这个属性
subtitle:   在全局变量下让某个div不被污染
date:       2018-09-06
author:     Ayi
header-img: 
catalog: true
tags:
    - box-sizing：border-box
    - 防止css污染
---

今天根据需求的需要将原本在CSS中写在全局属性的`*{box-sizing:border-box;}`之下的两个div两行显示，而不是并排显示

简单描述下`*{box-sizing:border-box;}`的作用就是可以使两个div并排显示

这里再拓展一下

我们熟悉的默认盒模型是content-box，也是标准的w3c标准的盒子，在形成的时候是先做content，然后在外面从里到外添加padding，border，margin。

而border-box是怪异盒模型，多用于手机移动端，但是在web也经常使用，他是先加载div中的margin，border，padding，然后再做content，采用这种模型的好处就是border和padding会设置在宽度的内部。

###举个例子

>如果要设置两个div并列显示，宽度各设置为50%，那么content下的盒子实际宽度就是50%+你设置的px，那么右边的div就会被挤到下一行错位显示，但是如果使用了border-box属性，那么div的宽度就是50%，你已经设置的px已经包含在box里，这样就可以使两个div并列显示。

PS：一个小细节就是无论是border-box还是conten-box都不会改变盒子的块级元素的属性，如果要并列显示还是要将块级元素改成行内元素或者其他

###言归正传

我们现在要在全局属性为border-box下使某个div失去这个属性，那么有两种方法，

- 使需要的div的属性设置回默认的content-box，即`<div style="box-sizing:content-box;"></div>`，但是我今天的代码中这样更改后仍然无法起作用，可能是因为我直接在公司的项目上改，他应该还以用了别的样式我还没有注意到的缘故，这个以后再更新。但是如果只是纯CSS全局属性下这样更改是可以起作用的

- 对于复杂的情况，就像我今天遇到的公司项目，对于一个你不熟悉的项目，里面样式太多一时间无法顾及，可以在更改回默认属性的同时更改div的宽度，反利用border-box属性，使并列的两个div宽度超出限制，比如一个设置50%，一个70%，那么这两个div必定会错行显示

PS:这个方法比较暴力，也比较应急，在没有深入理解border-box的情况下只用于粗糙的满足需求而已。