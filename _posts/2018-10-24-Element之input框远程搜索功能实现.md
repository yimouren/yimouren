---
layout:     post
title:      Element之input框远程搜索功能实现
subtitle:   下拉框不显示问题解决办法
date:       2018-10-24
author:     Ayi
header-img: 
catalog: true
tags:
    - Element
---

## 实现代码

```
        <el-autocomplete placeholder="分组名/对象名" v-model="filterText" :fetch-suggestions="querySearchAsync" popper-class="my-autocomplete" @select="handleSearchSelect" size="small">
          <i slot="suffix" class="el-input__icon el-icon-search"></i>
        </el-autocomplete>
```

## 函数实现

```
    querySearchAsync (queryString, cb) {
      if (queryString === '') {
        let air = []
        cb(air)
        // this.getfindGroupTree(this.BtnArr[0], 0)
      } else {
        clearTimeout(this.timeout)
        this.timeout = setTimeout(() => {
          Api.getfindByKeyWord(queryString)
            .then(data => {
              let datas = data.data
              if (data.status === 0) {
                if (datas !== []) {
                  var restaurants = datas
                  var arr = []
                  restaurants.forEach((item, index) => {
                    arr.push({
                      id: item.value,
                      value: item.label,
                      parentid: item.parentid
                    })
                  })
                  cb(arr)
                } else {
                  let air = []
                  cb(air)
                }
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
        }, 500)
      }
    }
```

> 需要注意的是回显的时候，必须要用`value`接收名称，否则下拉框会出现无法显示的问题，而其他的参数可以任意塞入