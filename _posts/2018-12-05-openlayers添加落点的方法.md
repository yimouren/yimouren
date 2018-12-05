---
layout:     post
title:      openlayers添加落点的方法
subtitle:   
date:       2018-12-05
author:     Ayi
header-img: 
catalog: true
tags:
    - openlayers
---

```
    let centerF = new ol.Feature(new ol.geom.Point(center)) // center是一个数组坐标
    let vectorSource = new ol.source.Vector({
      features: [centerF] // 这里必须是一个数组 
    })
    this.vectorcenter = new ol.layer.Vector({
      source: vectorSource,
      style: function (feature) {
        let color = 'blue'
        return new ol.style.Style({
          image: new ol.style.Circle({
            radius: 10,
            fill: new ol.style.Fill({
              color: color
            })
          })
        })
      }
    })
    this.map.addLayer(this.vectorcenter)
```