# 开发环境

#### 本地安装Ruby
如果不熟悉其他的脚本语言，建议在操作系统上也安装Ruby，便于熟悉语法以及执行一些脚本操作，比如处理并打包插件文件。在window上安装Ruby的方式一般是通过[RubyInstaller][]，安装后需在命令窗口执行`gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/`来切换gem源。
[RubyInstaller]: http://rubyinstaller.org/

#### 编辑器
代码编辑器方面喜欢哪个就用哪个，插件版的编辑器在测试小段代码时是比Ruby控制台要方便一点的。我主要使用的是Sublime，它可以轻松的在一个文件夹内的多个文件中进行跳转，用来编辑rb、html、js与vue等多种类型的文件也比较合适。其中编辑vue文件时需要安装Vue Syntax Highlight插件来进行高亮显示。

#### 版本控制
如果需要开发规模稍大的插件，建议对代码进行版本控制，git是一个很好的选择。
