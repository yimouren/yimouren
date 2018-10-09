---
layout:     post
title:      vue+Element循环生成button
subtitle:   button名称为图片
date:       2018-10-09
author:     Ayi
header-img: 
catalog: true
tags:
    - Vue+Element
---

用vue循环生成的button的好处就是可以统一添加样式以及方便与后台进行交互

试想，如果你要写四个button，每个button都调用同一个方法，只是传值传的不同而已，如果不用循环写button的话，那么每个button都要绑定一个方法，虽然四个按钮都是调用一个方法，但是代码会显得特别冗余。

话不多说，直接上代码

```
<template>
        <el-button v-for="(item, index) in BtnArr" :key="'BtnArr' + index" @click="getfindGroupTree(item, index)" :class="packetIndex === index ? 'el-button focus' : 'el-button' " size="mini" :title="item.title">
          <span class="container">
              <i class="iconfont" v-html="item.name">{{ item.name }}</i>
              <span class="number">({{item.num}})</span>
          </span>
        </el-button>
</template>
```

```
  data () {
    return {
      BtnArr: [{
        flag: 'ALL',
        name: '&#xe636;',
        num: '',
        title: '全部'
      }, {
        flag: '01',
        name: '&#xe79e;',
        num: '',
        title: '在线'
      }, {
        flag: '02',
        name: '&#xe786;',
        num: '',
        title: '离线'
      }, {
        flag: '03',
        name: '&#xe604;',
        num: '',
        title: '告警'
      }],
      packetIndex: '0'
   }
```

```
css

    .focus {
      color: #3af2ff;
    }
```

```
Api

  getfindGroupTree (flag) {
    return this._doGetPromise(url + '/targetGroup/findGroupTree', {
      flag: flag
    })
  }

 getfindGroupTree (item, index) {
      this.treeshow = true
      //  console.log(item)
      this.packetIndex = index
      if (item === undefined) {
        this.flag = 'ALL'
      } else {
        this.flag = item.flag
      }
      Api.getfindGroupTree(this.flag)
        .then(data => {
          let datas = data.data
          //  console.log(this.flag)
          if (data.status === 0) {
            this.treeData = datas
          } else {
            this.$message({
              type: 'warning',
              message: '获取数据失败'
            })
          }
        })
        .catch(res => {
          this.$message({
            type: 'error',
            message: '获取数据失败'
          })
        })
    }
```

## 代码解释

* `v-for="(item, index) in BtnArr"`写的原因是，当点击的时候获取到当前item，将item对象传入函数，可以对item对象里的属性进行操作，index是for循环自带的，从0开始，这里是为了后面为了给对应的button添加样式所需

* `<i class="iconfont" v-html="item.name">{{ item.name }}</i>` 因为我使用的是阿里图标库里的图片，但是从代码中看到，我是用图片代替了原本按钮上的文字，而图片的是用代码表示的，但是如果不用`v-html`输出`item.name`的话，循环输出的会变成纯文本，原因是`{%{% %}%}`会将元素当成纯文本输出，v-text将元素当成纯文本输出，v-html将元素当成html标签解析后输出，这样iconfont才能正确渲染

* 在方法`getfindGroupTree`中有一句`this.packetIndex = index`意思是当点击按钮的时候，将当前被点击按钮的index赋值给packetIndex，这样通过属性`"packetIndex === index ? 'el-button focus' : 'el-button' " size="mini"`能给被点击的按钮加上样式表示被选中，就算鼠标单击其他地方，被选中的颜色也不会消失

## 实现效果

![](https://i.imgur.com/i0YJqO2.gif)