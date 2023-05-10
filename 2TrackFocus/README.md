## 2.2.0 主要文件结构

- TextService.cpp
- ThreadMgrEventSink.cpp

ITfTextInputProcessor接口的实现在TextService.cpp文件中。

ThreadMgrEventSink接口的实现在ThreadMgrEventSink.cpp文件中。

## 2.2.1 TSF输入法中接口的含义

TSF输入法的接口类似与IME输入法的导出函数。输入法需要实现的内容，都是通过接口提供给TSF管理器的。
但是TSF输入法因为是COM组件的原因，系统并不知道TSF输入法所实现的接口，甚至并不知道这个COM组件是个输入法。

输入法调用了上一节介绍的ITfInputProcessorProfiles接口，系统才知道这个COM组件是个文本服务。
所以在每一个需要文本服务的进程中，调用了输入法的ITfTextInputProcessor接口。

也就是说，输入法必须通过某种方式告诉系统，自己实现了哪些接口，系统才会调用输入法提供的相应接口。
一共有两种方式实现这种“链接”。一种是像ITfInputProcessorProfiles那样，将组件类别写入到注册表中，在后面的小节会讲到如何注册组件类别。

另一种就是本节介绍的：安装事件接收器。

## 2.2.2 输入法与TSF管理器之间的关系。

[这是微软拼音开发团队的博客](https://blog.csdn.net/MSPinyin?type=blog)

![MSPinyin](http://hi.csdn.net/attachment/201101/14/0_12949724148bK8.gif)

