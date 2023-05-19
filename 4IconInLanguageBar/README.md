## 2.4.0 主要文件结构

- Server.cpp
- TextService.cpp
- LanguageBar.cpp

CClassFactory类厂组件的实现在Server.cpp.cpp文件中。

CTextService输入法组件的实现在TextService.cpp文件中。

CLangBarItemButton语言栏按钮组件的实现在LanguageBar.cpp文件中。

本节介绍如何将CLangBarItemButton语言栏按钮组件，装载到语言栏中。

## 2.4.1 添加系统语言

开发者的开发语言环境一般为简体中文，当输入法的运行环境也为简体中文时，就会给调试带来相当的困扰。
从当前工程开始，输入法的运行环境设置为繁体中文。如下图所示，为系统添加繁体中文。

![Language](img/Language.png)

## 2.4.2 显示语言栏

从当前工程开始，输入法需要语言栏，才能看到演示效果，所以还需要开启语言栏。

完成以上步骤后，在LanguageBar.cpp文件中设置跟踪，如下图所示，可以观察到语言栏的装载过程。

![LanguageBar](img/LanguageBar.png)

## 2.4.3 添加CLangBarItemButton语言栏按钮组件

当输入法被激活后，首先获取ITfLangBarItemMgr语言栏项目管理器。

```C++
ITfLangBarItemMgr *pLangBarItemMgr;
BOOL fRet;

if (_pThreadMgr->QueryInterface(IID_ITfLangBarItemMgr, (void **)&pLangBarItemMgr) != S_OK)
	return FALSE;
```

然后创建CLangBarItemButton语言栏按钮组件，并在构造函数中初始化TF_LANGBARITEMINFO结构。

```C++
if ((_pLangBarItem = new CLangBarItemButton()) == NULL)
	goto Exit;
```

最后将CLangBarItemButton语言栏按钮组件，添加到语言栏中。

```C++
if (pLangBarItemMgr->AddItem(_pLangBarItem) != S_OK)
{
	_pLangBarItem->Release();
	_pLangBarItem = NULL;
	goto Exit;
}
```

## 2.4.4 CLangBarItemButton语言栏按钮组件加载过程

