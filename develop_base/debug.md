# 插件存放与调试

以下介绍下我自己存放插件文件与调试的方式，字符串的内容都是根据自己情况填入的，仅供大家参考。

SketchUp存放插件文件的默认路径一般在用户文件夹内，且层级较深。在Ruby控制台中输入`$LOAD_PATH`，输出的路径中带有`/SketchUp/Plugins`就是该路径。我一般会在那里添加一个 load_local.rb 文件，内容是`require 'G:/git/su_load'#自己项目文件夹内的加载文件路径`。然后在自己的项目文件夹中添加 su_load.rb 文件，内容如下：
```ruby
load_path_arr = []
load_path_arr << "G:/git/插件文件夹名称/入口文件名称.rb"
#可读取多个插件
load_path_arr.each do |load_path|
  load load_path if File.exist?(load_path)
end
```
这样就可以在自己的文件中开发插件，并被SketchUp读取到了。

调试代码时，根据要调试的规模，一般分为三种情况：涉及多个插件文件的修改我会直接重启SketchUp，单个文件高频率的修改测试时我会在Ruby控制台中使用`load '发生修改的插件文件路径'`重载文件，几行代码的调试则会直接在Ruby控制台中进行修改。
