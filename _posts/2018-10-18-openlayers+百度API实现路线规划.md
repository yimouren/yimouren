---
layout:     post
title:      openlayers+百度Api实现路线规划
subtitle:   路线规划实现
date:       2018-10-18
author:     Ayi
header-img: 
catalog: true
tags:
    - openlayers
---

> 首先感谢@LZU-GIS的博客指导，他写了一篇openlayers+高德API实现路线规划的博客，但是我由于需求不一样，需要用百度API，所以写了这篇博客。[博客地址](https://blog.csdn.net/GISShiXiSheng/article/details/78936830)

接下来是百度API的实现方法

```
handleRoute (data, index) {
      if (index === undefined) {
        index = 0 // 默认选择第一条路线
      }
      if (this.vectorroute !== null) { // 消除上一次生成的路线
        this.map.removeLayer(this.vectorroute)
      }
      if (this.vectorHandselectstart !== null) { // 消除点击生成的起点坐标，否则会有偏移
        this.map.removeLayer(this.vectorHandselectstart)
      }
      if (this.vectorHandselectend !== null) { // 消除点击生成的终点坐标
        this.map.removeLayer(this.vectorHandselectend)
      }
      if (this.vectorresaw !== null) {
        this.map.removeLayer(this.vectorresaw)
      }
      let result = data.routes[index]
      let origin = coordtransform.bd09towgs84(result.origin.lng, result.origin.lat)
      let startC = origin // ol.proj.transform([origin[0], origin[1]], 'EPSG:4326', 'EPSG:3857')
      let destination = coordtransform.bd09towgs84(result.destination.lng, result.destination.lat)
      let endC = destination // ol.proj.transform([destination[0], destination[1]], 'EPSG:4326', 'EPSG:3857')
      let startF = new ol.Feature(new ol.geom.Point(startC))
      startF.name = '起点'
      let endF = new ol.Feature(new ol.geom.Point(endC))
      endF.name = '终点'
      let features = [startF, endF]
      let steps = result.steps
      for (let i = 0; i < steps.length; i++) {
        let _step = steps[i]
        let arr = _step.path.split(';')
        let tmcsPaths = arr
        let _coord = []
        // console.log(tmcsPaths.length) //每一段道路下具体坐标的数组
        for (let m = 0; m < tmcsPaths.length; m++) {         
          let path = tmcsPaths[m] // 具体点的数组[经度，纬度]
          let _path = path.split(',').map(Number)
          let pathwgs = coordtransform.bd09towgs84(_path[0], _path[1])
          _coord.push(pathwgs /* ol.proj.transform([pathwgs[0], pathwgs[1]], 'EPSG:4326', 'EPSG:3857') */)       
          // console.log(pathF.status) //道路状况标识位 0:无路况,1:畅通，2：缓行，3：拥堵，4：非常拥堵        
        }
        let pathF = new ol.Feature(new ol.geom.LineString(_coord))
        pathF.status = _step.traffic_condition[0].status
        features.push(pathF)
      }
      let vectorSource = new ol.source.Vector({
        features: features
      })
      this.vectorroute = new ol.layer.Vector({
        source: vectorSource,
        style: function (feature) {
          let name = feature.name
          let status = feature.status // feature.get('status')
          // console.log(status)
          name = name ? name.substring(0, 1) : '' // 只取name的前一个字
          let color = ''
          if (name === '起') {
            color = 'green'
          } else if (name === '终') {
            color = 'red'
          } else {

          }
          let _color = '#8f8f8f'
          if (status === 3) {
            _color = '#e20000'
          } else if (status === 2) {
            _color = '#ff7324'
          } else if (status === 1) {
            _color = '#00b514'
          } else {
            _color = '#67C23A'
          }
          return new ol.style.Style({
            image: new ol.style.Circle({
              radius: 15,
              fill: new ol.style.Fill({
                color: color
              })
            }),
            fill: new ol.style.Fill({
              color: '#0044CC'
            }),
            stroke: new ol.style.Stroke({
              color: _color,
              width: 5
            }),
            text: new ol.style.Text({
              text: name,
              font: 'bold 15px 微软雅黑',
              fill: new ol.style.Fill({
                color: 'white'
              }),
              textAlign: 'center',
              textBaseline: 'middle'
            })
          })
        }
      })
      this.map.addLayer(this.vectorroute)
      let extent = this.vectorroute.getSource().getExtent() // 合适比例缩放居中
      this.view.fit(extent, this.map.getSize())
      this.flag = ''
      // this.view.setZoom(15)
      // this.view.setCenter(startC)
    },
```

因为路线规划有多个方案可选，切换方案的思路是由于每次输入起点和终点，方案的形式百度会以数组的形式返回给我们，因此，我给button一个flag选择返回的路线方案，然后将路线传给openlayers渲染

```
    <div class="methodsbox">
      <hr v-show="methodShow" class="hrline">
      <el-button v-for="(item, index) in BtnArr" :key="'BtnArr' + index" size="mini" @click="selectroute(item, index)" :class="methodsIndex === index ? 'container focus' : 'container' ">
        <span class="container">
            <i class="iconfont" v-html="item.icon">{{item.icon}}</i>
            <span>{{item.name}}</span>
        </span>
      </el-button>
    </div>
```

```
    selectroute (item, index) {
      this.getdrivingroad(item.flag)
      this.methodsIndex = index
    },
    getdrivingroad (tag) {
      this.SearchLoading = true
      for (let i = 0; i < this.moreAddress.length; i++) {
        if (this.moreAddress[i].addressValue === '') {
          this.deleteAddress(i)
        } else {}
      }
      let waypoint = this.waypoint.join('|')
      let origin = this.selectItemstart.location.lat + ',' + this.selectItemstart.location.lng
      let destination = this.selectItemend.location.lat + ',' + this.selectItemend.location.lng
      Api.getdrivingroad(origin, destination, waypoint)
        .then(data => {
          if (data.status === 0) {
            if (tag === undefined) {
              tag = 0 // 默认选择第一条路线
            }
            this.route = data.result.routes[tag]
            if (data.result.total !== 0) {
              let arr = []
              for (let i = 0; i < data.result.total; i++) {
                arr.push(this.BtnArrtext[i])
              }
              this.BtnArr = arr
            }
            this.SearchLoading = false
            this.methodShow = true
            this.$emit('handle-route', data.result, tag, this.selectItemstart, this.selectItemend, this.start, this.end)
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
    },
```