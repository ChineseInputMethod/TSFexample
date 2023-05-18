## 2.0 说明

这个仓库是源自微软早期的TSF样例，包括9个输入法工程和2个附加工程。本仓库将这11个工程，整合到了一个解决方案中，稍微改动了一下，以方便阅读。并对，涉及到TSF输入法的关键知识点，进行了注释。

这个解决方案，使用Visual Studio 2019编辑。源码在工程目录的src文件夹中，样例的原文档在工程的doc文件夹中。

请按工程名的数字顺序阅读，附加的两个工程无先后顺序。

## 2.1 [BasicTextService](https://github.com/ChineseInputMethod/TSFexample/tree/master/1BasicTextService)

如何注册TSF输入法以及激活输入法

Interface						|Description
-|-
[ITfInputProcessorProfiles][1]	|文本服务语言配置操作。（可以视同为注册输入法）
[ITfTextInputProcessor][2]		|文本输入处理器，激活文本服务。（可以看成输入法被激活的第一个接口）

## 2.2 [TrackFocus](https://github.com/ChineseInputMethod/TSFexample/tree/master/2TrackFocus)

如何安装事件接收器以及调试输入法

Interface					|Description
-|-
[ITfThreadMgr][3]			|线程管理器，主要用于安装事件接收器和获取文档管理器。
[ITfSource][4]				|消息接收器，用于安装事件接收器。
[ITfThreadMgrEventSink][5]	|线程管理器事件接收器，主要捕获焦点事件。

## 2.3 [TrackTextChange](https://github.com/ChineseInputMethod/TSFexample/tree/master/3TrackTextChange)

如何处理焦点事件以及查看编辑记录

Interface				|Description
-|-
[ITfDocumentMgr][6]		|文档管理器，用来创建和管理编辑内容对象。
[ITfTextEditSink][7]	|编辑会话完成消息接收器，当编辑会话完成时，TSF管理器调用此接口。
[ITfEditRecord][8]		|编辑记录，用来确定编辑会话期间更改的内容。
[IEnumTfRanges][9]		|片段对象枚举器，枚举片段对象。

## 2.4 [IconInLanguageBar](https://github.com/ChineseInputMethod/TSFexample/tree/master/4IconInLanguageBar)

如何设置输入法语言以及显示语言栏

Interface				|Description
-|-
[ITfLangBarItem][6]		|语言栏项目信息。
[ITfLangBarItemButton][6]		|语言栏按钮项目信息。
[ITfLangBarItemSink][6]		|语言栏项目消息接收器。
[ITfMenu][6]		|语言栏菜单扩展。
[ITfLangBarItemMgr][6]		|语言栏项目管理器。


[1]: https://github.com/ChineseInputMethod/TextServicesFramework/blob/master/Reference/Interfaces/TSFmanager/ITfInputProcessorProfiles.md
[2]: https://github.com/ChineseInputMethod/TextServicesFramework/blob/master/Reference/Interfaces/TextService/ITfTextInputProcessor.md
[3]: https://github.com/ChineseInputMethod/TextServicesFramework/blob/master/Reference/Interfaces/TSFmanager/ITfThreadMgr.md
[4]: https://github.com/ChineseInputMethod/TextServicesFramework/blob/master/Reference/Interfaces/TSFmanager/ITfSource.md
[5]: https://github.com/ChineseInputMethod/TextServicesFramework/blob/master/Reference/Interfaces/TextService/ITfThreadMgrEventSink.md
[6]: https://github.com/ChineseInputMethod/TextServicesFramework/blob/master/Reference/Interfaces/TSFmanager/ITfDocumentMgr.md
[7]: https://github.com/ChineseInputMethod/TextServicesFramework/blob/master/Reference/Interfaces/TextService/ITfTextEditSink.md
[8]: https://github.com/ChineseInputMethod/TextServicesFramework/blob/master/Reference/Interfaces/TSFmanager/ITfEditRecord.md
[9]: https://github.com/ChineseInputMethod/TextServicesFramework/blob/master/Reference/Interfaces/TSFmanager/IEnumTfRanges.md