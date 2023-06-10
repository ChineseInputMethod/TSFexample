## 2.7.0 主要文件结构

- TextService.cpp
  - TextEditSink.cpp
  - KeyEventSink.cpp
- KeyHandler.cpp
- StartComposition.cpp
- EndComposition.cpp
- Composition.cpp

在TextEditSink.cpp文件中，添加了ITfTextEditSink编辑会话完成消息接收器的终止输入组合内容。<br/>
在KeyHandler.cpp文件中，实现了对按键的处理过程。<br/>
本节在StartComposition.cpp、EndComposition.cpp和Composition.cpp文件中，介绍ITfComposition输入组合的生命周期。

## 2.7.1 完整输入的处理过程

模拟一次完整的输入处理过程如下：<br/>
当用户按下编码键后，在OnTestKeyDown()函数中判断，这个按键是否需要输入法进行处理。

```C++
STDAPI CTextService::OnTestKeyDown(ITfContext *pContext, WPARAM wParam, LPARAM lParam, BOOL *pfEaten)
{
    *pfEaten = _IsKeyEaten(wParam);
    return S_OK;
}
```

当*pfEaten = TRUE时，TSF管理器调用OnKeyDown()函数。

```C++
STDAPI CTextService::OnKeyDown(ITfContext *pContext, WPARAM wParam, LPARAM lParam, BOOL *pfEaten)
{
    *pfEaten = _IsKeyEaten(wParam);

    if (*pfEaten)
    {
        _InvokeKeyHandler(pContext, wParam, lParam);
    }
    return S_OK;
}
```

输入法在_InvokeKeyHandler()函数中，请求ITfEditSession编辑会话。

```C++
HRESULT CTextService::_InvokeKeyHandler(ITfContext *pContext, WPARAM wParam, LPARAM lParam)
{
    CKeyHandlerEditSession *pEditSession;
    HRESULT hr = E_FAIL;

    // we'll insert a char ourselves in place of this keystroke
    if ((pEditSession = new CKeyHandlerEditSession(this, pContext, wParam)) == NULL)
        goto Exit;

    // A lock is required
    // nb: this method is one of the few places where it is legal to use
    // the TF_ES_SYNC flag
    hr = pContext->RequestEditSession(_tfClientId, pEditSession, TF_ES_SYNC | TF_ES_READWRITE, &hr);

    pEditSession->Release();

Exit:
    return hr;
}
```

TSF管理器调用ITfEditSession编辑会话接口，执行相应的处理过程。

```C++
STDAPI CKeyHandlerEditSession::DoEditSession(TfEditCookie ec)
{
    switch (_wParam)
    {
        case VK_LEFT:
        case VK_RIGHT:
            return _pTextService->_HandleArrowKey(ec, _pContext, _wParam);

        case VK_RETURN:
            return _pTextService->_HandleReturnKey(ec, _pContext);

        default:
            if (_wParam >= 'A' && _wParam <= 'Z')
                return _pTextService->_HandleCharacterKey(ec, _pContext, _wParam);
            break;
    }

    return S_OK;

}
```

重复以上步骤，直到用户按下回车键时，调用_TerminateComposition()函数，终止ITfComposition输入组合。

```C++
HRESULT CTextService::_HandleReturnKey(TfEditCookie ec, ITfContext *pContext)
{
    // just terminate the composition
    _TerminateComposition(ec);
    return S_OK;
}
```

下图是原文档中描述这一过程的流程图。

![flowchart](doc/handlekeyflowchart.JPG)

>与简体中文输入法不同，日本语输入法，繁体中文输入法与早期的微软拼音和智能ABC输入法，都是按回车键将候选字上屏。

## 2.7.2 输入组合的生命周期

>输入法并不直接将汉字发送给应用程序。而是通过一系列步骤，完成输入。<br/>
在空间上，输入法通过ITfInsertAtSelection在选定位置插入内容获得ITfRange文本范围，将ITfRange文本范围保存到TF_SELECTION文本选定数据中，然后将TF_SELECTION文本选定数据设置到ITfContext上下文中。<br/>
在时间上，输入法从ITfContext上下文中获得ITfContextComposition上下文输入组合，在ITfContextComposition上下文输入组合中，创建ITfComposition输入组合。<br/>
在ITfComposition输入组合生命周期中，操作ITfRange文本范围，实现汉字的输入。

### 2.7.2.1 输入组合的创建

当用户每回首次按下编码键时，将创建ITfComposition输入组合。

```C++
STDAPI CStartCompositionEditSession::DoEditSession(TfEditCookie ec)
{
    ITfInsertAtSelection *pInsertAtSelection = NULL;
    ITfRange *pRangeInsert = NULL;
    ITfContextComposition *pContextComposition = NULL;
    ITfComposition *pComposition = NULL;
    HRESULT hr = E_FAIL;
//获取ITfInsertAtSelection在选定位置插入内容接口
    // A special interface is required to insert text at the selection
    if (_pContext->QueryInterface(IID_ITfInsertAtSelection, (void **)&pInsertAtSelection) != S_OK)
    {
        goto Exit;
    }
//通过插入一个空字符，获得ITfRange文本范围
    // insert the text
    if (pInsertAtSelection->InsertTextAtSelection(ec, TF_IAS_QUERYONLY, NULL, 0, &pRangeInsert) != S_OK)
    {
        goto Exit;
    }
//从上下文中获取一个ITfContextComposition上下文输入组合对象
    // get an interface on the context to deal with compositions
    if (_pContext->QueryInterface(IID_ITfContextComposition, (void **)&pContextComposition) != S_OK)
    {
        goto Exit;
    }
//创建新的输入组合
    // start the new composition
    if ((pContextComposition->StartComposition(ec, pRangeInsert, _pTextService, &pComposition) == S_OK) && (pComposition != NULL))
    {//保存当前输入组合对象，输入法处于编码合成状态
        // Store the pointer of this new composition object in the instance 
        // of the CTextService class. So this instance of the CTextService 
        // class can know now it is in the composition stage.
        _pTextService->_SetComposition(pComposition);

        // 
        //  set selection to the adjusted range
        // 将当前输入组合状态设置到上下文中
        TF_SELECTION tfSelection;
        tfSelection.range = pRangeInsert;
        tfSelection.style.ase = TF_AE_NONE;
        tfSelection.style.fInterimChar = FALSE;
        _pContext->SetSelection(ec, 1, &tfSelection);
    }

Exit:
    if (pContextComposition != NULL)
        pContextComposition->Release();

    if (pRangeInsert != NULL)
        pRangeInsert->Release();

    if (pInsertAtSelection != NULL)
        pInsertAtSelection->Release();

    return S_OK;
}
```

### 2.7.2.2 输入组合与预上屏

>如果没有相应的背景知识，很难理解为什么会有预上屏和输入组合的概念。<br/>
电脑处理字母文字是相对简单的，键盘按键对应着相应字母。不同语言的文字，只要改变操作系统的内码页和键盘的驱动就可以了。<br/>
但是处理起某些东亚文字，这里特指中日韩文字，就非常困难了。因为键盘上没有那么多按键，来对应相应的文字字母。<br/>
最直接的解决方案是扩大键盘，日本发明了自己的日本语键盘。中文也在这方面探索过，林语堂的明快打字机就是这方面的代表。<br/>
繁体中文输入法沿袭了这种设计思想，仓颉、大千等输入法的键盘都像日本语键盘一样“大”。<br/>

>相对于大键盘，另一条技术路线叫罗马字。汉语拼音方案可以看成汉语罗马字的继承者。<br/>
汉语拼音方案违反了语音学的一音一符原则，故意设置了很多错误拼音。这是因为其并不是为了拼音设计的，其设计目的一直是取代汉字。
但是这种设计思想，恰好非常利于电脑键盘打字。（有很多原因，这里不展开了）

>现在的电脑端，日本也用罗马字输入法打字。将按键组合成一个平假名，这就是ITfComposition输入组合，这个术语的由来。<br/>

```C++
HRESULT CTextService::_HandleCharacterKey(TfEditCookie ec, ITfContext *pContext, WPARAM wParam)
{
    ITfRange *pRangeComposition;
    TF_SELECTION tfSelection;
    ULONG cFetched;
    WCHAR ch;
    BOOL fCovered;
//创建输入组合
    // Start the new compositon if there is no composition.
    if (!_IsComposing())
        _StartComposition(pContext);

    //这里将按键直接转换为了字符，输入法要在这里进行编码处理
    // Assign VK_ value to the char. So the inserted the character is always
    // uppercase.
    //
    ch = (WCHAR)wParam;

    // first, test where a keystroke would go in the document if an insert is done
    if (pContext->GetSelection(ec, TF_DEFAULT_SELECTION, 1, &tfSelection, &cFetched) != S_OK || cFetched != 1)
        return S_FALSE;

    // is the insertion point covered by a composition?
    if (_pComposition->GetRange(&pRangeComposition) == S_OK)
    {
        fCovered = IsRangeCovered(ec, tfSelection.range, pRangeComposition);

        pRangeComposition->Release();

        if (!fCovered)
        {
            goto Exit;
        }
    }
//将转换后的字符写入文本范围
    // insert the text
    // use SetText here instead of InsertTextAtSelection because a composition has already been started
    // Don't allow to the app to adjust the insertion point inside the composition
    if (tfSelection.range->SetText(ec, 0, &ch, 1) != S_OK)
        goto Exit;
//更新文本范围和文本范围的结束定位点
    // update the selection, we'll make it an insertion point just past
    // the inserted text.
    tfSelection.range->Collapse(ec, TF_ANCHOR_END);
    pContext->SetSelection(ec, 1, &tfSelection);

Exit:
    tfSelection.range->Release();
    return S_OK;
}
```

>初次接触TSF输入法编程的人，可能会这样认为：编码显示在编码窗口中；候选字显示在候选窗口中；还有一个输入法状态条。<br/>
调试过本解决方案中的输入法的人可能已经注意到了，微软总是默认将语言栏隐藏起来。微软不希望桌面中出现输入法状态栏这样的东西。<br/>
同样，微软也希望编码输入窗口和候选窗口，也处于自己的控制之下。理由有很多，除了前面提到的东亚文字需要编码转换外，还有一点是微软推崇的：所见即所得。<br/>
例如，很多文本编辑器的撰写和预览是分开的，而微软的理念是：用户在撰写的时候，就能看到预览时的样子。<br/>
具体到输入法上，微软希望输入编码和候选字与正在撰写的文章字体是一致的。<br/>
再比如，用户需要在一款全屏游戏中输入文字，这时如果显示输入法的UI窗口，微软（和游戏开发商）认为这样会破坏用户的沉浸感。<br/>
所以，微软把输入法的编码输入窗口，实现为了ITfComposition输入组合接口。微软拼音输入法中，带下划线的那个拼音字符串，就是典型的输入组合接口。<br/>
用户可以输入一个拼音串，并在这个拼音串中自造词。但此时，无论是拼音串，还是自造词，并没有真正发送给应用程序，只有在ITfComposition输入组合终止后，汉字才真正的发送给应用程序。<br/>
这就是预上屏。关于输入组合和预上屏，在本章后面两节还会继续阐述。

>本节内容，是本章核心内容。输入法就是通过本节所介绍的内容，将汉字发送给应用程序的。

如下图设置跟踪：

![Composition](img/Composition.png)

会发现，表面上字符被写到了上下文中，实际上需要在ITfComposition输入组合中，进行编码处理。直到ITfComposition输入组合终止，汉字才真正发送给应用程序，这就是预上屏的概念。

ITfComposition输入组合和ITfRange文本范围，是TSF编程中，最难以理解的部分。

### 2.7.2.3 输入组合的正常终止

### 2.7.2.4 输入组合的非正常终止

## 2.7.3 输入组合的意外终止