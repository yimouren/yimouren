---
layout:     post
title:      Vue+Element分组件调用dialog
subtitle:   分组件实现弹出和关闭功能
date:       2018-09-16
author:     Ayi
header-img: 
catalog: true
tags:
    - Vue+Element
---

## 前言

要实现点击某个按钮就能弹出一个对话框[Element文档](http://element-cn.eleme.io/#/zh-CN/component/dialog)中是这样写的

```
<el-button type="text" @click="centerDialogVisible = true">点击打开 Dialog</el-button>

<el-dialog
  title="提示"
  :visible.sync="centerDialogVisible"
  width="30%"
  center>
  <span>需要注意的是内容是默认不居中的</span>
  <span slot="footer" class="dialog-footer">
    <el-button @click="centerDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="centerDialogVisible = false">确 定</el-button>
  </span>
</el-dialog>

<script>
  export default {
    data() {
      return {
        centerDialogVisible: false
      };
    }
  };
</script>
```

但是这只是把对话框的代码写在一个页面里的时候可以这样写，在日常的开发中，如果对话框弹出的是一个表单或者更复杂的业务需求的时候，把对话框写在一个页面里就会使代码显得特别冗余，维护起来特别麻烦。

因此通常把这样的对话框单独写成一个子组件，在需要调用它的父组件里引用即可。

## 子组件该怎么定义

>在定义子组件时候，不能单纯的把对话框的代码复制到一个新定义的子组件里，因为在vue中，父组件是负责管理变量的，是没有权限去直接操作子组件里的变量。

因为这个机制呢，我们就要在定义子组件的时候提供给父组件管理的接口，让父组件通过这个接口能管理子组件里的变量。

代码实现

```
ManagePacket.vue //子组件
<template>
//组件内容
</template>
<script>
export default {
  name: 'ManagePacket', //接口变量名
  props: {
    ManagePacket: { 初始化
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      
    }
  },
  methods: {
    cancelManage() {
      this.$emit('dialog-cancel') //函数接口
    }
  }
}

</script>
```

```
//父组件
<manage-packet></manage-packet>//引用子组件
<script>
import ManagePacket from './sidebar/ManagePacket.vue' //引进子组件
export default {
  components: {
    ManagePacket  //声明子组件
  }
</script>
```

## 利用父子组件实现dialog弹框

```
子组件
<template>
  <div class=managepacket>
    <el-dialog title="分组管理" 
    :visible.sync="ManagePacket" 
	:before-close="cancelManage" 
	:modal-append-to-body="false" 
	:close-on-click-modal="false" 
	width="30%" 
	center>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: 'ManagePacket',
  props: {
    ManagePacket: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      
    }
  },
  methods: {
    cancelManage() {
      this.$emit('dialog-cancel')
    }
  }
}

</script>
<style lang="scss">


</style>

```

>代码解释

`:visible.sync="ManagePacket"`是定义决定dialog是否弹框的变量，这里我们将`ManagePacket`作为接口传给父组件，让父组件通过管理这个变量来操作子组件

此外`this.$emit('dialog-cancel')`声明了一个函数的接口给父组件，让父组件可以通过一个函数操作子组件

接下来我们看看父组件是怎么写的

```
父组件
 <el-button type="info" icon="el-icon-setting" @click="maNage">管理分组</el-button>
<manage-packet 
	v-if="manageOnoff" 
	:manage-packet="manageOnoff" 
	@dialog-cancel="closeManage">
</manage-packet>
<script>
import ManagePacket from './sidebar/ManagePacket.vue'
export default {
  components: {
    ManagePacket
  },
  data() {
    return {
      manageOnoff: false
    }
  },
  methods: {
    maNage() {
      this.manageOnoff = true
    },
    closeManage() {
      this.manageOnoff = false
    }
  }
}
</script>
```

这里我们可以看到子组件中的接口`ManagePacket`在父组件中因为驼峰命名法变成了`:manage-packet="manageOnoff"`，而`manageOnoff`就是用来给父组件管理的变量，`ManagePacket`监听着`manageOnoff`的值，并将他传给子组件，因此在父组件中只要操作`manageOnoff`就可以实现dialog的弹出和关闭

>但是如果我们想要在弹出的dialog中给关闭按钮绑定一个函数呢

我们当然不能在dialog中直接操作`ManagePacket`来关闭diaolog，因为vue是不允许在父组件中直接操作子组件的值的，这就是为什么我们在子组件中定义了一个函数的接口`dialog-cancel`，并在父组件中用`dialog-cancel`监听一个函数。

## 总结
 
> * 控制dialog的弹出，先在子组件里定义一个接口变量名，在父组件里将接口变量名A与一个变量B绑定，点击弹出按钮的时候，绑定一个函数C，C函数改变变量B的值，因为变量B与接口A绑定，A将改变的值传回给子组件使用
> * 关闭dialog，在子组件的取消按钮绑定一个函数D，并在method中声明一个函数接口E，在父组件中给E绑定一个新的函数F，在dialog中点击D的时候，会触发父组件的F，F定义改变B的值，再通过A传回子组件，从而关闭dialog

## 特别说明

>代码里有个方法需要让我觉得拿出来说一说的就是`before-close="cancelManage"`这个是专门用于给dialog右上角默认的那个关闭按钮绑定函数的。
>如果不用这个函数去改变`visible.sync`的值，当第一次开启dialog后，就没有方法去改变`:visible.sync`了，就会报错。
>这里的思路就是在点击右上角默认关闭前回调一个函数`cancelManage`，去改变`visible.sync`的值。

当时就是因为不知道这个，而踩了很久的坑。

最后在这里谢谢@瑰总对我关于父子组件调用的耐心指导。