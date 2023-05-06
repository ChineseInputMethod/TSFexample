## 微软早期的TSF样例


### [BasicTextService](https://github.com/ChineseInputMethod/TSFexample/tree/master/1BasicTextService)

演示如何注册TSF输入法以及介绍激活输入法的接口

Interface     | Description
-------- | -----
ITfInputProcessorProfiles  | TSF和IME输入法一样，都是DLL。IME是通过导出函数实现的输入法功能，而TSF输入法的功能是在COM接口中完成的。
ITfTextInputProcessor  | TSF是一个COM组件。不但要实现相应的TSF框架接口，而且一些输入法功能，也是Windows通过相应接口提供的。
Text Service  | 客户程序和文本服务通过TSF管理器实现管理。客户程序与文本服务不直接发生交互。


[TSF文本服务核心内容](https://github.com/ChineseInputMethod/mumble/blob/main/2023/5/1.md)

### 第二部分  微软早期的TSF样例

[注册TSF输入法以及加载输入法的接口](https://github.com/ChineseInputMethod/mumble/blob/main/2023/5/1.md)