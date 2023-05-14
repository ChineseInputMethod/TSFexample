## 2.3.0 主要文件结构

- TextService.cpp
  - ThreadMgrEventSink.cpp
  - ITfTextEditSink

ThreadMgrEventSink线程管理器事件接收器的实现在ThreadMgrEventSink.cpp文件中。

ITfTextEditSink编辑会话完成消息接收器的实现在ITfTextEditSink.cpp文件中。

本节介绍ThreadMgrEventSink和ITfTextEditSink接口，如何初始化编辑环境和编辑完成时被TSF管理器回调。

## 2.3.1 