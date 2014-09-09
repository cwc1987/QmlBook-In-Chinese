# 捕捉图像（Capturing Images）

Camera元素一个关键特性就是可以用来拍照。我们将在一个简单的定格动画程序中使用到它。在这章中，你将学习如何显示一个视图查找器，截图和追踪拍摄的图片。

用户界面如下所示。它由三部分组成，背景是一个视图查找器，右边有一列按钮，底部有一连串拍摄的图片。我们想要拍摄一系列的图片，然后点击Play Sequence按钮。这将回放图片，并创建一个简单的定格电影。

![](http://qmlbook.org/_images/camera-ui.png)

相机的视图查找器部分是在VideoOutput中使用一个简单的Camera元素作为资源。这将给用户显示一个来自相机的流媒体视频。

```
    VideoOutput {
        anchors.fill: parent
        source: camera
    }

    Camera {
        id: camera
    }
```

使用一个水平放置的ListView显示来自ListModel的图片，这个部件叫做imagePaths。在背景中使用一个半透明的Rectangle。

```
    ListModel {
        id: imagePaths
    }

    ListView {
        id: listView

        anchors.left: parent.left
        anchors.right: parent.right
        anchors.bottom: parent.bottom
        anchors.bottomMargin: 10

        height: 100

        orientation: ListView.Horizontal
        spacing: 10

        model: imagePaths

        delegate: Image { source: path; fillMode: Image.PreserveAspectFit; height: 100; }

        Rectangle {
            anchors.fill: parent
            anchors.topMargin: -10

            color: "black"
            opacity: 0.5
        }
    }
```

为了拍摄图像，你需要知道Camera元素包含了一组子对象用来完成各种工作。使用Camera.imageCapture用来捕捉图像。当你调用capture方法时，一张图片就被拍摄下来了。Camera.imageCapture的结果将会发送imageCaptured信号，接着发送imageSaved信号。

```
        Button {
            id: shotButton

            width: 200
            height: 75

            text: "Take Photo"
            onClicked: {
                camera.imageCapture.capture();
            }
        }
```

为了拦截子元素的信号，需要一个Connections元素。在这个例子中，我们不需要显示预览图片，仅仅只是将结果图片加入底部的ListView中。就如下面的例子展示的一样，图片保存的路径由信号的path参数提供。

```
    Connections {
        target: camera.imageCapture

        onImageSaved: {
            imagePaths.append({"path": path})
            listView.positionViewAtEnd();
        }
    }
```

为了显示预览，连接imageCaptured信号，并且使用preview信号参数作为Image元素的source。requestId信号参数与imageCaptured和imageSaved一起发送。这个值由capture方法返回。这样，就可以完整的跟踪拍摄的图片了。预览的图片首先被使用，然后替换为保存的图片。然而在这个例子中我们不需要这样做。

最后是自动回放的部分。使用Timer元素来驱动它，并且加上一些JavaScript。_imageIndex变量被用来跟踪当前显示的图片。当最后一张图片被显示时，回放停止。在例子中，当播放序列时，root.state被用来隐藏用户界面。

```
    property int _imageIndex: -1

    function startPlayback()
    {
        root.state = "playing";
        setImageIndex(0);
        playTimer.start();
    }

    function setImageIndex(i)
    {
        _imageIndex = i;

        if (_imageIndex >= 0 && _imageIndex < imagePaths.count)
            image.source = imagePaths.get(_imageIndex).path;
        else
            image.source = "";
    }

    Timer {
        id: playTimer

        interval: 200
        repeat: false

        onTriggered: {
            if (_imageIndex + 1 < imagePaths.count)
            {
                setImageIndex(_imageIndex + 1);
                playTimer.start();
            }
            else
            {
                setImageIndex(-1);
                root.state = "";
            }
        }
    }
```
