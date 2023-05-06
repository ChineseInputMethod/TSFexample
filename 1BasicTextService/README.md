## 2.0文件结构

- DllMain.cpp	DLL入口函数
  - Server.cpp	COM导出函数
    - Globals.cpp		CLSID全局标识符
    - Register.cpp	注册COM组件
- TextService.cpp 输入法核心

## 2.1COM组件的导出函数

函数|说明
-------- | -----
DllGetClassObject|PRIVATE
DllCanUnloadNow|PRIVATE
DllRegisterServer|PRIVATE
DllUnregisterServer|PRIVATE
