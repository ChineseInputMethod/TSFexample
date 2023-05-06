## 2.0文件结构

- DllMain.cpp
- Server.cpp
  - Globals.cpp
  - Register.cpp
- TextService.cpp

## 2.1COM组件的导出函数

函数|说明
-------- | -----
DllGetClassObject|获取COM组件对象
DllCanUnloadNow|查询能否注销COM组件
DllRegisterServer|注册COM组件
DllUnregisterServer|注销COM组件

## 2.2注册COM组件
