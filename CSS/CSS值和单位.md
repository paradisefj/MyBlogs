# CSS值和单位

## 数字

CSS中有两类数字：整数和实数

## 百分数

百分数是一个计算得出的实数，总是相对于另一个值，这个值是任意的，可能是**同一个元素另一个属性的值**、**从父元素继承的一个值**、**祖先元素的一个值**。

## 颜色

### 命名颜色

CSS规范中定义了17个颜色名。`aqua`,`fuchsia`,`lime`,`olive`,`red`,`white`,`black`,`gray`,`maroon`,`orange`,`silver`,`yellow`,`blue`,`green`,`navy`,`purple`,`teal`

### RGB/RGBA

### 函数RGB

函数rgb(color),其中color用一个百分数或者整数三元组表示。百分数范围0%~100%,整数范围0~255。

rgb(red, green, blue, alpha), Alpha透明度,取值0~1之间

### 十六进制RGB

#加三个介于00~FF的十六进制数连起来来表示颜色。

例如: h1 { color: #FF000; } 等价于 RGB(255,0,0);

## 长度单位

### 绝对长度

有五种绝对长度单位:

1) 英寸(in)

2) 厘米(cm)

3) 毫米(mm)

4) 点(pt) (标准印刷度量单位) 1英寸是72点

5) 派卡(pc) 印刷单位 1派卡相当于12点，6派卡等于1英寸

**只有当浏览器知道用来显示页面的显示器、打印机或者其他任何用户代理的所有细节时，这些单位才有用。**

### 相对长度

1. em

   1个em定义为一种给定字体的font-size值。如果一个元素的font-size为14像素，那么对于该元素，1em就等于14像素。

2. ex

   是指所用字体中小写x的高度。

3. px

   像素

## URL

`url(prococol://server/pathname)`或者`url(pathname)`

例如: url(http://www.baidu.com/a.png)

**在CSS中，相对URL要相对于样式表本身，而不是使用该样式的HTML文档**

**url和开始括号之间不能有空格**

