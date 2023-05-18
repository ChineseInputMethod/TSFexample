## 2.4.0 主要文件结构

- Server.cpp
- TextService.cpp
- LanguageBar.cpp

CClassFactory类厂组件的实现在Server.cpp.cpp文件中。

CTextService输入法组件的实现在TextService.cpp文件中。

CLangBarItemButton语言栏按钮组件的实现在LanguageBar.cpp文件中。

本节介绍如何将CLangBarItemButton语言栏按钮组件，装载到语言栏中。

## 2.4.1 添加系统语言

开发者的开发语言环境一般为简体中文，当输入法的运行语言也为简体中文时，就会给调试带来相当的困扰。
可以为开发者的系统添加新的语言，例如添加美国英语。但是，在美国英语系统里输入简体中文的场景不多。
所以，一般还是修改输入法的运行语言比较好，从当前工程开始，输入法的运行语言设置为繁体中文。

![Language](img/Language.png)

## 2.4.2 显示语言栏

从当前工程开始，输入法需要语言栏，才能看到演示效果，所以需要开启语言栏。
完成以上两步后，在LanguageBar.cpp文件中设置跟踪，可以观察到语言栏的装载过程。

![LanguageBar](img/LanguageBar.png)

## 2.4.3 将CLangBarItemButton语言栏按钮组件，装载到语言栏中。

## 2.4.4 将语言栏项目消息通报给接收器。