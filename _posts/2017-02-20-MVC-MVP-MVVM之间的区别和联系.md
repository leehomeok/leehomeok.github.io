转载自https://www.cnblogs.com/guwei4037/p/5591183.html

MVC、MVP、MVVM 这些模式是为了解决开发过程中的实际问题而提出来的，目前作为主流的几种架构模式而被广泛使用。

<span style="line-height: 1.5;">一、MVC（Model-View-Controller）</span>

MVC 是比较直观的架构模式，用户操作 -&gt;View（负责接收用户的输入操作）-&gt;Controller（业务逻辑处理）-&gt;Model（数据持久化）-&gt;View（将结果反馈给 View）。

MVC 使用非常广泛，比如 JavaEE 中的 SSH 框架（Struts/Spring/Hibernate），Struts（View, STL）-Spring（Controller, Ioc、Spring MVC）-Hibernate（Model, ORM）以及 ASP.NET 中的 ASP.NET MVC 框架，xxx.cshtml-xxxcontroller-xxxmodel。（实际上后端开发过程中是 v-c-m-c-v，v 和 m 并没有关系，下图仅代表经典的 mvc 模型）

![](https://images2015.cnblogs.com/blog/112316/201606/112316-20160616144231401-1133255130.png)

二、MVP（Model-View-Presenter）

MVP 是把 MVC 中的 Controller 换成了 Presenter（呈现），目的就是为了完全切断 View 跟 Model 之间的联系，由 Presenter 充当桥梁，做到 View-Model 之间通信的完全隔离。

.NET 程序员熟知的 ASP.NET webform、winform 基于事件驱动的开发技术就是使用的 MVP 模式。控件组成的页面充当 View，实体数据库操作充当 Model，而 View 和 Model 之间的控件数据绑定操作则属于 Presenter。控件事件的处理可以通过自定义的 IView 接口实现，而 View 和 IView 都将对 Presenter 负责。

![](https://images2015.cnblogs.com/blog/112316/201606/112316-20160616144243917-1180649863.png)

三、MVVM（Model-View-ViewModel）

如果说 MVP 是对 MVC 的进一步改进，那么 MVVM 则是思想的完全变革。它是将 “数据模型数据双向绑定” 的思想作为核心，因此在 View 和 Model 之间没有联系，通过 ViewModel 进行交互，而且 Model 和 ViewModel 之间的交互是双向的，因此视图的数据的变化会同时修改数据源，而数据源数据的变化也会立即反应到 View 上。

这方面典型的应用有. NET 的 WPF，js 框架 Knockout、AngularJS 等。

![](https://images2015.cnblogs.com/blog/112316/201606/112316-20160616144255948-1076854192.png)

参考资料：

http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html
