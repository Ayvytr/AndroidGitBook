# ANR
(Application Not responding)，是指应用程序未响应，Android系统对于一些事件需要在一定的时间范围内完成，如果超过预定时间能未能得到有效响应或者响应时间过长，都会造成ANR。一般地，这时往往会弹出一个提示框，告知用户当前xxx未响应，用户可选择继续等待或者Force Close。    

那么哪些场景会造成ANR呢？
* 前台服务在20s内未执行完成
* 前台广播10s内未执行完成
* 输入事件分发超过5s
* 内容提供者publish过程超过10s

## 原理
发生ANR时会调用AppNotRespondingDialog.show()方法弹出对话框提示用户
