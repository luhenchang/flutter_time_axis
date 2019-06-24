### 一，真实点,直接上图吧。
[github地址](https://github.com/luhenchang/flutter_time_axis)  
[博客地址](https://juejin.im/post/5d105d9af265da1b6f4380e6#heading-2)   
[视频教学地址](https://member.bilibili.com/v2#/upload-manager/article)  
![](https://user-gold-cdn.xitu.io/2019/6/24/16b87f5b67e6644e?w=2880&h=1636&f=png&s=2132827)

> `1.位置，上图可见,时间轴可以在左边，也可以在中间，当然了可以在任何位置。`  
  `2.时间轴样式，当然了我们时间轴比仅仅限制为一个圆圈是吧,当然了你的部件能写多炫酷，砸门的时间轴也可以,上图（圆里面爱，图片，黄色背景文字，其实都是一长串部件）。`   
  `3.线，我们需要和内容的高度一样，这里估计是很多人的痛点，没法自适应，这里也做到了。线的粗细，颜色，虚线间隔，渐变...当然砸门也实现了`

### 二 ,看一眼吧 如何实现。
    群里很多人都需要一个时间轴，对于时间轴自适应高度难倒了很多人。当然了，我试着搞了搞，搞了两种思路， 
    第一种有点low但是也能实现。我们知道Container是一个部件，它有一个decoration属性里面又一个  
    boder,而且boder可以设置left,top,right,bootom等让其显示。

代码如下：
    
```
 return Scaffold(
      body: ListView.builder(
        itemCount:10,
        itemBuilder:(context,index){
          return   Column(
            crossAxisAlignment:CrossAxisAlignment.start,
            children: <Widget>[
              Container(
                margin:EdgeInsets.only(left:10),
                height: 200,
                decoration: BoxDecoration(
                    border: Border(
                        left: BorderSide(
                          width: 3,
                          color: Colors.deepOrange,
                        ))),
                child:Text("内容"),
              ),
            ],
          );
        },

      ),
    );
```

![](https://user-gold-cdn.xitu.io/2019/6/24/16b882bbf92c5f52?w=632&h=1252&f=png&s=500230)  

当然很low吧。我们可以看到绘制完成时候可以通过border来绘制边来画出线。其实到这里我想简单的时间轴不用我写了吧？
Colum(
  圆圈，
  容器(border),
  圆圈
)  
![](https://user-gold-cdn.xitu.io/2019/6/24/16b88374f5194ce9?w=632&h=1252&f=png&s=377942)

我们看看border源码：

```

  switch (left.style) {
    case BorderStyle.solid:
      paint.color = left.color;
      path.reset();
      path.moveTo(rect.left, rect.bottom);
      path.lineTo(rect.left, rect.top);
      if (left.width == 0.0) {
        paint.style = PaintingStyle.stroke;
      } else {
        paint.style = PaintingStyle.fill;
        path.lineTo(rect.left + left.width, rect.top + top.width);
        path.lineTo(rect.left + left.width, rect.bottom - bottom.width);
        path.lineTo(rect.left, rect.bottom);

      }
      canvas.drawPath(path, paint);
      break;
    case BorderStyle.none:
      break;
  }
```
到这里我们可以发现，可以通过绘制边来进行时间轴的高度自适应，在Flutter里面又一个CustomPaint。因为画布可以知道部件自己的size那么我们就可以通过在canvas上画时间轴了。这样可以达到自适应。

下面关键代码：
通过CustomPaint来绘制时间线。设置绘制的各种样式。更加灵活相比与border
```
class MyPainterLeft extends CustomPainter {
  //虚线
  double DottedLineLenght;
  //虚线之间的间距
  double DottedSpacing;
  double circleSize;
  Gradient mygradient;
  bool isDash;
  Color LineColor;
  double paintWidth;
  MyPainterLeft(
      {this.circleSize,
        this.mygradient,
        this.isDash = false,
        this.DottedLineLenght = 5.0,
        this.DottedSpacing = 10.0,
        this.LineColor = Colors.redAccent,this.paintWidth=4});

  Paint _paint = Paint()
    ..strokeCap = StrokeCap.square //画笔笔触类型
    ..isAntiAlias = true;//是否启动抗锯齿//画笔的宽度
  Path _path = new Path();

  @override
  Future paint(Canvas canvas, Size size) {
    final Rect arcRect = Rect.fromLTWH(10, 5, 4, size.height);
    _paint.style = PaintingStyle.stroke; // 画线模式
    _paint.color = this.LineColor;
    _paint.strokeWidth=this.paintWidth;
    _path.moveTo(size.width, 0); // 移动起点到（20,40）
    _path.lineTo(size.width, size.height); // 画条斜线
    if (mygradient != null) {
      _paint.shader = mygradient.createShader(arcRect);
    }
    if (mygradient != null && isDash) {
      canvas.drawPath(
        dashPath(_path,
            dashArray: CircularIntervalList<double>(
                <double>[DottedLineLenght, DottedSpacing]),
            dashOffset: DashOffset.absolute(1)),
        _paint,
      );
    } else {
      canvas.drawPath(_path, _paint);
    }
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    return true;
  }
}
```

三，使用： 
| 属性                 | 属性使用介绍       |  是否必须           |
|----------------------|--------------------|---------------------|
| int index            | 列表的index，用来搞定时间轴部件|   true     |
| double timeAxisSize  | 时间轴部件大小    |             true            |
| double contentLeft | 内容距离时间轴距离      | false       |
| Widget leftWidget | 请展示你的技术时间轴部件      | false       |
| double lineToLeft | 时间轴距屏幕左边距离     | false       |
| Gradient mygradient | 时间轴线是否渐变      | false       |
| bool isDash | 时间轴线是否是虚线 true or false     | false       |
| double DottedLineLenght| 虚线部分线段长度      | false       |
| double DottedSpacing | 虚线间隔      | false       |
| double marginLeft | 时间轴线开始画的起点。      | false       |
| Alignment alignment | 时间轴显示的位置  left|center      | false       |
| Widget centerRightWidget | 如果时间轴在中间，左边内容      | false       |
| Widget cententWight | 如果时间轴在中左边，中间的内容      | false       |
| Widget centerLeftWidget | 如果时间轴在中间，右边内容     | false       |
| double timeAxisLineWidth | 时间轴线的宽度      | false       |





 1.pubspec.yaml里面
   flutter_time_axis:  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;git: https://github.com/luhenchang/flutter_time_axis.git
    
2.实例1:

![](https://user-gold-cdn.xitu.io/2019/6/24/16b8854b03df617e?w=828&h=666&f=png&s=172056)  
![](https://user-gold-cdn.xitu.io/2019/6/24/16b884fb7573664f?w=632&h=1252&f=png&s=116245)

其他的:
![](https://user-gold-cdn.xitu.io/2019/6/24/16b88557b59c82c2?w=632&h=1252&f=png&s=114044)  
![](https://user-gold-cdn.xitu.io/2019/6/24/16b8855a11241c18?w=632&h=1252&f=png&s=109617)

![](https://user-gold-cdn.xitu.io/2019/6/24/16b88560a0f63f39?w=632&h=1252&f=png&s=110027)  
![](https://user-gold-cdn.xitu.io/2019/6/24/16b88564fa4dab81?w=2880&h=1636&f=png&s=2132827)




  
