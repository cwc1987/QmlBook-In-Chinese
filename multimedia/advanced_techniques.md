# 高级用法（Advanced Techniques）

## 10.5.1 实现一个播放列表（Implementing a Playlist）

Qt 5 multimedia接口没有提供播放列表。幸好，它非常容易实现。通过设置模型子项与MediaPlayer元素可以实现它，如下所示。当playstate通过player控制时，Playlist元素负责设置MediaPlayer的source。

```
    Playlist {
        id: playlist

        mediaPlayer: player

        items: ListModel {
            ListElement { source: "trailer_400p.ogg" }
            ListElement { source: "trailer_400p.ogg" }
            ListElement { source: "trailer_400p.ogg" }
        }
    }

    MediaPlayer {
        id: player
    }
```

Playlist元素的第一部分如下，注意使用setIndex函数来设置source元素的索引值。我们也实现了next与previous函数来操作链表。

```
Item {
    id: root

    property int index: 0
    property MediaPlayer mediaPlayer
    property ListModel items: ListModel {}

    function setIndex(i)
    {
        console.log("setting index to: " + i);

        index = i;

        if (index < 0 || index >= items.count)
        {
            index = -1;
            mediaPlayer.source = "";
        }
        else
            mediaPlayer.source = items.get(index).source;
    }

    function next()
    {
        setIndex(index + 1);
    }

    function previous()
    {
        setIndex(index + 1);
    }
```

让播放列表自动播放下一个元素的诀窍是使用MediaPlayer的status属性。当得到MediaPlayer.EndOfMedia状态时，索引值增加，恢复播放，或者当列表达到最后时，停止播放。

```
    Connections {
        target: root.mediaPlayer

        onStopped: {
            if (root.mediaPlayer.status == MediaPlayer.EndOfMedia)
            {
                root.next();
                if (root.index == -1)
                    root.mediaPlayer.stop();
                else
                    root.mediaPlayer.play();
            }
        }
    }
```
