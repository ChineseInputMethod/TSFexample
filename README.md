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
[ITfSource][4]				|事件安装器，用于安装事件接收器。
[ITfThreadMgrEventSink][5]	|线程管理器事件接收器，主要捕获焦点事件。

## 2.3 [TrackTextChange](https://github.com/ChineseInputMethod/TSFexample/tree/master/3TrackTextChange)

如何处理焦点事件以及查看编辑记录

Interface				|Description
-|-
[ITfDocumentMgr][6]		|文档管理器，主要用来创建和管理上下文。
[ITfTextEditSink][7]	|编辑会话完成消息接收器，当编辑会话完成时，TSF管理器调用此接口。
[ITfEditRecord][8]		|编辑记录，用来确定编辑会话期间更改的内容。
[IEnumTfRanges][9]		|文本范围枚举器，枚举文本范围对象。

## 2.4 [IconInLanguageBar](https://github.com/ChineseInputMethod/TSFexample/tree/master/4IconInLanguageBar)

如何设置输入法语言以及显示语言栏

Interface					|Description
-|-
[ITfLangBarItemMgr][10]		|语言栏项管理器，用于管理语言栏中的项。
[ITfLangBarItem][11]		|语言栏项信息，由语言栏管理器用来获取语言栏项的详细信息。
[ITfLangBarItemButton][12]	|语言栏按钮项信息，由语言栏管理器用来获取语言栏上的按钮项信息。
[ITfMenu][13]				|语言栏菜单扩展，用于为语言栏按钮添加菜单项。
[ITfLangBarItemSink][14]	|语言栏项消息接收器，用于将语言栏项中的更改通知语言栏。

## 2.5 [TextInsertion](https://github.com/ChineseInputMethod/TSFexample/tree/master/5TextInsertion)

请求编辑会话以及使用客户端标识符

Interface					|Description
-|-
[ITfContext][15]			|上下文，用来创建和管理编辑上下文。
[ITfEditSession][16]		|编辑会话，由TSF管理器调用，用来修改上下文的文本和属性。
[ITfInsertAtSelection][17]	|在选定位置插入内容，用于在上下文中插入文本或嵌入对象。
[ITfRange][18]				|文本范围，用来引用和操作给定上下文中的文本。

## 2.6 [Keyboard](https://github.com/ChineseInputMethod/TSFexample/tree/master/6Keyboard)

注册输入法类别以及安装键盘事件接收器

Interface					|Description
-|-
[ITfCategoryMgr][19]			|类别管理器，为输入法注册类别。
[ITfCompartmentMgr][19]			|公共缓冲池管理器。
[ITfCompartment][19]			|类公共缓冲池对象。

[1]: https://github.com/ChineseInputMethod/Interface/blob/master/TSFmanager/ITfInputProcessorProfiles.md
[2]: https://github.com/ChineseInputMethod/Interface/blob/master/TextService/ITfTextInputProcessor.md
[3]: https://github.com/ChineseInputMethod/Interface/blob/master/TSFmanager/ITfThreadMgr.md
[4]: https://github.com/ChineseInputMethod/Interface/blob/master/TSFmanager/ITfSource.md
[5]: https://github.com/ChineseInputMethod/Interface/blob/master/TextService/ITfThreadMgrEventSink.md
[6]: https://github.com/ChineseInputMethod/Interface/blob/master/TSFmanager/ITfDocumentMgr.md
[7]: https://github.com/ChineseInputMethod/Interface/blob/master/TextService/ITfTextEditSink.md
[8]: https://github.com/ChineseInputMethod/Interface/blob/master/TSFmanager/ITfEditRecord.md
[9]: https://github.com/ChineseInputMethod/Interface/blob/master/TSFmanager/IEnumTfRanges.md
[10]: https://github.com/ChineseInputMethod/Interface/blob/master/LanguageBar/ITfLangBarItemMgr.md
[11]: https://github.com/ChineseInputMethod/Interface/blob/master/TextService/ITfLangBarItem.md
[12]: https://github.com/ChineseInputMethod/Interface/blob/master/TextService/ITfLangBarItemButton.md
[13]: https://github.com/ChineseInputMethod/Interface/blob/master/LanguageBar/ITfMenu.md
[14]: https://github.com/ChineseInputMethod/Interface/blob/master/LanguageBar/ITfLangBarItemSink.md
[15]: https://github.com/ChineseInputMethod/Interface/blob/master/TSFmanager/ITfContext.md
[16]: https://github.com/ChineseInputMethod/Interface/blob/master/TextService/ITfEditSession.md
[17]: https://github.com/ChineseInputMethod/Interface/blob/master/TSFmanager/ITfInsertAtSelection.md
[18]: https://github.com/ChineseInputMethod/Interface/blob/master/TSFmanager/ITfRange.md
[19]: https://github.com/ChineseInputMethod/Interface/blob/master/TSFmanager/ITfCategoryMgr.md