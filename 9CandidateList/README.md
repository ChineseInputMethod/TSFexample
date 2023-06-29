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

### 2.9.1 候选列表类

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

### 2.9.2 候选列表对象

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

### 2.9.3 候选列表对象创建过程