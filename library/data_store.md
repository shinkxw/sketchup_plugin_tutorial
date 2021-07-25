# 本地数据存储

数据存储功能是由SuDb模块中的各个类来提供的，它们分别提供了从简单的单个配置表到对象数组到大量自定义类数据的存储，具体说明请参考[su_db说明][des]。
[des]: https://github.com/shinkxw/shink_sketchup_library/blob/master/su_db%E8%AF%B4%E6%98%8E.txt

存储的实质是将保存对象序列化为json格式然后保存在文件或者对象的属性中。在定义db对象时可以通过mode参数指定数据储存在电脑或是当前打开的模型中。global模式因为在SketchUp2018中read_default接口有bug而建议用file模式来代替，为此专门增加了文件自动备份功能来防止意外导致的数据大量丢失。
