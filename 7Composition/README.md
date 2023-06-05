## 2.7.0 主要文件结构

- TextService.cpp
  - TextEditSink.cpp
  - KeyEventSink.cpp
    - KeyHandler.cpp
- StartComposition.cpp
- EndComposition.cpp
- Composition.cpp

在TextEditSink.cpp文件中，添加了ITfTextEditSink编辑会话完成消息接收器的终止输入组合内容。<br/>
在KeyHandler.cpp文件中，实现了对按键的处理过程。<br/>
本节在StartComposition.cpp、EndComposition.cpp和Composition.cpp文件中，介绍ITfComposition输入组合的生命周期。

## 2.7.1 完整输入的处理过程

![flowchart](doc/handlekeyflowchart.JPG)

## 2.7.2 输入组合的生命周期

## 2.7.3 输入组合的意外终止