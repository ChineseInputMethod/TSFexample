## 2.A.0 主要文件结构

- TextService.cpp
  - ThreadFocusSink.cpp
- DumpProperties.cpp
  - MemoryStream.cpp
  - PopupWindow.cpp

在ThreadFocusSink.cpp文件中，实现了线程输入焦点消息接收器。<br/>
在DumpProperties.cpp文件中，演示了如何监视显示属性。<br/>
在MemoryStream.cpp文件中，管理了一块内存流对象。<br/>
在PopupWindow.cpp文件中，监视了上下文中的显示属性。

本节介绍如何监视上下文中的显示属性，以及自定义属性。

## 2.A.1 演示的主要流程

当上下文的文档锁被释放时，ITfTextEditSink编辑会话完成消息接收器被调用。

```C++
STDAPI CPropertyMonitorTextService::OnEndEdit(ITfContext *pContext, TfEditCookie ecReadOnly, ITfEditRecord *pEditRecord)
{
    DumpProperties(pContext);
    return S_OK;
}
```
