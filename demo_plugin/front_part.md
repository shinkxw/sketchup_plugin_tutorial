# 前端构建

在前端方面，为了更加方便地使用框架与调试，使用[Node.js][]作为安装包与运行的环境，没有安装的需要进官网下载安装包进行安装。在前端文件夹中添加由vue-cli生成的模板文件后，在空白处使用shift+鼠标右键呼出菜单，点击在此处打开命令窗口，在弹出的命令窗口中执行`npm install`来安装需要的包(如安装速度过慢可考虑换源)。
[Node.js]: https://nodejs.org/en/

安装完毕后，在前端文件夹中创建src文件夹，在其中添加前端需要的文件与文件夹，目录结构如下：
```
|-- src
    |-- main.js
    |-- router.js
    |-- components.js
    |-- assets
    |   |-- fonts
    |   |-- javascripts
    |   |   |-- global_function.js
    |   |-- stylesheets
    |       |-- index.css
    |-- components
    |   |-- obj_select.vue
    |-- views
        |-- quick_naming.vue
        |-- plugin_settings
            |-- layer_setting.vue
            |-- layout.vue
            |-- naming_setting.vue
```
其中main.js文件是整个前端执行的入口，它引入了其他的js与css文件，并对页面进行初始化。router.js定义了路由，将views文件夹中的vue文件与网址对应起来。components.js文件则把components文件夹中的vue文件注册为全局组件，可供其他组件来使用。assets文件夹中存放的是资源文件，包括js,样式表以及字体文件，它们都需要在main.js中被引入。

``` js
let route_config = [
  { path: '/quick_naming', component: require('v/quick_naming') },
  { path: '/plugin_settings', component: require('v/plugin_settings/layout'),
    children: [
      { path: 'layer_setting', component: require('v/plugin_settings/layer_setting') },
      { path: 'naming_setting', component: require('v/plugin_settings/naming_setting') }
    ]
  }
]
```
我们来仔细看看router.js文件以及对应的views文件夹的结构，vue-router支持多级路由，所以页面结构是使用界面为一级路由，它的地址是/quick_naming，图层设置与命名设置页面则在plugin_settings路径下，分别为/plugin_settings/layer_setting与/plugin_settings/naming_setting，其中layout.vue文件就是两个页面共通的布局内容，即左边栏。三个功能页面目前并无太多内容，只是用来占位。

global_function.js文件中定义的是一些可被全局调用的函数，其中su_location函数就是用作与Browser类中的add_callback方法进行配合的，我们使用它来封装参数并传递给SU，这通常用来执行会改变SU模型的命令，获取数据则通过本地服务器。

在添加完前端文件后就可以开始查看页面内容了。这里为了方便起见，在前端文件夹中添加两个bat文件，用来快速执行常用的两个命令，双击npm dev.bat用来调试页面，它会在8080端口开启测试服务器，可在浏览器的地址栏中输入`http://localhost:8080/#/quick_naming`来查看功能页面，并会随着文件的修改而实时改变页面内容，双击npm build.bat则用来生成SU中显示的页面，在页面内容更新后需要手动执行后SU中的页面才会变化。

最后，在后端文件夹中添加local_server.rb文件并在打开窗口时调用其中的本地服务器，就可以在SU中点击按钮查看对应的页面了(记得先双击npm build.bat生成)，功能窗口中有一个测试按钮用来触发SU的弹窗功能，设置窗口中则可以通过左边栏进行两个设置界面的跳转。两个窗口可以同时打开，他们使用的是同一个本地服务器。前端的构建就讲到这里，下一节开始正式的功能制作。
