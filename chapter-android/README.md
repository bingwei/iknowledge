# Android

## Record & replay 相关

adb shell getevent 返回的内容其实是 Linux 内核的协议。

可以直接在底层完成 record & replay，方法是将内容写到设备 `/dev/input/eventx`（也就是 getevent 的时候输出的那个设备）。可以参考 [GitHub - android-touch-record-replay](https://github.com/Cartucho/android-touch-record-replay)。

`/dev/input/` 下有很多 event 设备：event0, event1, ... 可以在 `/proc/bus/input/devices` 文件中看到 eventx 对应的设备信息。也可以使用 `xinput` 命令查看。

### Monkey

`monkey --port`: 连接服务器进行Monkey操作，服务器上通过tcp或者adb生成事件，具体说明参考Monkey源代码中的README.NETWORK.txt文件