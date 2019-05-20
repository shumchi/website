---
layout: post
title:  "Three methods in plotting a map by ggplot2"
excerpt: ""
date:   2019-04-11 11:56:45
author: Chi Shen
tags: [R, ggplot2, Map]
comments: true
---




## 1.地图素材类型

- 传统的shp素材
- json素材
- 地图包内置地图素材

**shp和json格式素材均可转换为以下两种格式**

## 2.地图数据类型

+ sp:SpatialPolygonDataFrame
+ sf:Simple feature list column

这两种格式的数据集所描述的信息差不多是一致的。第一种格式（**sp**）是R语言绘图比较传统的数据格式，它将地理信息数据分割为两大块：描述层和映射层。

在数据存放时，描述层记录各个地理区域的名称、ID、编号、简写、iOS编码，以及其他标识信息和度量变量，描述层是一个dataframe，我们可以用**data@data**来提取描述层的数据框。

而对应的几何映射层，是每一个行政区域的多边形边界点，这些边界点按照order排序，按照group分组。多边形边界点信息是一个多层嵌套的list结构，但是我们仍然可以通过**fortity**函数将其转化为数据框。

即sp空间数据对象是一个dataframe（描述层）和polygons（几何映射层）两个对象的组合对象。

---------------------------
而sf对象将这种控件数据格式件进行了更加整齐的布局，使用st_read（）导入的空间数据对象完全是一个整齐的数据框，拥有整齐的行列，这些行列中包含着数据描述和几何多边形的边界点信息。其中最大的特点是，它将每一个行政区划所对应的几何边界点封装成了一个list对象的记录，这条记录就像其他普通的文本记录、数值记录一样，被排列在对应行政区划描述的单元格中

## 3.地图数据的读入方式

+ sp::readShapePoly()  *此方法目前虽可用，但不推荐*
+ rgdal::readOGR()
+ sf::st_read() 

## 4.地图可视化的三种方式

### geom_polygon()函数

	ggplot(data = map_data, aes(x=long, y=lat, group=group, fill = fill_var)) +
	  geom_polygon()
	或：
	ggplot(data = map_data) +
	  geom_polygon(aes(x=long, y=lat, group=group, fill = fill_var))
	因为data参数会继承==

此方法必须通过@data及fortity()函数将描述层与几何层分别读取后，添加需要映射的变量，然后合并后进行画图。


### geom_map()函数

	ggplot(fill_file) + 
		geom_map(aes(map_id = merge_id, fill = fill_var), map = map_file)
	或：
	ggplot(fill_file, aes(fill = fill_var)) + 
		geom_map(aes(map_id = merge_id), map = map_file)	

fill_file为包含需要作图变量的数据框，map_file 为地图素材，即通过第4节中直接读取的文件，使用geom_map()函数不需要进行合并，因为合并的过程由**map_id**所指定的参数自动进行合并，merge_id为fill_file中的识别变量。

**Note:  map_file中必须包含三个变量x或long、y或lat、region或id**
**map_id指定的merge_id必须是能与region或id变量进行合并**

例子：

	ids <- factor(c("1.1", "2.1", "1.2", "2.2", "1.3", "2.3"))
	
	values <- data.frame(
	  ids = ids,
	  value = c(3, 3.1, 3.1, 3.2, 3.15, 3.5)
	)
	
	positions <- data.frame(
	  id = rep(ids, each = 4),
	  x = c(2, 1, 1.1, 2.2, 1, 0, 0.3, 1.1, 2.2, 1.1, 1.2, 2.5, 1.1, 0.3,
	  0.5, 1.2, 2.5, 1.2, 1.3, 2.7, 1.2, 0.5, 0.6, 1.3),
	  y = c(-0.5, 0, 1, 0.5, 0, 0.5, 1.5, 1, 0.5, 1, 2.1, 1.7, 1, 1.5,
	  2.2, 2.1, 1.7, 2.1, 3.2, 2.8, 2.1, 2.2, 3.3, 3.2)
	)
	
	library(ggplot2)
	
	ggplot(values, aes(fill = value)) + 
		geom_map(aes(map_id = ids, fill = value), map = positions, color = "red") +
		expand_limits(positions)

**以上两种方法使用coord_map()函数指定坐标投影系**

### geom_sf()函数

	ggplot(map_data) +
	geom_sf(aes(fill = fill_var))

此方法仍然需要一次合并过程

**coord_sf()**
coord_sf()中有参数crs用于指定投影方式，
需使用proj4string格式的字符串。如墨卡托投影用crs = "+proj=merc"。

某些地图投影其字符串中包含经纬度等参数，如crs = "+proj=laea +lat_0=35 +lon_0=-100"。
其它地图投影见Projection methods（https://proj4.org/operations/projections/index.html）。
crs本身包含坐标数据的，就不能再增加xlim, ylim等参数。
