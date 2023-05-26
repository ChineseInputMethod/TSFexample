## 2.6.0 主要文件结构

- Server.cpp
  - Register.cpp
- TextService.cpp
  - KeyEventSink.cpp
- LanguageBar.cpp
  - Compartment.cpp

在Register.cpp文件中，添加了ITfCategoryMgr类别管理器的使用方法。
在KeyEventSink.cpp文件中，演示了ITfKeystrokeMgr按键管理器的调用方法。
在Compartment.cpp文件中，展示了ITfCompartment公共缓冲池的应用场景。

本节介绍如何注册输入法类别，安装按键事件接收器。

## 2.6.1 注册输入法类别

要被TSF识别为键盘文本服务，输入法的CLSID必须具有GUID_TFCAT_TIP_KEYBOARD类别值。

首先创建类别管理器组件，同时获得类别管理器接口。（这是COM组件的标准过程）
然后调用ITfCategoryMgr::RegisterCategory()方法为输入法添加类别。

```C++
BOOL RegisterCategories()
{
    ITfCategoryMgr *pCategoryMgr;
    HRESULT hr;

    hr = CoCreateInstance(CLSID_TF_CategoryMgr, NULL, CLSCTX_INPROC_SERVER, 
                          IID_ITfCategoryMgr, (void**)&pCategoryMgr);

    if (hr != S_OK)
        return FALSE;

    hr = pCategoryMgr->RegisterCategory(c_clsidTextService,
                                        GUID_TFCAT_TIP_KEYBOARD, 
                                        c_clsidTextService);

    pCategoryMgr->Release();
    return (hr == S_OK);
}
```

将输入法注册为键盘文本服务后，注册表添加如下键值：

![Category](img/Category.png)

>输入法注册为键盘文本服务后，本解决方案的所有工程，在Windows 11中，也可以正常演示了。

## 2.6.1 安装键盘事件接收器

将输入法注册为键盘文本服务后，只是让系统能够识别输入法，在相应过程中加载输入法。
例如，单独出现在语言栏中。

如果想要捕获键盘按键，还要像前面介绍的安装事件接收器那样，在激活输入法的时候，安装键盘事件接收器。

```C++
BOOL CTextService::_InitKeyEventSink()
{
    ITfKeystrokeMgr *pKeystrokeMgr;
    HRESULT hr;

    if (_pThreadMgr->QueryInterface(IID_ITfKeystrokeMgr, (void **)&pKeystrokeMgr) != S_OK)
        return FALSE;

    hr = pKeystrokeMgr->AdviseKeyEventSink(_tfClientId, (ITfKeyEventSink *)this, TRUE);

    pKeystrokeMgr->Release();

    return (hr == S_OK);
}
```