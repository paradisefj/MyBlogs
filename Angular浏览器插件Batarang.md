
#Angular浏览器插件Batarang介绍

对于Angular新手来说，刚接手Angular的时候都会比较痛苦。确实，相对于JQuery、Backbone等，Angular门槛确实相对较高，而且比较难以调试。今天给大家带来一个Angular Chrome 插件Batarang的介绍，运用好改插件，会帮助加深对Angular的理解。

##准备工作

安装Batarang：

- 方法一：在Chrome应用商店中查找Batarang，并安装。
- 方法二：在网上查找Batarang的安装包，直接在Chrome浏览器中安装。


##使用

在已安装的Batarang插件的浏览器中打开一个Angular应用，并打开控制台，如下图：
![图1](http://img.blog.csdn.net/20160420160323933)

会发现控制台中多了一个AngularJS的页面，勾选“Enable”，该控件就可以使用了：

###**Models**

点开Models，如下图，左侧是该应用下的所有Scope的信息，右侧是Scope对应的模型信息。点击某个作用域，右侧相应的会显示出该作用域中的所有模型信息。
点击Scope前的"<"，会跳到Elements中该作用域所在的DOM标签上。

![图2](http://img.blog.csdn.net/20160420160400200)

###**Performance**

Performace展示的是该应用的性能，左侧显示的是监控树，右侧显示的是监控表达式的性能，这个页面能帮助我们进行性能优化。

![图3](http://img.blog.csdn.net/20160420160424935)

###**Dependenices**

Dependenices展示的指令和服务之间的依赖关系，选定某个指令，可以看到其依赖的服务。

![图4](http://img.blog.csdn.net/20160420160455966)

###**Options**

最后是options页面。有三个选项："show applications," "show scopes," 和 "show bindings."。每个选项勾选时，在debugger时，相应的内容会在页面高亮。

![图5](http://img.blog.csdn.net/20160420160626499)
![图6](http://img.blog.csdn.net/20160420160653610)

###**help**

如有任何问题，请查看help

###**Element**

其实我最常用的应该是Element右侧的AngularJS Properties标签。在Element标签中选定某个标签时，Element页面的右侧的内容，会多一个AngularJS Properties页面，该页面显示的是选定的html内容的作用域的属性，该功能对于对Angular Scope的理解非常有用。如果不是很理解Angular Scope，可以多用一个这个功能。

![图7](http://img.blog.csdn.net/20160420160721282)

