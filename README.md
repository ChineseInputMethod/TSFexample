## 2.0 说明

这个仓库是源自微软早期的TSF样例，包括9个输入法工程和2个附加工程。本仓库将这11个工程，整合到了一个解决方案中，稍微改动了一下，以方便阅读。并对，涉及到TSF输入法的关键知识点，进行了注释。

这个解决方案，使用Visual Studio 2019编辑。源码在工程目录的src文件夹中，样例的原文档在工程的doc文件夹中。

请按工程名的数字顺序阅读，附加的两个工程无先后顺序。

## 2.1 [BasicTextService](https://github.com/ChineseInputMethod/TSFexample/tree/master/1BasicTextService)

如何注册TSF输入法以及激活输入法

Interface					|Description
-|-
ITfInputProcessorProfiles	|注册TextInputProcessor。（可以视同为注册输入法）
ITfTextInputProcessor		|激活文本服务。（可以看成输入法被激活的第一个接口）

## 2.2 [TrackFocus](https://github.com/ChineseInputMethod/TSFexample/tree/master/2TrackFocus)