# api服务搭建

搭建api服务需要先创建一个ApiServer对象，然后为它定义需要的接口，最后注册为公用服务，这样多个窗口就可共用同一个服务器。示例代码如下，其中值得一谈的是开启性能提升这一行，SketchUp作为一个建模软件，主界面无时不在进行渲染，所以留给Ruby非主线程执行代码的时间是非常少的，通过使用轮询器在主线程中以比较高的频率来sleep一个很短的时间就能把计算资源让给其他线程，从而提高了api服务器的反应速度以及文件的下载速度。
```ruby
server = ApiServer.new(local_server_port, PATH, log_path)
server.add_api '/plugin_version' do |query|#定义接口
  Version
end

LocalServer = ReuseService.new('local_server') do#注册公用服务
  server_thread = Thread.new { server.start }#开启本地服务器
  sleep(0.1)#等待服务器开启
  Circulator.new(0.1) {sleep(0.01)}.set_stop_condition{ !server_thread.alive? }#开启性能提升
  server_thread
end.at_close do |server_thread|#没有用户使用该服务时
  server_thread.kill#关闭服务器线程
end

LocalServer.add_user('XXXDialog')#打开窗口时调用
LocalServer.delete_user('XXXDialog')#关闭窗口时调用
```
前端如何调用api请参考演示插件的前端内容
