## 2.A.0 主要文件结构

- TextService.cpp
  - ThreadFocusSink.cpp
- DumpProperties.cpp
  - MemoryStream.cpp
  - PopupWindow.cpp

在ThreadFocusSink.cpp文件中，实现了线程输入焦点消息接收器。<br/>
在DumpProperties.cpp文件中，演示了如何监视显示属性。<br/>
在MemoryStream.cpp文件中，管理了一块内存流对象。<br/>
在PopupWindow.cpp文件中，监视了上下文中的显示属性。

本节介绍如何监视上下文属性，以及响应线程焦点消息。

>$(SystemRoot)\system32\regsvr32.exe /u
$(TargetPath)

## 2.A.1 演示的主要流程

当用任意输入法按下编码键后，本节输入法的ITfTextEditSink编辑会话完成消息接收器被调用。

```C++
STDAPI CPropertyMonitorTextService::OnEndEdit(ITfContext *pContext, TfEditCookie ecReadOnly, ITfEditRecord *pEditRecord)
{
    DumpProperties(pContext);
    return S_OK;
}
```

输入法创建一次编辑会话。

```C++
void CPropertyMonitorTextService::DumpProperties(ITfContext *pContext)
{
    CDumpPropertiesEditSession *pEditSession;
    HRESULT hr = E_FAIL;

    if ((pEditSession = new CDumpPropertiesEditSession(this, pContext)) == NULL)
        goto Exit;

    pContext->RequestEditSession(_tfClientId, pEditSession, TF_ES_ASYNCDONTCARE | TF_ES_READWRITE, &hr);

    pEditSession->Release();

Exit:
    return;
}
```

在ITfEditSession编辑会话中，调用_DumpProperties()函数转储上下文属性。

```C++
STDAPI CDumpPropertiesEditSession::DoEditSession(TfEditCookie ec)
{
    _pTextService->_DumpProperties(ec, _pContext);
    return S_OK;
}
```

_DumpProperties()函数是本节演示的主要内容，下面小节按顺序讲解该函数。

```C++
void CPropertyMonitorTextService::_DumpProperties(TfEditCookie ec, ITfContext *pContext)
{
    IEnumTfProperties *penum = NULL;
    ITfProperty *pprop;

    if (FAILED(pContext->EnumProperties(&penum)))
        goto Exit;
//## 2.A.2 内存流对象
    ClearStream(_pMemStream);
 
    while (penum->Next(1, &pprop, NULL) == S_OK)
    {
        IEnumTfRanges *penumRanges;
        GUID guid;

        pprop->GetType(&guid);
        _DumpPropertyInfo(guid);

        if (SUCCEEDED(pprop->EnumRanges(ec, &penumRanges, NULL)))
        {
            ITfRange *prange;
            while(penumRanges->Next(1, &prange, NULL) == S_OK)
            {
                _DumpPropertyRange(guid, ec, pprop, prange);
                prange->Release();
            }
            penumRanges->Release();
        }
        pprop->Release();
    }

    _ShowPopupWindow();

Exit:
    if (penum)
        penum->Release();
    return;
}
```

## 2.A.2 内存流对象