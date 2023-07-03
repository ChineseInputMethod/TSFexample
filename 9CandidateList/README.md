## 2.9.0 主要文件结构

- TextService.cpp
  - KeyEventSink.cpp
    - KeyHandler.cpp
CandidateList.cpp
CandidateWindow.cpp

在KeyHandler.cpp文件中，添加了创建候选列表对象。<br/>
在CandidateList.cpp文件中，实现了候选列表类。<br/>
在CandidateWindow.cpp文件中，定义了候选列表窗口类。<br/>

本节介绍如何实现一个基本的候选窗口，本节未实现ITfCandidateList接口。

## 2.9.1 候选列表类

如同MFC风格的应用程序那样，TSF候选窗口的实现，也是UI、数据和控制相分离的。
在CCandidateList候选列表类中，继承了ITfContextKeyEventSink上下文键盘事件接收器和ITfTextLayoutSink文本布局消息接收器。

```C++
class CCandidateList : public ITfContextKeyEventSink,
                       public ITfTextLayoutSink
{
public:
    CCandidateList(CTextService *pTextService);
    ~CCandidateList();

    //
    // IUnknown methods
    //
    STDMETHODIMP QueryInterface(REFIID riid, void **ppvObj);
    STDMETHODIMP_(ULONG) AddRef(void);
    STDMETHODIMP_(ULONG) Release(void);

    //
    // ITfContextKeyEventSink
    //
    STDMETHODIMP OnKeyDown(WPARAM wParam, LPARAM lParam, BOOL *pfEaten);
    STDMETHODIMP OnKeyUp(WPARAM wParam, LPARAM lParam, BOOL *pfEaten);
    STDMETHODIMP OnTestKeyDown(WPARAM wParam, LPARAM lParam, BOOL *pfEaten);
    STDMETHODIMP OnTestKeyUp(WPARAM wParam, LPARAM lParam, BOOL *pfEaten);

    //
    // ITfTextLayoutSink
    //
    STDMETHODIMP OnLayoutChange(ITfContext *pContext, TfLayoutCode lcode, ITfContextView *pContextView);
```

当候选窗口开启时，由候选窗口类的上下文键盘事件接收器处理键盘事件。通过候选窗口类的文本布局消息接收器实现候选窗口的光标跟随。

## 2.9.2 候选列表对象

>学了很多年C，一直感觉不到自己有多少进步。直到看了《C Primer Plus》，感觉自己终于算是入了一点门。
很多年前就看过C++，实在是学不会。看过《C Primer Plus》后，再次鼓起勇气学习C++，乘胜追击看《C++ Primer Plus》。
仿佛打开了新世界大门，困扰我很多年的问题，原来答案一直在书里写着。有一种被夺舍了的感觉，书里这么清清楚楚的描述，我竟然浑浑噩噩的学了这么多年。

>其中一个重要原因就是专业知识与科普知识的区别。我有一个象相像理论。
象，是事物的本相，是知识概念的本身。
相，是事物的表象，是对事物的描述。
像，是人通过表象对事物的认知。

>专业知识是由专业知识点链接而成的。掌握一个概念，需要掌握相关的专业知识背景。这一过程是枯燥乏味的。
而科普知识，是用你听得懂的语言描述的。其中一个最显著的特点是，用一个你熟知的概念来类比。
这就形成了认知的偏差，你以为的，仅仅是你以为的，或者是别人让你以为的。

>一直不敢看《C++ Primer》，因为都说这本书很难，我感觉凭我的水平，看了又是浪费功夫。但是，现在开发要求我必须掌握C++。
我目前的水平有点不够用，在重读《C++ Primer Plus》和《C++ Primer》之间，我选择了《C++ Primer》。才发现了原来《C++ Primer》并不难啊。
只是需要有C++基础，有了基础，我发现这是一本最清晰的书。

>例如类和对象的概念，在别的书中，讲了很多什么对现实的抽象。这些看似容易理解的概念其实是科普知识，并不涉及专业知识。
而《C++ Primer》的相关介绍只有短短的一句话，类就是类型，对象就是变量。

输入编码后，按下空格键，将创建一个候选列表对象。

```C++
HRESULT CTextService::_HandleSpaceKey(TfEditCookie ec, ITfContext *pContext)
{
    //
    // set the display attribute to the composition range.
    //
    // The real text service may have linguistic logic here and set 
    // the specific range to apply the display attribute rather than 
    // applying the display attribute to the entire composition range.
    //
    _SetCompositionDisplayAttributes(ec, pContext, _gaDisplayAttributeConverted);

    // 
    // create an instance of the candidate list class.
    // 
    if (_pCandidateList == NULL)
        _pCandidateList = new CCandidateList(this);

    // 
    // The document manager object is not cached. Get it from pContext.
    // 
    ITfDocumentMgr *pDocumentMgr;
    if (pContext->GetDocumentMgr(&pDocumentMgr) == S_OK)
    {
        // 
        // get the composition range.
        // 
        ITfRange *pRange;
        if (_pComposition->GetRange(&pRange) == S_OK)
        {
            _pCandidateList->_StartCandidateList(_tfClientId, pDocumentMgr, pContext, ec, pRange);
            pRange->Release();
        }
        pDocumentMgr->Release();
    }
    return S_OK;
}
```

## 2.9.3 候选列表对象创建过程

本小节对CCandidateList::_StartCandidateList()函数进行拆分讲解。

### 2.9.3.1 清除候选列表

首先清除以请存在的候选列表。

```C++
HRESULT CCandidateList::_StartCandidateList(TfClientId tfClientId, ITfDocumentMgr *pDocumentMgr, ITfContext *pContextDocument, TfEditCookie ec, ITfRange *pRangeComposition)
{
    TfEditCookie ecTmp;
    HRESULT hr = E_FAIL;
    BOOL fClipped;

    //
    // clear the previous candidate list.
    // only one candidate window is supported.
    //
    _EndCandidateList();
```

### 2.9.3.2 创建上下文

该部分在DOC目录下，原文档有详细讲解。

```C++
    //
    // create a new context on the document manager object for
    // the candidate ui.
    //
    if (FAILED(pDocumentMgr->CreateContext(tfClientId, 0, NULL, &_pContextCandidateWindow, &ecTmp)))
        return E_FAIL;

    //
    // push the new context. 
    //
    if (FAILED(pDocumentMgr->Push(_pContextCandidateWindow)))
        goto Exit;

    _pDocumentMgr = pDocumentMgr;
    _pDocumentMgr->AddRef();

    _pContextDocument = pContextDocument;
    _pContextDocument->AddRef();

    _pRangeComposition = pRangeComposition;
    _pRangeComposition->AddRef();
```

### 2.9.3.3 安装上下文键盘事件接收器

在新创建的上下文中安装上下文键盘事件接收器。

```C++
    // 
    // advise ITfContextKeyEventSink to the new context.
    // 
    if (FAILED(_AdviseContextKeyEventSink()))
        goto Exit;
```

### 2.9.3.4 安装文本布局消息接收器

在文档上下文中安装文本布局消息接收器。

```C++
    // 
    // advise ITfTextLayoutSink to the document context.
    // 
    if (FAILED(_AdviseTextLayoutSink()))
        goto Exit;
```

### 2.9.3.5 创建候选列表窗口

本段代码演示了，如何创建一个在输入组合下方显示的候选列表窗口。

```C++
    // 
    // create an instance of CCandidateWindow class.
    //
    if (_pCandidateWindow = new CCandidateWindow())
    {
        RECT rc;
        ITfContextView *pContextView;

        //
        // get an active view of the document context.
        //
        if (FAILED(pContextDocument->GetActiveView(&pContextView)))
            goto Exit;

        //
        // get text extent for the range of the composition.
        //
        if (FAILED(pContextView->GetTextExt(ec, pRangeComposition, &rc, &fClipped)))
            goto Exit;

        pContextView->Release();

        
        //
        // create the dummy candidate window
        //
        if (!_pCandidateWindow->_Create())
            goto Exit;

        _pCandidateWindow->_Move(rc.left, rc.bottom);
        _pCandidateWindow->_Show();

        hr = S_OK;
    }

Exit:
    if (FAILED(hr))
    {
        _EndCandidateList();
    }
    return hr;
}
```

## 2.9.4 处理上下文键盘事件

请比较KeyEventSink.cpp文件中的_InitKeyEventSink()函数和以下代码内容。

```C++
HRESULT CCandidateList::_AdviseContextKeyEventSink()
{
    HRESULT hr;
    ITfSource *pSource = NULL;

    hr = E_FAIL;

    if (FAILED(_pContextCandidateWindow->QueryInterface(IID_ITfSource, (void **)&pSource)))
        goto Exit;

    if (FAILED(pSource->AdviseSink(IID_ITfContextKeyEventSink, (ITfContextKeyEventSink *)this, &_dwCookieContextKeyEventSink)))
        goto Exit;

    hr = S_OK;

Exit:
    if (pSource != NULL)
        pSource->Release();
    return hr;
}
```

将上下文键盘事件注册到新创建的上下文中后，上下文键盘事件接收器也会收到键盘事件。
所以，本节示例代码的逻辑是，当候选列表窗口开启时，键盘事件接收器不处理键盘事件。
相关代码如下：

```C++
BOOL CTextService::_IsKeyEaten(ITfContext *pContext, WPARAM wParam)
{
    // if the keyboard is disabled, keys are not consumed.
    if (_IsKeyboardDisabled())
        return FALSE;

    // if the keyboard is closed, keys are not consumed.
    if (!_IsKeyboardOpen())
        return FALSE;

    //
    // The text service key handler does not do anything while the candidate
    // window is shown.
    // The candidate list handles the keys through ITfContextKeyEventSink.
    //如果候选窗口开启，文本服务不再处理键盘事件。
    if (_pCandidateList &&
        _pCandidateList->_IsContextCandidateWindow(pContext))
    {
        return FALSE;
    }
```

## 2.9.5 处理文本布局消息

当文本布局发生改变时，需要重绘候选列表窗口。本节演示的是获取屏幕坐标，实现一个光标跟随的候选列表窗口。

### 2.9.5.1 跟踪文本布局消息

当文本布局发生改变时，CCandidateList::OnLayoutChange()方法被调用。
输入法创建一个ITfEditSession编辑会话对象，请求编辑会话。

```C++
STDAPI CCandidateList::OnLayoutChange(ITfContext *pContext, TfLayoutCode lcode, ITfContextView *pContextView)
{
    if (pContext != _pContextDocument)
        return S_OK;

    switch (lcode)
    {
        case TF_LC_CHANGE:
            if (_pCandidateWindow != NULL)
            {
                CGetTextExtentEditSession *pEditSession;

                if ((pEditSession = new CGetTextExtentEditSession(_pTextService, pContext, pContextView, _pRangeComposition, _pCandidateWindow)) != NULL)
                {
                    HRESULT hr;
                    // a lock is required
                    // nb: this method is one of the few places where it is legal to use
                    // the TF_ES_SYNC flag
                    pContext->RequestEditSession(_pTextService->_GetClientId(), pEditSession, TF_ES_SYNC | TF_ES_READ, &hr);

                    pEditSession->Release();
                 }
            }
            break;

        case TF_LC_DESTROY:
            _EndCandidateList();
            break;

    }
    return S_OK;
}
```

### 2.9.5.2 获取屏幕坐标

输入法调用_pContextView->GetTextExt()函数，获取_pRangeComposition文本范围的屏幕坐标。

```C++
STDAPI CGetTextExtentEditSession::DoEditSession(TfEditCookie ec)
{
    RECT rc;
    BOOL fClipped;

    if (SUCCEEDED(_pContextView->GetTextExt(ec, , &rc, &fClipped)))
        _pCandidateWindow->_Move(rc.left, rc.bottom);
    return S_OK;
}
```

### 2.9.5.3 移动候选列表窗口

移动候选列表窗口到输入组合起点的左下方。

```C++
void CCandidateWindow::_Move(int x, int y)
{
    if (_hwnd != NULL)
    {
        RECT rc;
        GetWindowRect(_hwnd, &rc);
        MoveWindow(_hwnd, x, y, rc.right - rc.left, rc.bottom - rc.top, TRUE);
    }
}
```

### 2.9.6 终止候选列表

当释放回车键时，输入法调用CCandidateList::_EndCandidateList()方法，终止候选列表。

```C++
void CCandidateList::_EndCandidateList()
{
    if (_pCandidateWindow)
    {
        _pCandidateWindow->_Destroy();
        delete _pCandidateWindow;
        _pCandidateWindow = NULL;
    }

    if (_pRangeComposition)
    {
       _pRangeComposition->Release();
       _pRangeComposition = NULL;
    }

    if (_pContextCandidateWindow)
    {
       _UnadviseContextKeyEventSink();
       _pContextCandidateWindow->Release();
       _pContextCandidateWindow = NULL;
    }

    if (_pContextDocument)
    {
       _UnadviseTextLayoutSink();
       _pContextDocument->Release();
       _pContextDocument = NULL;
    }

    if (_pDocumentMgr)
    {
       _pDocumentMgr->Pop(0);
       _pDocumentMgr->Release();
       _pDocumentMgr = NULL;
    }
}
```