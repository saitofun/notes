# Java Case Analysis

## Java Stack Trace分析方法

### 如何产生Java Stack Trace

### 进行分析的第一步

### 确定软件版本和运行平台

### 判断线程的状态

### 检查Monitors

### 实践

### 专家建议

  对于hanging、deadlocked、frozen程序：如果你认为你的程序处于hanging状态，生成一个Java Stack Trace，检查处于MW或者CW状态的线程。如果程序处于deadlocked状态，则系统的一些线程会以“current thread”的状态出现，因为对于他们来说没有事情可做了！

对于crashed、aborted程序：在Unix平台，查找Core文件。使用本地调试工具进行调试，例如gdb或者dbx。找到调用本地方法的线程。因为Java本身是内存安全的，出问题的地方可能是在本地方法调用上面。

对于busy的程序：对于处于busy状态的程序，最好的办法就是多次生成Java Stack Trace，然后缩小可疑代码范围，然后进行研究。

