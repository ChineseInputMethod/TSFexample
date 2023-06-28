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

### 2.9.2 创建候选列表对象
