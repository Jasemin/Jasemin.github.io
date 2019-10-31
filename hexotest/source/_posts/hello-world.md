---
title: iOS的crash日志符号化操作
categories: 
    - Crash分析
tags: 
    - iOS
---
在我们日常开发中，总有数不清的bug产出，有些bug能够清楚的通过崩溃日志的调用栈清楚的定位到bug出现的位置，从而方便的进行修复，然而往往很多bug的崩溃日志会出现很多我们所不能理解的符号，这时候我们就需要对crash日志进行符号化。

<!--more-->


# 需要提前准好的文件
1.xxx.crash (崩溃日志)
2.xxx..app.dSYM (打包的时候生成的符号表)
3.symbolicatecrash （Xcode的符号化工具）

- symbolicatecrash的获取方式：
```
xxx.app.dSYM duoyi$ find /Users/duoyi/Desktop/Xcode-beta.app -name symbolicatecrash -type f
```
- 将上述提到的文件全部复制到一个叫crash的文件夹下；
- 要确保xxx..app.dSYM的UUID和xxx.crash的UUID相同；
- 查看UUID的方式：
```
dwarfdump --uuid OAAssistant.app.dSYM
```
- 得到以下UUID（需要注意的是现在我们绝大部分设备都是arm64的）：
```
UUID: 6D6C05ED-8109-33AC-8501-FB133B55EBA8 (armv7) OAAssistant.app.dSYM/Contents/Resources/DWARF/OAAssistant
UUID: 3E593480-AEE8-3316-8C63-6AB1A92E7435 (arm64) OAAssistant.app.dSYM/Contents/Resources/DWARF/OAAssistant
```
- 查看我们crash的日志（找到Binary Image）；
```
Binary Images:
0x1001e4000 - 0x100ba7fff OAAssistant arm64  <3e593480aee833168c636ab1a92e7435> /var/containers/Bundle/Application/398894C1-15CE-4286-B286-58DDA256E202/OAAssistant.app/OAAssistant
```
- 这里可以看出我们的UUID是相同的。

# 执行符号化操作
- 打开terminal终端
- 进入到crash文件下
- 执行命令：
```
crash duoyi$ ./symbolicatecrash xxx.crash xxx.app.dSYM > xxx.crash
```
- 在这里可能会出现这样的问题：
```
Error: "DEVELOPER_DIR" is not defined at ./symbolicatecrash line 69.
```
- 这时我们需要对环境变量进行修改（需要注意的是后面的路径是你电脑上的Xcode下的Developer路径）：
```
crash duoyi$ export DEVELOPER_DIR="/Applications/XCode.app/Contents/Developer"
```
- 这时候我们再执行上一步符号化操作，就能够在crash文件夹下得到一个叫做“xxx.crash”的经过符号化后的崩溃日志了；
- 有的时候我们可能会得到这样的一个错误提示：
```
No symbolic information found
```
- 这样的话我们就需要仔细检查再次确认我们的UUID是否一致了，即使版号相同但是不是同一次打包UUID也可能不同。
