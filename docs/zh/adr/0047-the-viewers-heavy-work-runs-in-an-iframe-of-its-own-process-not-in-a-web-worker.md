# 文件查看器的繁重处理不在 Web Worker 里，而在另一个进程里运行的 iframe 里完成

文件卡片和放大显示的查看器所做的繁重处理（解析文档、解码图片、绘制页面），在一个藏在应用界面里的 iframe 里完成。main 给这个 iframe 加载一个用 `asist-preview://` 这个专用 scheme 分发的页面。这个页面和应用界面属于不同站点，所以 Chromium 让这个 iframe 在另一个进程里运行。即使文件过大导致内存被用尽，终止的也只是这个 iframe 的进程，应用界面和对话会继续。iframe 崩溃这件事，会以等待回应的请求失败的形式传到查看器，查看器把它当作错误显示。下一次请求会启动一个新的 iframe。用 Web Worker，界面线程也不会停，但 Worker 和界面跑在同一个 renderer 进程里，所以 Worker 耗尽内存时会连 renderer 一起崩溃，界面被重新加载，对话中断。

## 未采纳的方案

- **用 Web Worker 处理。** 解析期间界面也不会停，但 Worker 耗尽内存时，会连同界面的 renderer 进程一起崩溃。

## 实测

2026-10-02，Electron 43.7.7，Apple M5（内存 32 GB）。在 iframe 里不停分配数组，让它把 V8 堆的上限（约 4 GB）用尽。

- 开发版（`electron-vite dev`）：iframe 在和界面不同的另一个 renderer 进程里运行，什么都没打开时的内存（RSS）是 79～84 MB。耗尽之后过了 2.0～2.4 秒，等待中的请求以失败的形式传到界面。这期间，界面上每 10 ms 动一次的定时器，延迟最大 3 ms，界面没有重新加载。下一次请求启动了新进程的 iframe，54～97 ms 后回应到达。在同一个界面上让 Web Worker 同样耗尽内存时，界面的 renderer 进程崩溃（`render-process-gone`，reason 是 crashed），界面被重新加载。
- 发布版（`npm run dist:mac:unsigned`，2 次）：iframe 在另一个 renderer 进程里运行，什么都没打开时的内存是 78～80 MB。从启动 iframe 到页面回应到达是 46～89 ms。耗尽之后过了 0.6～0.7 秒，iframe 崩溃这件事传到了界面，定时器的延迟最大 1 ms，界面没有重新加载。紧接着启动的新 iframe 在 58 ms 内回应。其他处理繁忙时（load average 约 30）用了 1,064 ms。

## 已知限制

- 任何查看器都没有打开文档时，就把 iframe 去掉。打开期间，它作为一个进程额外占用约 80 MB，去掉之后再打开时，启动 iframe 要花 50～100 ms 左右。
- 这个页面能像应用界面一样用 `asist-file://` 读取获准文件夹里的文件。因此 main 把能嵌入这个页面的对象限定为应用界面（`frame-ancestors`），页面只接受来自嵌入它的界面的请求。页面和页面启动的 Worker 只能运行自己的脚本和读取 `asist-file://`，无法向外部主机发送任何东西，也打不开新窗口。
- iframe 崩溃这件事，是通过 Electron 独有的 MessagePort close 事件得知的。Chrome（154）没有这个事件，所以在 Chrome 里打开的 demo 中，即使 iframe 崩溃，等待中的请求也会继续等下去。
