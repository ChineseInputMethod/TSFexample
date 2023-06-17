## 2.8.0 主要文件结构

- Server.cpp
  - Register.cpp

在Register.cpp文件中，添加了注册显示属性提供者类别。

### 2.8.1 注册显示属性提供者类别

可以像第6节介绍的注册键盘文本服务那样，注册显示属性提供者类别。

```C++
BOOL RegisterCategories()
{
    ITfCategoryMgr *pCategoryMgr;
    HRESULT hr;

    hr = CoCreateInstance(CLSID_TF_CategoryMgr, NULL, CLSCTX_INPROC_SERVER, 
                          IID_ITfCategoryMgr, (void**)&pCategoryMgr);

    if (hr != S_OK)
        return FALSE;

    //
    // register this text service to GUID_TFCAT_TIP_KEYBOARD category.
    //
    hr = pCategoryMgr->RegisterCategory(c_clsidTextService,
                                        GUID_TFCAT_TIP_KEYBOARD, 
                                        c_clsidTextService);

    //
    // register this text service to GUID_TFCAT_DISPLAYATTRIBUTEPROVIDER category.
    //注册显示属性提供者类别
    hr = pCategoryMgr->RegisterCategory(c_clsidTextService,
                                        GUID_TFCAT_DISPLAYATTRIBUTEPROVIDER, 
                                        c_clsidTextService);


    pCategoryMgr->Release();
    return (hr == S_OK);
}
```

输入法注册显示属性提供者类别之后，应用程序可以通过输入法提供的属性，显示输入组合的输入状态和转换状态。

### 2.8.2 