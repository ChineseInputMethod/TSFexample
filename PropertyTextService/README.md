## 2.B.0 主要文件结构

- TextService.cpp
  - LanguageBar.cpp
    - StaticCompactProperty.cpp
    - StaticProperty.cpp
    - CustomProperty.cpp
-CustomPropertyStore.cpp

在Register.cpp文件中，增加了将自定义属性的GUID添加到注册表输入法预定义属性类别操作。<br/>
在StaticCompactProperty.cpp、StaticProperty.cpp、CustomProperty.cpp文件中，演示了如何将属性附加到文本范围。<br/>
在CustomPropertyStore.cpp文件中，演示了如何实现一个属性对象。<br/>

本节介绍如何将自定义属性附加到选择的文本范围，本节未演示ITfCreatePropertyStore创建属性存储接口。

## 2.B.1 将属性附加到文本范围

本节输入法需要在win7或win8系统中演示。

请将项目-属性-调试-命令，设置为使用写字板进行调试。

>$(ProgramW6432)\Windows NT\Accessories\wordpad.exe

GUID_TFCAT_PROPSTYLE_STATICCOMPACT和GUID_TFCAT_PROPSTYLE_STATIC属性的演示过程相同。
用微软拼音输入任意汉字，然后用鼠标选中汉字，在语言栏选择"Attach Static Compact Property to selection"或"Attach Static Property to selection"的子菜单项。

![select](doc/PropertyTextService.JPG)

然后就可以在上一节的属性监视窗口中观察到属性被附加到了选择的文本范围。

![select](doc/PropertyMonitor.JPG)

## 2.B.2 将自定义属性附加到文本范围

```C++
```