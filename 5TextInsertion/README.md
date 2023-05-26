## 2.5.0 主要文件结构

- TextService.cpp
- LanguageBar.cpp
- InsertHello.cpp

编辑会话组件在InsertHello.cpp文件中。
本节介绍如何请求编辑会话，插入字符。

## 2.5.1 请求编辑会话

ITfContext上下文并不能直接修改内容添加文字，而是需要调用RequestEditSession()方法，传入一个ITfEditSession编辑会话。
由TSF管理器回调ITfEditSession编辑会话，才能实现对文本内容的修改。

点击输入法图标菜单项，本节将演示如何请求编辑会话，插入字符。

首先，获取ITfContext上上下文。

```C++
ITfDocumentMgr *pDocMgrFocus;
ITfContext *pContext;
CInsertHelloEditSession *pInsertHelloEditSession;
HRESULT hr;

// get the focus document
if (_pThreadMgr->GetFocus(&pDocMgrFocus) != S_OK)
	return;

// get the topmost context, since the main doc context could be
// superseded by a modal tip context
if (pDocMgrFocus->GetTop(&pContext) != S_OK)
{
	pContext = NULL;
	goto Exit;
}
```

然后，创建ITfEditSession编辑会话。

```C++
if (pInsertHelloEditSession = new CInsertHelloEditSession(pContext))
```

接着，调用RequestEditSession()方法，请求编辑会话。

```C++
{
	//A document write lock is required to insert text
	// the CInsertHelloEditSession will do all the work when the
	// CInsertHelloEditSession::DoEditSession method is called by the context
	pContext->RequestEditSession(_tfClientId, pInsertHelloEditSession, TF_ES_READWRITE | TF_ES_ASYNCDONTCARE, &hr);

	pInsertHelloEditSession->Release();
}
```

>在输入法被激活时，保存了TfClientId客户端标识符，该标识符用于输入法在调用各种TSF管理器方法时标识自身。

## 2.5.2 在编辑会话中插入文本

当输入法调用ITfContext::RequestEditSession()方法，启动编辑会话后，
TSF管理器根据传入的参数TF_ES_ASYNCDONTCARE，同步或异步回调ITfEditSession::DoEditSession(TfEditCookie ec)方法。

>TfEditCookie数据类型由TSF管理器提供，用于在编辑会话中标识读写锁。

有两种方式输入字符，本节演示ITfInsertAtSelection在选定位置插入内容。

首先，获取ITfInsertAtSelection接口。

```C++
if (pContext->QueryInterface(IID_ITfInsertAtSelection, (void **)&pInsertAtSelection) != S_OK)
	return;
```

然后，将字符写入ITfRange文本范围。

```C++
if (pInsertAtSelection->InsertTextAtSelection(ec, 0, pchText, cchText, &pRange) != S_OK)
	goto Exit;
```

这样，就完成了字符的输入。

但实际上输入法并不按照这种方式输入汉字，而是使用ITfRange::SetText()方法输入汉字。
ITfInsertAtSelection接口一般用来插入一个空字符。

从下一节开始，介绍从按下按键到输出汉字，输入法的工作步骤。