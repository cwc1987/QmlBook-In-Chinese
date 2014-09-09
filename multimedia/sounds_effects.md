# 声音效果（Sounds Effects）

当播放声音效果时，从请求播放到真实响应播放的响应时间非常重要。在这种情况下，SoundEffect元素将会派上用场。设置source属性，一个简单调用play函数会直接开始播放。

当敲击屏幕时，可以使用它来完成音效反馈，如下所示：

```
    SoundEffect {
        id: beep
        source: "beep.wav"
    }

    Rectangle {
        id: button

        anchors.centerIn: parent

        width: 200
        height: 100

        color: "red"

        MouseArea {
            anchors.fill: parent
            onClicked: beep.play()
        }
    }
```

这个元素也可以用来完成一个配有音效的转换。为了从转换触发，使用ScriptAction元素。

```
    SoundEffect {
        id: swosh
        source: "swosh.wav"
    }

    transitions: [
        Transition {
            ParallelAnimation {
                ScriptAction { script: swosh.play(); }
                PropertyAnimation { properties: "rotation"; duration: 200; }
            }
        }
    ]
```

除了调用play函数，在MediaPlayer中类似属性也可以使用。比如volume和loops。loops可以设置为SoundEffect.Infinite来提供无限重复播放。停止播放调用stop函数。

**注意**

**当后台使用PulseAudio时，stop将不会立即停止，但会阻止继续循环。这是由于底层API的限制造成的。**
