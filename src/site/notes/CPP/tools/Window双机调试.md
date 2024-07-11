---
{"dg-publish":true,"tags":["cpp"],"permalink":"/CPP/tools/Window双机调试/","dgPassFrontmatter":true,"noteIcon":"","created":"2024-07-06T00:36:29.868+08:00","updated":"2024-07-12T03:35:40.078+08:00"}
---


## 1.添加启动引导 ：

在 Windows 的启动管理器中添加一个新的启动项，专门用于调试。 

```cpp
​​​​bcdedit /copy {current} /d "WindowDebug"​​​​
```

创建了一个新的启动配置，复制了当前的配置，并将其命名为“WindowDebug”。

```cpp
bcdedit /displayorder {guid} /addlast​​​​
```
​
将新创建的启动项添加到启动菜单。​​​​{guid}​​​​需要替换为上一命令返回的标识符。  

```cpp
bcdedit /dbgsettings SERIAL DEBUGPORT:1 BAUDRATE:115200
```
​​​​​​​​
设置调试端口和波特率。这里使用的是串行端口1，波特率设置为115200。  

```cpp
bcdedit /debug {guid} ON​​​​
```
​​​​
启用调试模式，​​​​{guid}​​​​为新启动项的标识符。

```cpp
bcdedit /timeout 20​​​​
```

​​​​设置启动菜单的等待时间为20秒。  
## 2.虚拟机设置 ： 

如果您使用的是虚拟机进行调试，需要在虚拟机设置中添加串行设备，并配置命名管道。  

通常在虚拟机设置中可以找到添加硬件的选项，选择添加串行端口，并设置为“命名管道”模式。这里的​​​​./pipe/com_1​​​​是管道的名称。  
### 3.调试目标桌面属性配置 ： 

这一步是配置调试客户端的属性。添加配置 

```cpp
-k com:port=//./pipe/com_1,baud=115200,pipe
```
 
 在调试工具（如 WinDbg）的快捷方式中，添加对应的命令行参数，以便它能够通过命名管道连接到虚拟机。  
## 4.符号文件配置 ：

符号文件对于调试是非常重要的，它们提供了代码与其编译后的二进制表示之间的映射。  

```cpp
srv*D:\dev\symbol*;srv*D:\symbol*http://msdl.microsoft.com/download/symbols​​​​ 
```

​​​​是符号路径的配置，告诉调试器在哪里寻找符号文件。这个设置首先会在本地的 ​​​​D:\dev\symbol​​​​ 和 ​​​​D:\symbol​​​​ 目录下查找符号文件，如果没有找到，它会去Microsoft的符号服务器下载。