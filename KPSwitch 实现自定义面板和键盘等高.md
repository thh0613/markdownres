## KPSwitch 源码分析

有一个这样的需求，点击输入框显示键盘，点击输入框旁边的按钮，显示表情，且需要表情和键盘高度一致。最终实现的效果如图所示：

![https://raw.githubusercontent.com/southmoney/markdownres/master/gif.gif](https://raw.githubusercontent.com/southmoney/markdownres/master/gif.gif)

在github上找到一个库，可以实现键盘和表情的高度一致。https://github.com/Jacksgong/JKeyboardPanelSwitch/tree/master/library/src/main/java/cn/dreamtobe/kpswitch

但这个库只有一个sample, 没有对这个库的实现进行文档说明，最后填了几天的坑，最终实现了效果图。

**类图关系**

本文主要讲解非全屏模式下的键盘





![类图](https://raw.githubusercontent.com/southmoney/markdownres/master/class.png)







![https://raw.githubusercontent.com/southmoney/markdownres/master/sequence.png](https://raw.githubusercontent.com/southmoney/markdownres/master/sequence.png)

