---
layout:     post
title:      用settimeout写一个定时器
subtitle:   用settimeout实现定时上传或下载
date:       2018-10-15
author:     Ayi
header-img: 
catalog: true
tags:
    - js用法
---

```
// 获取统计数据
    getstateStatistics () {
      var _this = this
      setTimeout(function f () {
        Api.getstateStatistics()
          .then(data => {
            let datas = data.data
            // console.log(datas)
            if (data.status === 0) {
              _this.BtnArr[0].num = datas.total
              _this.BtnArr[1].num = datas.online
              _this.BtnArr[2].num = datas.offline
              _this.BtnArr[3].num = datas.alarm
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
        setTimeout(f, 600000)
      }, 200)
    }
```

利用settimeout可以实现定时完成你想要的功能，

用法是`settimeout( function f () {}, 1000 //执行时间，单位为毫秒 )`

循环的实现方法
```
  setTimeout(function f() {
    //你的函数体
    setTimeout(f, 1000)
  },1000)
```