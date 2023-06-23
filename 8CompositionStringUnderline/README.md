## 2.8.0 主要文件结构

- Server.cpp
  - Register.cpp
- TextService.cpp
  - KeyHandler.cpp
    - DisplayAttribute.cpp
  - DisplayAttributeProvider.cpp
    - DisplayAttributeInfo.cpp
    - EnumDisplayAttributeInfo.cpp

在Register.cpp文件中，添加了注册显示属性提供者类别。<br/>
在KeyHandler.cpp文件中，添加了设置显示属性。<br/>
在DisplayAttributeProvider.cpp文件中，实现了ITfDisplayAttributeProvider显示属性提供者接口。<br/>
在DisplayAttributeInfo.cpp和EnumDisplayAttributeInfo.cpp文件中，实现了为应用程序提供的输入和转换显示属性。

本节介绍如何为输入组合提供显示属性。

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

### 2.8.2 注册显示属性标识符

显示属性是由应用程序获取使用的，因为COM机制和效率的原因，TSF框架并不直接通过GUID指针或引用来传递显示属性。
而是需要将显示属性的GUID注册为TfGuidAtom，以用于输入法设置和应用程序获取显示属性。

```C++
BOOL CTextService::_InitDisplayAttributeGuidAtom()
{
    ITfCategoryMgr *pCategoryMgr;
    HRESULT hr;

    if (CoCreateInstance(CLSID_TF_CategoryMgr,
                         NULL, 
                         CLSCTX_INPROC_SERVER, 
                         IID_ITfCategoryMgr, 
                         (void**)&pCategoryMgr) != S_OK)
    {
        return FALSE;
    }

    // register the display attribute for input text.
    hr = pCategoryMgr->RegisterGUID(c_guidDisplayAttributeInput, &_gaDisplayAttributeInput);

    // register the display attribute for the converted text.
    hr = pCategoryMgr->RegisterGUID(c_guidDisplayAttributeConverted, &_gaDisplayAttributeConverted);

    pCategoryMgr->Release();
        
    return (hr == S_OK);
}
```

### 2.8.3 设置显示属性

当用户按下编码键后，输入法将显示属性设置为输入状态。
当用户按下空格键后，输入法将显示属性设置为转换状态。

```C++
BOOL CTextService::_SetCompositionDisplayAttributes(TfEditCookie ec, ITfContext *pContext, TfGuidAtom gaDisplayAttribute)
{
    ITfRange *pRangeComposition;
    ITfProperty *pDisplayAttributeProperty;
    HRESULT hr;

    // A range and its context are required
    if (_pComposition->GetRange(&pRangeComposition) != S_OK)
        return FALSE;

    hr = E_FAIL;

    // get our the display attribute property
    if (pContext->GetProperty(GUID_PROP_ATTRIBUTE, &pDisplayAttributeProperty) == S_OK)
    {
        VARIANT var;
        // set the value over the range
        // the application will use this guid atom to lookup the acutal rendering information
        var.vt = VT_I4; // set a TfGuidAtom
        var.lVal = gaDisplayAttribute; 
//设置显示属性
        hr = pDisplayAttributeProperty->SetValue(ec, pRangeComposition, &var);

        pDisplayAttributeProperty->Release();
    }

    pRangeComposition->Release();
    return (hr == S_OK);
}
```

随后ITfDisplayAttributeProvider::GetDisplayAttributeInfo()方法会被调用。

### 2.8.4 显示属性提供者接口

ITfDisplayAttributeProvider显示属性提供者接口的GetDisplayAttributeInfo()方法，创建ITfDisplayAttributeInfo显示属性信息对象，提供给应用程序。

```C++
STDAPI CTextService::GetDisplayAttributeInfo(REFGUID guidInfo, ITfDisplayAttributeInfo **ppInfo)
{
    if (ppInfo == NULL)
        return E_INVALIDARG;

    *ppInfo = NULL;

    // Which display attribute GUID?
    if (IsEqualGUID(guidInfo, c_guidDisplayAttributeInput))
    {
        if ((*ppInfo = new CDisplayAttributeInfoInput()) == NULL)
            return E_OUTOFMEMORY;
    }
    else if (IsEqualGUID(guidInfo, c_guidDisplayAttributeConverted))
    {
        if ((*ppInfo = new CDisplayAttributeInfoConverted()) == NULL)
            return E_OUTOFMEMORY;
    }
    else
    {
        return E_INVALIDARG;
    }


    return S_OK;
}
```

### 2.8.5 显示属性信息对象

应用程序调用ITfDisplayAttributeInfo显示属性信息对象的GetAttributeInfo()方法，获取显示属性信息。

```C++
STDAPI CDisplayAttributeInfo::GetAttributeInfo(TF_DISPLAYATTRIBUTE *ptfDisplayAttr)
{
    HKEY hKeyAttributeInfo;
    LONG lResult;
    DWORD cbData;

    if (ptfDisplayAttr == NULL)
        return E_INVALIDARG;

    if (_pszValueName == NULL)
        return E_FAIL;

    lResult = E_FAIL;

    if (RegOpenKeyEx(HKEY_CURRENT_USER, c_szAttributeInfoKey, 0, KEY_READ, &hKeyAttributeInfo) == ERROR_SUCCESS)
    {
        cbData = sizeof(*ptfDisplayAttr);

        lResult = RegQueryValueEx(hKeyAttributeInfo, _pszValueName,
                                  NULL, NULL,
                                  (LPBYTE)ptfDisplayAttr, &cbData);

        RegCloseKey(hKeyAttributeInfo);
    }

    if (lResult != ERROR_SUCCESS || cbData != sizeof(*ptfDisplayAttr))
    {
        // return the default display attribute.
        *ptfDisplayAttr = *_pDisplayAttribute;
    }

    return S_OK;
}
```

### 2.8.6 清除显示属性

当用户按下回车键，输入法调用ITfProperty::Clear()方法，清除显示属性。

```C++
void CTextService::_ClearCompositionDisplayAttributes(TfEditCookie ec, ITfContext *pContext)
{
    ITfRange *pRangeComposition;
    ITfProperty *pDisplayAttributeProperty;

    // get the compositon range.
    if (_pComposition->GetRange(&pRangeComposition) != S_OK)
        return;

    // get our the display attribute property
    if (pContext->GetProperty(GUID_PROP_ATTRIBUTE, &pDisplayAttributeProperty) == S_OK)
    {
        // clear the value over the range
        pDisplayAttributeProperty->Clear(ec, pRangeComposition);

        pDisplayAttributeProperty->Release();
    }

    pRangeComposition->Release();
}
```