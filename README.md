# Senseauto Map Annotation Tool

## 目录
* [使用安装](#使用安装)
* [User手册](#User手册)
    * [界面介绍](#界面介绍)
    * [标注流程和注意事项](#标注流程和注意事项)
        * [0. 基本操作](#0-基本操作)
        * [1. 创建PolygonObstcle](#1-创建linepolygonobstacle)
        * [2. 调整Line](#2-调整line)
        * [3. Zip Line](#3-zip-line)
        * [4. Zip Lane](#4-zip-lane)
        * [5. Zip Section](#5-zip-section)
        * [6. 标注ID](#6-标注id)
        * [7. Link操作](#7-link操作)
        * [8. Bind操作](#8-bind操作)
    * [按键操作列表](#按键操作列表)

## 使用安装

1. 从git上下载项目，解压
2. 编译后端

    ```
    cd senseauto_map_annotation_tool/
    mkdir build
    cd build
    cmake ..
    make -j
    ```
3. 确认前端编译后的压缩包，存在 `build.zip`
    ```
    ls senseauto_map_annotation_tool/frontend
    ```
    启动后端

    ```
    cd senseauto_map_annotation_tool/build
    ./backend
    ```
4. 查看网页端
    http://localhost:8085/
    推荐使用谷歌浏览器

   

## User 使用手册

### 界面介绍

![whole interface](image/interface_window.png)

**窗口**

1. 道路元素窗口
    - 用一个Tree View来显示地图上的道路元素，层级关系可以<u>展开</u>和<u>收起</u>
    - 元素过多时，出现滚动条上下<u>滚动</u>
2. 地图窗口
    - 主要的地图绘制及标注窗口，在使用键盘操作前，需要先focus这个窗口（点击即可），避免误操作
3. 操作按钮窗口
   - 通过按钮来显示当前可以进行的操作，操作的触发方式包括<u>点击按钮</u>和<u>键盘快捷键</u>
     - 蓝色按钮：任意时刻都可以进行的操作，包括`Save`、`Clear`、`Download`等操作，其中`View`按钮不可点击
     - 灰色按钮：根据在`窗口2`中选中的元素，来判断可以对地图元素进行的操作，包括`Delete`、 `Zip`、`Bind`等操作
4. 属性标注窗口
   - 选中元素后，该窗口将显示该元素的相关属性，包括`ID`、`Type`等等，可以直接<u>编辑文本框</u>或者<u>选择下拉框</u>来改变地图元素的属性
   - 可以选择多个元素，该窗口会依次加载元素属性窗
   - `global_ID`灰色文本框不可修改
5. 日志窗口
   - 每次操作会输出时间和操作，辅助Debug
6. 创建元素窗口
   - 点击按钮，开始创建道路元素，包括：`Line`、`Junction`、`Obstacle`等，最左侧的按钮功能由于需求不明确暂时不会触发创建点元素的事件



### 标注流程和注意事项

*说明：`concept`代表元素概念，**黑体字**代表可视化后的绘图样式，在第一次介绍后均使用`concept`来代表该类元素。*

#### 0. 基本操作
   - 拖动地图：鼠标右键拖动
   - 选中：当鼠标在元素上变换样式时，左键单击选中，按`shift`键多选

#### 1. 创建`Line`/`Polygon`/`Obstacle`

   <div align="center"><img src="image/1-createLine.png" /></div>

   <u>点击创建按钮</u>后，在地图上<u>单击鼠标左键</u>，创建`Point`，即**蓝色圆圈/蓝色三角形**，`Point`元素会依次自动连接成为一条`Line`，<u>双击</u>结束绘制。

   `Polygon`和`Obstacle`的操作同上，`Polygon`

   *注：蓝色三角形代表一条线的起点*

#### 2. 调整`Line`

   <div align="center"><img src="image/2-adjustLine.png" /></div>
   - 调整位置：选中`Line`元素后，<u>拖动</u>`Point`移动到合适的位置
   - 添加点元素：选中`Line`元素，可以发现每两个`Point`中间有一个`Virtual Point`，即**灰色实心圆**，<u>点击</u>即可将其转化为真实的`Point`，再调整位置
   - 删除点元素：选中`Line`元素，选中若干点，按`D`键删除点元素（该操作未在`button window`中显示）

#### 3. Zip Line

<div align="center"><img src="image/3-zipLine.png" /></div>

选中2条`Line`，按`Z`键将车道线组合成车道`Lane`，即**蓝色多边形填充区域**，鼠标光标在该类填充区域会变换样式。
    
**注意**
    
   - 两条`Line`的方向必须一致（可以利用**蓝色三角形**来辨别首位），否则无法合并，该预防机制可以便利后续几个层级的`zip`操作的实现逻辑
   - 两条已经被`zip`的`Line`是无法继续`zip`的，可能会出现下面这种尴尬的情况：中间选中的两条红色车道线无法被`zip`，因而我们建议按照一定的顺序来合成连续的车道

    <div align="center"><img src="image/4-zipOrder.png" /></div>

陷入上图情况的，可以考虑按`⌫`键删除某条`Lane`，再操作。

**操作小技巧**
    
- 当你发现你选不中已经被`zip`过的`Line`的时候，可以按数字键`1~5`来尝试切换可视化级别，再来选中相关元素

#### 4. Zip Lane

   <div align="center"><img src="image/5-zipLane.png" /></div>

   按`shift`键连续选中多条`Lane`，按`Z`键将车道合并成一个`Section`，即**紫色多边形填充区域**。

   单条`Lane`也可以`zip`成为一个`Section`。

#### 5. Zip Section

   选中前后多条`Section`，按`Z`合成一条`Road`，即**棕色多边形填充区域**。

   单条`Section`也可以`zip`成为一个`Road`。
   
   <div align="center"><img src="image/6-zipSection.png" /></div>

   **注意**

   - 所有的`Section`要保持方向一致，并且按顺序依次连接

#### 6. 标注ID

   - 本标注系统中，所有的元素都有自己的`Global_ID`，并会以`Tag`的形式显示在元素周围，即**灰色文字标签**。
     - 对线来说，显示在头尾两端的延长线上
     - 对填充区域来说，显示在多边形的质心上

   - 其中，`Line`是唯一一个既有`Global_ID`又有`Local_ID`的元素，`Local_ID`初始值为`null`，可以选中之后，在右侧属性框中赋予值
   - 按`G`键，可以切换显示`Global_ID`/`Local_ID`

   <div align="center"><img src="image/7-giveID.png" /></div>

#### 7. Link操作

   该操作的主要目的是将一些道路辅助元素和`Road`、`Junction`等道路主体元素相关起来，以此表示他们之间的关系。

   举例：link停车线

   - 将人行横道前的停车线绘制出来，`Line`的`lineType`设置为`stop_line`，会变成**绿色线条**
   - 依次选中`Road`和`stop_line`，按`L`键完成`link`操作，后者被写入前者的objects中

   <div align="center"><img src="image/8-link.png" /></div>

#### 8. Bind操作

   Bind操作主要是标记同等级元素之间的前驱后继关系，例如一条车道`Lane`在一条`road`上遇到车道数变化之后，它接下来的最佳行驶车道就需要人工标记。

   以`Lane`为例：

   - 按`shift`键连续选中2条`Lane`，按`B`键链接，显示**白色虚线箭头**，前者为前驱元素，后者为后继元素
   - 一条`Lane`可以有多条后继
   - 选中前驱元素才会显示虚线链接

   <div align="center"><img src="image/9-bind.png" /></div>

   *上图从左到右依次为`Lane`、`Section`、`Road`的链接*

   

### 按键操作列表

**无对象操作**

- `S` --> `save`

    描述：将当前所有画布元素保存到`local storage`中

- `G` -->  `global`

    描述：切换`global/local_ID` 模式

- `[]`  

    描述：左右回滚操作栈，`volume=8`

- `1-5`

    描述：切换显示层级，Line在任意模式下都显示

    `1-All; 2-Road; 3-Section; 4-Lane; 5-Line`

- `T` -->  `Tag`

    描述：显示/隐藏`Tag`

- `Spacebar` --> `Display`

    描述：按下时隐藏所有`Feature`，松开按键时重新显示。



**单目操作**

- `D` --> `delete`

    描述：删除点

    条件：选中了若干数量的点

- `⌫` --> `Delete`

    描述：删除features（Line及以上的元素）

    条件：选中若干features

- `R`  --> `reverse`

    描述：将一条Line包含的点顺序头尾反向

    条件：选中一条Line



**对象间关系操作**

- `L` --> `link`

    描述：link两个元素，使其构成类似从属的关系，`feature1.objects.push(feature2)`

    条件：

  - `road.push(line)` && `line.lineType == stop_line`
  - `polygon.push(obstacle)` && `obstacle.objectType == obstacle`
  - `road.push(obstacle)` 
    ​	&&  `obstacle.objectType == crosswalk`
    ​	&& `obstacle.objectType == sidewalk`
  - `polygon.push(road)`

- `C` --> `connect`

    描述：连接两条Line的头尾，组合成一条Line

    条件：选中2条Line `feature1 = line` &&  `feature2 == line`

- `B` --> `bind`

    描述：连接元素的前驱后继，选中２个feature，第一个是第二个的前驱

     条件：

  - `pred = line` && `succ = line`
  - `pred = lane` && `succ = lane`
  - `pred = section` && `succ = section`
  - `pred = road` && `succ = road`
  - `pred = polygon` && `succ = road`
  - `pred = road` && `succ = polygon`

- `Z` --> `zip`

    描述：组合道路元素
  
    条件：
  
  - Line
    选中两条Line，其中至少有一条`hasZiped == false`
  - Lane
    选中多条Lane，每一条`hasZiped == false`
  - Section
    选中多条Section，每一条`hasZiped == false`