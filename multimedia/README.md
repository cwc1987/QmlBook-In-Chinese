# Multimedia

在QtMultimedia模块中的multimedia元素可以播放和记录媒体资源，例如声音，视频，或者图片。解码和编码的操作由特定的后台完成。例如在Linux上的gstreamer框架，Windows上的DirectShow，和OS X上的QuickTime。
multimedia元素不是QtQuick核心的接口。它的接口通过导入QtMultimedia 5.0来加入，如下所示：

```
import QtMultimedia 5.0
```
