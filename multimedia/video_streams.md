# 视频流（Video Streams）

VideoOutput元素不被限制与MediaPlayer元素绑定使用的。它也可以直接用来加载实时视频资源显示一个流媒体。应用程序使用Camera元素作为资源。来自Camera的视频流给用户提供了一个实时流媒体。

```
import QtQuick 2.0
import QtMultimedia 5.0

Item {
    width: 1024
    height: 600

    VideoOutput {
        anchors.fill: parent
        source: camera
    }

    Camera {
        id: camera
    }
}
```
