# 插件雏形

现在开始第一步，先建立shink_quick_naming文件夹作为根文件夹(也是版本控制的根文件夹，后文会用根文件夹指代)，然后在其中建立shink_quick_naming文件夹来存放Ruby代码(后文会用后端文件夹指代)，建立shink_quick_naming_front文件夹来存放前端代码(后文会用前端文件夹指代)，新建shink_quick_naming.rb作为入口文件，它的内容如下：
```ruby
require "sketchup.rb"
require "extensions.rb"

module ShinkQuickNaming
  PluginName = 'shink_quick_naming'
  PluginChineseName = '快速命名分层'
  Version = '0.0.1'.freeze

  load_path = File.join(File.dirname(__FILE__), "#{PluginName}/load.rb")
  extension = SketchupExtension.new(PluginChineseName, load_path)
  extension.version = Version
  extension.creator = 'Shink'
  Sketchup.register_extension(extension, true)
end
```
前面两行算是例行公事。第4行起开始正式内容，首先定义了一个命名空间，即一个全局模块，名称一般就是插件名称的驼峰式写法，因为Ruby的常量需要以大写字母开头，它会作为后面写的所有代码的容器，在名称上要尽量不和别的插件重复。后面三行通过定义常量的方式保存了插件的名称，中文名与版本号。第9行通过拼接方法获得了插件加载文件的路径，其中\_\_FILE\_\_的值是当前文件的路径。第10行使用插件的中文名以及插件加载文件路径通过SU提供的api创建了一个extension对象，给它设置了版本与创建人的信息后，我们使用SU的api来注册他并希望被立刻加载。这就是入口文件的意义，SU每次开启时都会加载所有插件的入口文件，但是插件的启用状态决定了它的加载文件是否会被执行。

然后，开始初始化后端代码的整体结构，首先在后端文件夹中将[基础库][library]的代码整个文件夹复制过来，然后新建插件加载文件load.rb，内容如下：
```ruby
$LOAD_PATH.unshift(File.dirname(__FILE__))

load('shink_sketchup_library/load.rb')#加载基础库
SHINK_LIBRARY.inject_to_module(ShinkQuickNaming)#注入到模块

Sketchup::require('path')#路径相关方法
Sketchup::require('db')#数据定义
Sketchup::require('main')#按钮菜单定义
Sketchup::require('browser')#对话框定义
```
这里讲讲Ruby是如何找到代码文件来加载的，当你require一个文件时，Ruby会依次在$LOAD_PATH(这是Ruby提供的一个全局数组对象)中的路径里去寻找有这个名称的rb文件并执行。所以在加载文件的第一行通过调用$LOAD_PATH的unshift方法将当前的后端文件夹路径放在数组第一的位置，这样按照顺序来进行检索的话，即使其他插件中有和我们相同名称的rb文件也不会加载错误。第3第4行加载并把基础库注入了插件模块，第6行起就是把插件中的rb文件都通过Sketchup::require方法来加载，这是SU提供的，它扩展了Ruby自带的require方法，能够加载加密后的rb文件。
[library]: https://github.com/shinkxw/shink_sketchup_library

接下来讲讲插件中几个通用的rb文件，他们各自负责了一部分基础功能。
```ruby
module ShinkQuickNaming
  PATH = File.dirname(__FILE__)
  module_function

  def data_path
    "#{DocumentsPath}/sketchup_plugin/#{PluginName}_#{SuVersion}"
  end

  def log_path(name = '')
    "#{data_path}/log/#{name}"
  end

  def tmp_path(name = '')
    "#{data_path}/tmp/#{name}"
  end

  def small_icon_path(icon_name)
    "#{PATH}/icon/#{icon_name}_32.png"
  end

  def large_icon_path(icon_name)
    "#{PATH}/icon/#{icon_name}_64.png"
  end

  def preset_data_path(file_name)
    "#{PATH}/preset_data/#{file_name}"
  end

  def local_page_path(type)
    "#{local_server_path}/index.html?type=#{type}&time=#{Time.now.to_i}"
  end

  def local_server_path
    "http://localhost:#{local_server_port}"
  end

  def local_server_port
    '38586'
  end

  def document_root_path
    if IsDevelop
      "#{PATH}/../shink_quick_naming_front/dist"
    else
      "#{PATH}/dist"
    end
  end

  FileUtils.mkdir_p(log_path)
  FileUtils.remove_dir(tmp_path) if Dir.exist?(tmp_path)
  FileUtils.mkdir_p(tmp_path)
end
```
path.rb文件中定义了插件中和路径相关的方法，包括日志路径，图标路径，临时路径等都是在这里定义的。需要注意的是，这里还定义了服务器的地址，其中local_server_port即服务器挂载的端口，尽量选取不会冲突的端口号(端口上限是65535，最好选择五位数的端口号)。document_root_path方法则根据插件的开发状态返回服务器的文件根目录路径。
```ruby
module ShinkQuickNaming
  SuDb.set_section("#{PluginName}_DB")
  Config = SuDb::Config.new_by_key('config')
end
```
db.rb文件中定义了SuDb使用的section(用来划分不同插件存储的数据位置)，以及全局会使用到的数据结构，这里定义了一个全局配置对象，用来存贮插件相关的配置，基础库中的有些功能也会使用到这个常量。
```ruby
module ShinkQuickNaming
  module Main
    module_function

    def show_quick_naming_dialog
      if check_browser
        @quick_naming_dialog ||= Browser.new("快速命名分层", 'quick_naming')
        @quick_naming_dialog.show
      end
    end

    def show_plugin_settings_dialog
      if check_browser
        @plugin_settings_dialog ||= Browser.new("插件设置", 'plugin_settings')
        @plugin_settings_dialog.show
      end
    end

    def check_browser
      if ShinkQuickNaming.const_defined?(:IEVersion) && IEVersion < 9
        UI.messagebox('您的浏览器版本过低, 请至少升级至IE9')
        false
      else
        true
      end
    end
  end

  unless file_loaded?(__FILE__)#保证代码块中代码只执行一次
    toolbar = UI.toolbar(PluginName)#创建工具栏

    cmd1 = UI::Command.new("Show Quick Naming Dialog") {
      Main.show_quick_naming_dialog
    }
    cmd1.small_icon = small_icon_path('quick_naming')#设置小图标
    cmd1.large_icon = large_icon_path('quick_naming')#设置大图标
    cmd1.tooltip = '快速命名分层'#设置提示文字
    toolbar = toolbar.add_item(cmd1)#添加按钮

    cmd99 = UI::Command.new("Show Plugin Settings Dialog") {
      Main.show_plugin_settings_dialog
    }
    cmd99.small_icon = small_icon_path('plugin_settings')
    cmd99.large_icon = large_icon_path('plugin_settings')
    cmd99.tooltip = '插件设置'
    toolbar = toolbar.add_item(cmd99)

    toolbar.show#显示工具栏

    file_loaded(__FILE__)
  end
end
```
main.rb文件中定义了显示两个窗口的方法，并创建了一个工具栏，设置了两个按钮与窗口调用的关联。其中`@quick_naming_dialog ||= Browser.new("快速命名分层", 'quick_naming')`这行用到了||=运算符，它的意思是当@quick_naming_dialog为nil时，将一个新建的窗口对象赋给@quick_naming_dialog，当@quick_naming_dialog不为nil时，则不进行操作。因此当这行代码被多次调用时，@quick_naming_dialog只会被赋值一次，这样每次调用这个方法就会打开同一个窗口对象。因为Vue框架需要浏览器版本至少为IE9，check_browser方法会在打开窗口前调用，检查当前的IE浏览器版本。
```ruby
module ShinkQuickNaming
  class Browser < ShinkBrowser
    def initialize(title, type)
      @type = type
      super(title, true, type, 0, 0, 500, 150, true)
      set_background_color(get_default_dialog_color)
      set_callback
    end

    def set_callback
      add_callback("alert"){|d, param| UI.messagebox(param.message)}
    end

    def show
      if visible?
        bring_to_front
      else
        set_url(index_path)
        set_window_min_size_by_type
        SuRunJs.set_web_dialog(@type, self)#将窗口类型与窗口实例绑定
        case @type
        when 'quick_naming'
          RefreshEntityObserver.open(@type) do#设置选中对象改变时调用的代码
            run_js(@type, "quick_naming.refresh_entity_info()")
          end
          after_close{ RefreshEntityObserver.close(@type) }#设置窗口关闭时调用的代码
        end
        super
      end
    end

    def set_window_min_size_by_type
      case @type
      when 'quick_naming'
        set_window_min_size(500, 460)
      when 'plugin_settings'
        set_window_min_size(1200, 670)
      end
    end

    def index_path
      ShinkQuickNaming.local_page_path(@type)
    end
  end

  class RefreshEntityObserver < Sketchup::SelectionObserver
    def self.open(type, &block)
      @type_observer_hash ||= {}
      observer = RefreshEntityObserver.new(&block)
      @type_observer_hash[type] = observer
      Sketchup.active_model.selection.add_observer(observer)
      ModelChangeObserver.open(type){ RefreshEntityObserver.open(type, &block) }
    end

    def self.close(type)
      if @type_observer_hash[type]
        Sketchup.active_model.selection.remove_observer(@type_observer_hash[type])
        @type_observer_hash[type] = nil
        ModelChangeObserver.close(type)
      end
    end

    def initialize(&block);@block = block end
    def onSelectionBulkChange(selection);@block.call end
    def onSelectionCleared(selection);@block.call end
  end

  class ModelChangeObserver < Sketchup::AppObserver
    def self.open(type, &block)
      @type_observer_hash ||= {}
      if @type_observer_hash[type].nil?
        observer = ModelChangeObserver.new(&block)
        @type_observer_hash[type] = observer
        Sketchup.add_observer(observer)
      end
    end

    def self.close(type)
      if @type_observer_hash[type]
        Sketchup.remove_observer(@type_observer_hash[type])
        @type_observer_hash[type] = nil
      end
    end

    def initialize(&block);@block = block end
    def onNewModel(model);@block.call end
    def onOpenModel(model);@block.call end
  end
end
```
browser.rb文件包含了两个部分的内容。第一部分以基础库提供的ShinkBrowser类作为基类定义了窗口类，添加了初始化方法，需要窗口标题与窗口类型两个参数。窗口类型用于区分窗口的作用，它影响到窗口的最小长宽、打开时使用的路径以及作为某些方法的索引参数。其中set_callback方法中设置的都是对前端的回调处理，这里使用的是基类提供的add_callback方法，它与前端的方法一起作用，压缩并优化了传入参数，使得后端可以以对象的形式来操作数据，我们在后面的章节中还会继续根据要实现的功能增加回调。第二部分则定义了两个观察者类，分别用来在选中对象改变与当前打开模型变化时执行预设的回调，用于根据用户的操作来刷新前端页面上的数据。可看到RefreshEntityObserver中有对ModelChangeObserver的调用，这是因为当打开的模型改变时，之前设置的SelectionObserver会无效，所以需要监测这个变化并再次开启SelectionObserver。

在添加了这些代码文件以及两种图标后，打开SU我们就能看到有两个图标的工具栏出现了，现在由于尚未构建前端，点击按钮后出现的窗口还是没有内容的，在下个章节中将重点讲述前端如何构建。
