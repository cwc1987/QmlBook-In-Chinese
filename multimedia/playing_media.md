# 媒体播放（Playing Media）

在QML应用程序中，最基本的媒体应用是播放媒体。使用MediaPlayer元素可以完成它，如果源是一个图片或者视频，可以选择结合VideoOutput元素。MediaPlayer元素有一个source属性指向需要播放的媒体。当媒体源被绑定后，简单的调用play函数就可以开始播放。

如果你想播放一个可视化的媒体，例如图片或者视频等，你需要配置一个VideoOutput元素。MediaPlayer播放通过source属性与视频输出绑定。

在下面的例子中，给MediaPlayer元素一个视频文件作为source。一个VideoOutput被创建和绑定到媒体播放器上。一旦主要部件完全初始化，例如在Component.onCompleted中，播放器的play函数被调用。

```
import QtQuick 2.0
import QtMultimedia 5.0
import QtSystemInfo 5.0

Item {
    width: 1024
    height: 600

    MediaPlayer {
        id: player
        source: "trailer_400p.ogg"
    }

    VideoOutput {
        anchors.fill: parent
        source: player
    }

    Component.onCompleted: {
        player.play();
    }

    ScreenSaver {
        screenSaverEnabled: false;
    }
}
// M1>>
```

除了上面介绍的视频播放，这个例子也包括了一小段代码用于禁止屏幕保护。这将阻止视频被中断。通过设置ScreenSaver元素的screenSaverEnabled属性为false来完成。通过导入QtSystemInfo 5.0可以使用ScreenSaver元素。

基础操作例如当播放媒体时可以通过MediaPlayer元素的volume属性来控制音量。还有一些其它有用的属性。例如，duration与position属性可以用来创建一个进度条。如果seekable属性为true，当拨动进度条时可以更新position属性。下面这个例子展示了在上面的例子基础上如何添加基础播放。

```
    Rectangle {
        id: progressBar

        anchors.left: parent.left
        anchors.right: parent.right
        anchors.bottom: parent.bottom
        anchors.margins: 100

        height: 30

        color: "lightGray"

        Rectangle {
            anchors.left: parent.left
            anchors.top: parent.top
            anchors.bottom: parent.bottom

            width: player.duration>0?parent.width*player.position/player.duration:0

            color: "darkGray"
        }

        MouseArea {
            anchors.fill: parent

            onClicked: {
                if (player.seekable)
                    player.position = player.duration * mouse.x/width;
            }
        }
    }
```

默认情况下position属性每秒更新一次。这意味着进度条将只会在大跨度下的时间周期下才会更新，需要媒体持续时间足够长，进度条像素足够宽。然而，这个可以通过mediaObject属性的notifyInterval属性改变。它可以设置每个position之间更新的毫秒数，增加用户界面的平滑度。

```
    Connections {
        target: player
        onMediaObjectChanged: {
            if (player.mediaObject)
                player.mediaObject.notifyInterval = 50;
        }
    }
```

当使用MediaPlayer创建一个媒体播放器时，最好使用status属性来监听播放器。这个属性是一个枚举，它枚举了播放器可能出现的状态，从MediaPlayer.Buffered到MediaPlayer.InvalidMedia。下面是这些状态值的总结：

* MediaPlayer.UnknownStatus - 未知状态

* MediaPlayer.NoMedia - 播放器没有指定媒体资源，播放停止

* MediaPlayer.Loading - 播放器正在加载媒体

* MediaPlayer.Loaded - 媒体已经加载完毕，播放停止

* MediaPlayer.Stalled - 加载媒体已经停止

* MediaPlayer.Buffering - 媒体正在缓冲

* MediaPlayer.Buffered - 媒体缓冲完成

* MediaPlayer.EndOfMedia - 媒体播放完毕，播放停止

* MediaPlayer.InvalidMedia - 无法播放媒体，播放停止

正如上面提到的这些枚举项，播放状态会随着时间变化。调用play，pause或者stop将会切换状态，但由于媒体的原因也会影响这些状态。例如，媒体播放完毕，它将会无效，导致播放停止。当前的播放状态可以使用playbackState属性跟踪。这个值可能是MediaPlayer.PlayingState，MediaPlayer.PasuedState或者MediaPlayer.StoppedState。

使用autoPlay属性，MediaPlayer在source属性改变时将会尝试进入播放状态。类似的属性autoLoad将会导致播放器在source属性改变时尝试加载媒体。默认下autoLoad是被允许的。

当然也可以让MediaPlayer循环播放一个媒体项。loops属性控制source将会被重复播放多少次。设置属性为MediaPlayer.Infinite将会导致不停的重播。非常适合持续的动画或者一个重复的背景音乐。
