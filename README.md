# ws scrcpy

[Genymobile/scrcpy][scrcpy] 的Web客户端及其他功能。

## 要求

浏览器必须支持以下技术：
* WebSockets
* Media Source Extensions 和 h264 解码；
* WebWorkers
* WebAssembly

服务器：
* Node.js v10+
* node-gyp ([安装](https://github.com/nodejs/node-gyp#installation))
* `adb` 可执行文件必须在 PATH 环境变量中可用

设备：
* Android 5.0+ (API 21+)
* 启用 [adb 调试](https://developer.android.com/studio/command-line/adb.html#Enabling)
* 在某些设备上，还需要启用
  [一个额外选项](https://github.com/Genymobile/scrcpy/issues/70#issuecomment-373286323)
  以使用键盘和鼠标控制。

## 构建和启动

确保已安装 [node.js](https://nodejs.org/en/download/),
[node-gyp](https://github.com/nodejs/node-gyp) 和
[构建工具](https://github.com/nodejs/node-gyp#installation)
```shell
git clone https://github.com/NetrisTV/ws-scrcpy.git
cd ws-scrcpy

## 为了稳定版本，查找最新的标签并切换到它：
# git tag -l
# git checkout vX.Y.Z

npm install
npm start
```

## 支持的功能

### Android

#### 屏幕投影
修改的 [版本][fork] 用于流式传输 H264 视频，然后由以下其中一个解码器解码：

##### Mse Player

基于 [xevokk/h264-converter][xevokk/h264-converter]。
HTML5 视频。<br>
需要 [Media Source API][MSE] 和 `video/mp4; codecs="avc1.42E01E"`
[支持][isTypeSupported]。从设备接收到的 NALU 创建 mp4 容器，然后将它们传递给 [MediaSource][MediaSource]。理论上可以使用硬件加速。

##### Broadway Player

基于 [mbebenita/Broadway][broadway] 和
[131/h264-live-player][h264-live-player]。<br>
软件视频解码器编译为 wasm 模块。
需要 [WebAssembly][wasm] 和最好有 [WebGL][webgl] 支持。

##### TinyH264 Player

基于 [udevbe/tinyh264][tinyh264]。<br>
软件视频解码器编译为 wasm 模块。是 [mbebenita/Broadway][broadway] 的略微更新版本。
需要 [WebAssembly][wasm]、[WebWorkers][workers]、[WebGL][webgl] 支持。

##### WebCodecs Player

通过浏览器内置（软件/硬件）媒体解码器解码。
需要 [WebCodecs][webcodecs] 支持。目前仅在 [Chromium](https://www.chromestatus.com/feature/5669293909868544) 及其衍生品中可用。

#### 远程控制
* 触摸事件（包括多点触控）
* 多点触控模拟：按下 <kbd>CTRL</kbd> 以从屏幕中心开始，按下 <kbd>SHIFT</kbd> + <kbd>CTRL</kbd> 以从当前点开始
* 鼠标滚轮和触控板垂直/水平滚动
* 捕捉键盘事件
* 注入文本（仅 ASCII）
* 复制到/从设备剪贴板
* 设备“旋转”

#### 文件推送
拖放 APK 文件将其推送到 `/data/local/tmp` 目录。您可以从包含的 [xtermjs/xterm.js][xterm.js] 终端模拟器手动安装它（见下文）。

#### 远程 shell
从浏览器中的 `adb shell` 控制您的设备。

#### 调试网页/WebView
[/docs/Devtools.md](/docs/Devtools.md)

#### 文件列表
* 列出文件
* 通过拖放上传文件
* 下载文件

### iOS

***实验性功能***：*默认未构建*
（见[自定义构建](#custom-build)）

#### 屏幕投影

需要 [ws-qvh][ws-qvh] 在 `PATH` 中可用。

#### MJPEG 服务器

在构建配置文件中启用 `USE_WDA_MJPEG_SERVER`
（见[自定义构建](#custom-build)）。

另一种传输屏幕内容的方式。它不需要额外的软件，如 `ws-qvh`，但可能需要更多资源，因为每帧都编码为 jpeg 图像。

#### 远程控制

为了控制设备，我们使用 [appium/WebDriverAgent][WebDriverAgent]。
功能仅限于：
* 简单触摸
* 滚动
* 主屏幕按钮点击

确保您已正确设置 [WebDriverAgent](https://appium.io/docs/en/drivers/ios-xcuitest-real-devices/)。
WebDriverAgent 项目位于 `node_modules/appium-webdriveragent/` 下。

您可能需要在设备上启用 `AssistiveTouch`：`设置/通用/辅助功能`。

## 自定义构建

您可以通过覆盖 [默认配置](/webpack/default.build.config.json) 中的
[build.config.override.json](/build.config.override.json) 自定义项目：
* `INCLUDE_APPL` - 包含 iOS 设备跟踪和控制代码
* `INCLUDE_GOOG` - 包含 Android 设备跟踪和控制代码
* `INCLUDE_ADB_SHELL` - [远程 shell](#remote-shell) 用于 Android 设备
  ([xtermjs/xterm.js][xterm.js], [Tyriar/node-pty][node-pty])
* `INCLUDE_DEV_TOOLS` - [开发工具](#debug-webpageswebview) 用于调试 Android 设备上的网页和 WebView
* `INCLUDE_FILE_LISTING` - 最小化 [文件管理](#file-listing)
* `USE_BROADWAY` - 包括 [Broadway Player](#broadway-player)
* `USE_H264_CONVERTER` - 包括 [Mse Player](#mse-player)
* `USE_TINY_H264` - 包括 [TinyH264 Player](#tinyh264-player)
* `USE_WEBCODECS` - 包括 [WebCodecs Player](#webcodecs-player)
* `USE_WDA_MJPEG_SERVER` - 配置 WebDriverAgent 启动 MJPEG 服务器
* `USE_QVH_SERVER` - 包括 [ws-qvh][ws-qvh] 支持
* `SCRCPY_LISTENS_ON_ALL_INTERFACES` - `scrcpy-server.jar` 中的 WebSocket 服务器将监听所有可用接口上的连接。当 `true` 时，允许直接从浏览器连接设备。否则，必须通过 adb 建立连接。

## 运行配置

您可以在 `WS_SCRCPY_CONFIG` 环境变量中指定配置文件的路径。

如果您想要其他的路径名而不是“/”，可以在
`WS_SCRCPY_PATHNAME` 环境变量中指定。

配置文件格式： [Configuration.d.ts](/src/types/Configuration.d.ts)。

配置文件示例： [config.example.yaml](/config.example.yaml)。

## 已知问题

* Android 模拟器上的服务器在内部接口上监听，不从外部可用。从接口列表中选择 `通过 adb 代理`。
* TinyH264Player 可能无法启动，尝试重新加载页面。
* MsePlayer 在质量统计中报告丢帧过多：需要进一步调查。
* 在 Safari 上文件上传不显示进度（一次性传输）。

## 安全警告
请注意并牢记：
* 浏览器和 node.js 服务器之间没有加密（您可以 [配置](#run-configuration) HTTPS）。
* 浏览器和 Android 设备上的 WebSocket 服务器之间没有加密。
* 在任何级别上都没有授权。
* 带有集成 WebSocket 服务器的修改版 scrcpy 正在监听所有网络接口上的连接（见[自定义构建](#custom-build)）。
* 在最后一个客户端断开连接后，修改版 scrcpy 将继续运行。

## 相关项目
* [Genymobile/scrcpy][scrcpy]
* [xevokk/h264-converter][xevokk/h264-converter]
* [131/h264-live-player][h264-live-player]
* [mbebenita/Broadway][broadway]
* [DeviceFarmer/adbkit][adbkit]
* [xtermjs/xterm.js][xterm.js]
* [udevbe/tinyh264][tinyh264]
* [danielpaulus/quicktime_video_hack][qvh]

## scrcpy websocket 分支

目前，支持 WebSocket 协议的 scrcpy 已添加到 v1.19
* [预构建包](/vendor/Genymobile/scrcpy/scrcpy-server.jar)
* [源代码][fork]

[fork]: https://github.com/NetrisTV/scrcpy/tree/feature/websocket-v1.19.x

[scrcpy]: https://github.com/Genymobile/scrcpy
[xevokk/h264-converter]: https://github.com/xev
