## 2.5.0 主要文件结构

- TextService.cpp
- LanguageBar.cpp
- InsertHello.cpp

本节介绍如何请求编辑会话，插入字符。

## 2.5.1 编辑会话

ITfContext上下文并不能直接修改内容添加文字，而是需要调用RequestEditSession()方法，传入一个ITfEditSession编辑会话。
由TSF管理器回调ITfEditSession编辑会话，才能实现对文本内容的修改。

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

最后，TSF管理器根据TF_ES_ASYNCDONTCARE，同步或异步回调ITfEditSession编辑会话。

## 2.5.2 插入文本

