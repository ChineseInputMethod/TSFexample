## 2.6.0 主要文件结构

- Server.cpp
  - Register.cpp
- TextService.cpp
- LanguageBar.cpp
  - Compartment.cpp
KeyEventSink.cpp
在Register.cpp文件中，添加了ITfCategoryMgr类别管理器的使用方法。
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

![Category](img/Category.png)