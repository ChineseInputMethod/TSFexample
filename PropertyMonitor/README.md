## 2.A.0 主要文件结构

- TextService.cpp
  - ThreadFocusSink.cpp
- DumpProperties.cpp
  - MemoryStream.cpp
  - PopupWindow.cpp

在ThreadFocusSink.cpp文件中，实现了线程输入焦点消息接收器。<br/>
在DumpProperties.cpp文件中，演示了如何监视显示属性。<br/>
在MemoryStream.cpp文件中，管理了一个内存流对象。<br/>
在PopupWindow.cpp文件中，监视了上下文中的显示属性。

本节介绍如何监视上下文属性，以及响应线程焦点消息。

本节输入法会随虚拟机启动被激活，所以再次调试时，需要先卸载本节输入法，然后重启虚拟机。以管理员身份启动CMD，然后执行如下命令：

>regsvr32.exe /u 输入法安装的目录\PropertyMonitor.dll

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
//## 2.A.3 枚举文档属性
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

在输入法激活的时候，创建了一个内存流对象，用于保存获取的属性，然后在监视窗口输出。

```C++
void CPropertyMonitorTextService::_DumpProperties(TfEditCookie ec, ITfContext *pContext)
{
IStream* CreateMemoryStream()
{
    LPSTREAM lpStream = NULL;

    // Create a stream object on a memory block.
    HGLOBAL hGlobal = GlobalAlloc(GMEM_MOVEABLE|GMEM_SHARE, 0);
    if (hGlobal != NULL)
    {
        HRESULT hr ;
        if (FAILED(hr = CreateStreamOnHGlobal(hGlobal, TRUE, &lpStream)))
        {
             GlobalFree(hGlobal);
        }
    }

    return lpStream;
}
```

每次调用_DumpProperties()函数的时候，先清除已有的流。

```C++
void ClearStream(IStream *pStream)
{
    ULARGE_INTEGER ull;
    ull.QuadPart = 0;
    pStream->SetSize(ull);
}
```

然后，在每次转储属性的时候调用AddStringToStream()函数，将属性保存到内存流中。

```C++
void AddStringToStream(IStream *pStream, const WCHAR *psz)
{
    pStream->Write(psz, lstrlenW(psz) * sizeof(WCHAR), NULL);
}
```

## 2.A.3 枚举文档属性

在_DumpProperties()函数中，首先获取IEnumTfProperties属性对象枚举器。

```C++
void CPropertyMonitorTextService::_DumpProperties(TfEditCookie ec, ITfContext *pContext)
{
    IEnumTfProperties *penum = NULL;
    ITfProperty *pprop;

    if (FAILED(pContext->EnumProperties(&penum)))
        goto Exit;
```

然后枚举文档属性。

```C++
    while (penum->Next(1, &pprop, NULL) == S_OK)
    {
        IEnumTfRanges *penumRanges;
        GUID guid;

        pprop->GetType(&guid);
        _DumpPropertyInfo(guid);
```

将文档属性的GUID和描述存储到内存流中。

```C++
void CPropertyMonitorTextService::_DumpPropertyInfo(REFGUID rguid)
{
    WCHAR sz[512];
    CLSIDToStringW(rguid, sz);
    AddStringToStream(_pMemStream, sz);

    int i = 0;
    while (!IsEqualGUID(*g_giKnownGuids[i].pguid, GUID_NULL))
    {
        if (IsEqualGUID(*g_giKnownGuids[i].pguid, rguid))
        {
            AddStringToStream(_pMemStream, L"\t");
            AddStringToStream(_pMemStream, g_giKnownGuids[i].szDesc);
            break;
        }
        i++;
    }
    AddStringToStream(_pMemStream, L"\r\n");
}
```

## 2.A.4 枚举文本范围