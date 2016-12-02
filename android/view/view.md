View流程

|步骤|关键字|作用|
|1|构造函数|初始化(初始化画笔Paint)|
|2|onMeasure|测量View的大小(暂时不用关心)|
|3|onSizeChanged|确定View大小(记录当前View的宽高)|
|4|onLayout|确定子View布局(无子View，不关心)|
|5|onDraw|实际绘制内容(绘制饼状图)|
|6|提供接口|提供接口(提供设置数据的接口)|
