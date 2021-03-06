# 像开发网站一样开发插件

在开发SketchUp插件前，我从事的是使用Ruby开发网站的工作。开发插件时不免将两者的异同进行了一番比较，SketchUp使用了系统自带的浏览器作为插件的界面容器，同样使用的是html的技术，除了增加了一些限制以外并没有什么本质的不同。彼时开发的第一个插件又是一个页面诸多，功能繁复的商城类插件，因此在不断的尝试中，逐渐发现此路可行，最后沉淀下来的，就是这个[基础库][library]了。
[library]: https://github.com/shinkxw/shink_sketchup_library

网站开发作为程序员界最普遍的工作，其中的工程思想自然是经过时间锤炼的。在前后端分离的开发模式下，后端由数据库与api服务器组成，前端则包含了路由以及页面展示。要在SketchUp中复现的话，这几个角色还是不可少。由于作为插件本身的限制和对性能的要求不高(网站只需服务插件使用者本人)，数据存贮这块由我自己写了些模块来实现，以便捷好用为主要目的，api服务器则选择了Ruby语言自带的服务器WEBrick。前端方面，考虑到浏览器兼容性，上手难度，灵活程度等因素，使用了[Vue][]这个js框架来做页面展示，同时Vue的核心组件[Vue Router][]也能够很好的满足对路由功能的需求。
[Vue]: https://cn.vuejs.org/
[Vue Router]: https://router.vuejs.org/zh/

那么这里来讲讲使用api服务器与前端框架的好处，毕竟增加了这么多的学习成本。首先，页面上获取数据的操作可以变得简化与直观，原本SketchUp所提供的方式只能单方向的向后端发出命令，再执行js才能向前端输送需要的数据，这是一个很别扭的操作，意味着你发出了请求，却不知道数据什么时候才会从另一个方法中被传过来。而使用http请求来作为数据的传输通道，在编写相同的功能时就可以很简洁的通过请求-回调的方式来处理数据。其次，页面上的元素内容在调整样式时会变得很方便，解决错误也会容易得多，因为api服务器的存在，我们可以在各种优秀的浏览器上访问插件页面，使用开发者工具来调整样式，查看数据包，排查页面错误，而不是只能在插件界面用着IE，干瞪着眼无从下手。最后，网站的路由系统简化了页面间的跳转与信息传递，使得开发逻辑相当灵活，你可以开发一个很复杂的功能，通过页面间的跳转来完成一个个步骤，也能用来开发工具箱，每一个按钮打开的界面对应的是网站中不同的页面。

使用这种方式工作难免会有一些局限，由于api服务器是运行在线程中的，SketchUp所提供的部分接口在非主线程中执行时会没有效果或者导致软件崩溃。所以我统一了在插件中输出到控制台、执行js、设置对象属性这三种操作的接口，通过基础库中提供的方法来跨线程调用它们。另外插件前后端通过端口号来进行相互通信，故开发的插件不应该同时在多个SketchUp窗口中被使用。
 
最后我得承认这算是一种屠龙术，只适合对插件页面有复杂需求的开发者使用，只需要简单界面的，看看参考下就行了。
