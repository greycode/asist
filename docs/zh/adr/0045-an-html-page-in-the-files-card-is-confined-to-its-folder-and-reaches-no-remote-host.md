# 文件卡片里打开的 HTML 页面只读取该文件夹内部，并在与网络隔开的会话中显示

在文件卡片里打开的 HTML 页面，以及浏览器在其中运行脚本的其他文档（SVG 等），在通过 `asist-file` 分发时都附带一份 CSP（`documentPolicy`），只允许读取该文件所在文件夹及其下层。可读取的只有自己文件夹里的脚本、样式、图片、字体、媒体，以及文档自身持有的 `data:` 和 `blob:`，其他文件夹的文件和外部服务器的资源都读不到。`connect-src 'none'` 封住 `fetch`、`sendBeacon`、WebSocket，也封住表单提交、frame 和 worker 对外的通信。CSP 的 source 需要有主机，但本地路径的 `asist-file` URL 没有主机，所以在空主机的位置放一个 `*`，靠路径把范围限制在文件夹里。Windows 共享时，则直接把服务器名当作主机。

在此基础上，页面不再用与应用界面同一会话的 iframe，而是用与网络隔开的专用会话中的 `<webview>` 显示。原因是有两种通信 CSP 覆盖不到。一种是通过 `<link rel="dns-prefetch">` 向 OS 的名称解析查询页面选定的主机名。另一种是 WebRTC，它与 CSP 无关，可以向页面指定的任何地址用 UDP 和 TCP 发送。在 main 里，应用界面的所有 `<webview>` 都接到这个会话上，并在页面的渲染进程里关掉 DNS 预读（`--blink-settings=dnsPrefetchingEnabled=false`）。会话取消 `asist-file` 以外的所有请求，把不是请求的连接（WebRTC 的 TCP）导向一个没有连接目标的代理（`127.0.0.1` 的端口 0，OS 在发出数据包之前就拒绝）。同时还要去掉 Chromium 只向这台 Mac 直接连接的例外。WebRTC 的 UDP 也一并停掉。页面自己发起的跳转、新窗口、权限请求也全部拒绝。CSP 的 `sandbox allow-scripts` 让页面的来源变得不确定，并阻止弹出窗口、表单和对话框。webview 是没有 preload 的另一个 webContents，所以页面既无法接触应用界面，也无法接触 preload 的桥。应用界面的 `frame-src` 里不放 `asist-file:`，文件文档不会在应用会话里显示。

## 未采纳的方案

- **照旧允许从 CDN 读取脚本和样式（在 `script-src`/`img-src` 等处写 `https:`、`connect-src https:`）。** 当初的许可是为了让报告从 CDN 读取图表的部件，但通向外部 https 的口子原样留着，就成了把读到的内容发往外部的口子。“不能连到外部”和“可以读取任意的 https”无法并存，所以不再读取外部，文档只在自己的文件夹内完成。
- **对每个子资源请求，在 main 侧也拒绝页面文件夹之外的东西。** `asist-file` 的请求上不携带是哪个页面发出的信息（不带 Referer，来源也不确定），所以没有办法按请求判断页面属于哪个文件夹，要加入这一层就得重做这个 scheme。把读得到的内容送往外部的口子既然已经全部关闭，那么在下面限制的范围内，即使页面能读到另一个获准文件夹的内容，也带不出去。于是不在 main 侧加额外的检查，只在下面写清可读范围。
- **保持 iframe，在整个应用里关掉 DNS 预读。** dns-prefetch 会停，但 WebRTC 不会停。要用代理阻止 WebRTC 的 TCP，只能改变整个会话的代理，连应用界面的通信也会停。
- **把整个应用的名称解析指向一个不存在的 DNS over HTTPS 服务器（`app.configureHostResolver`）。** 应用其他通信的名称解析也会全部停掉，dns-prefetch 还会尝试连接那个 DNS over HTTPS 服务器。
- **用 `WebContentsView` 把另一个 webContents 叠在窗口上。** 因为它是画在窗口固定位置上的一层面，无法配合卡片的滚动、下端裁切以及与焦点显示的重叠。`<webview>` 则和 iframe 一样排在界面当中。

## 实测

2026-10-02，Electron 43.7.7，Apple Silicon 的 Mac。在一个页面里逐个放置的元素全部指向 `prefetch-check.invalid`，每次加载都用新的 `--user-data-dir` 启动 Electron。名称的查询看 net log 里 HOST_RESOLVER 的记录和所有进程的 `getaddrinfo` 调用，连接则看 net log 的套接字记录，以及监听在这台 Mac 这一 WebRTC 目的地上的端口。表中的记法是“查询 / 连接”，CSP 和跳转的拒绝对表中的每一行都生效。

| 设置 | dns-prefetch(`https://`) | dns-prefetch（用脚本添加） | preconnect | img | 点击链接 | WebRTC（以名称给出 STUN/TURN） | WebRTC（以名称给出对端候选） | WebRTC（连到这台 Mac 的地址） |
|---|---|---|---|---|---|---|---|---|
| 应用会话的 iframe（以前的形式） | 有 / 无 | 有 / 无 | 无 / 无 | 无 / 无 | 无 / 无 | 有 / 有 | 有 / 无 | 无 / 有 |
| 只有专用会话的 webview | 有 / 无 | 有 / 无 | 无 / 无 | 无 / 无 | 无 / 无 | 有 / 有 | 有 / 无 | 无 / 有 |
| + 没有连接目标的代理 | 有 / 无 | 有 / 无 | 无 / 无 | 无 / 无 | 无 / 无 | 有 / 无 | 有 / 无 | 无 / 有 |
| + 取消 `asist-file` 以外的请求 | 有 / 无 | 有 / 无 | 无 / 无 | 无 / 无 | 无 / 无 | 有 / 有 | 有 / 无 | 无 / 有 |
| + preconnect 事件的 `preventDefault` | 有 / 无 | 有 / 无 | 无 / 无 | 无 / 无 | 无 / 无 | 有 / 有 | 有 / 无 | 无 / 有 |
| + 网络离线模拟 | 有 / 无 | 有 / 无 | 无 / 无 | 无 / 无 | 无 / 无 | 有 / 有 | 有 / 无 | 无 / 有 |
| + 停止 WebRTC 的 UDP | 有 / 无 | 有 / 无 | 无 / 无 | 无 / 无 | 无 / 无 | 有 / 有 | 有 / 无 | 无 / 有 |
| 上面的会话设置全部加上 | 有 / 无 | 有 / 无 | 无 / 无 | 无 / 无 | 无 / 无 | 有 / 无 | 有 / 无 | 无 / 无 |
| 采用的形式（代理、取消请求、停止 WebRTC 的 UDP、停止渲染进程的 DNS 预读、去掉连到这台 Mac 的例外） | 无 / 无 | 无 / 无 | 无 / 无 | 无 / 无 | 无 / 无 | 有 / 无 | 有 / 无 | 无 / 无 |

WebRTC 的名称查询不会留在 net log 里，而是作为网络服务进程的 `getaddrinfo` 记录下来。对 `.local` 的名字不会调用 `getaddrinfo`，而是从这台 Mac 发出了多播 DNS 的查询（224.0.0.251:5353）。`href="//prefetch-check.invalid"` 的 dns-prefetch 会作为相对 URL 解析到 `asist-file`，所以在任何形式下都没有发起查询。连到这台 Mac 自身（`127.0.0.1`）的 WebRTC TCP，在去掉 Chromium 的例外之前，即使有代理也照样能连上。在采用的形式下，跳转到页面自身、meta refresh、`window.open`、speculation rules、`ping`、WebSocket、`fetch` 与 `sendBeacon`、favicon 与 manifest、`rel="prefetch"`，以及来自 about:blank 的 iframe 的 `fetch`，都既没有查询，也没有连接。即使去掉 CSP 和跳转拒绝，只靠采用形式的会话，这些同样既没有查询，也没有连接。开发版应用和使用 `npm run dist:mac:unsigned` 制作的发布版应用，结果也与采用形式的那一行相同。

到显示为止的时间（从加入元素到页面的脚本运行，5 次）在 iframe 下是 52～82 ms，在 webview 下是 71～94 ms。在应用卡片里，从打开卡片到页面的脚本运行需要 115～250 ms（4 次）。内存在这一点上两者相同：都为页面多出一个进程，整体增加约 134 MB。在应用卡片里，进程多出一个，内存增加 101～126 MB（4 次）。专用会话本身约 1 MB。

## 已知限制

- 页面读不到外部资源，所以依赖 CDN 库的报告，必须把部件一并放进自己的文件夹，或者内嵌到页面里。
- 限制范围靠 CSP 的路径实现。同一文件夹里的文件之间，照旧可以互相读取内容。页面文件夹里指向另一个获准文件夹的符号链接（例如克隆下来的仓库、解压后的 zip 内部就有的那种），顺着它走到的位置可以读取；把页面直接放在较宽的文件夹（例如 Downloads）下时，该文件夹整体也可以读取。只要 URL 落在那个页面文件夹之下，就能读取。这两种情况都只是能读，送不到外部。
- 页面写下的 ICE 服务器或对端候选的主机名会进入网络服务的名称解析，代理和取消请求都管不到。`.local` 的名字会变成对本地网络的多播 DNS 查询，其他名字则交给系统的名称解析。与那些主机的连接不会发生。2026-10-02 逐一试过 CSP 的 `webrtc 'block'`（Chromium 150 把它当作不认识的指令忽略）、禁用全部 18 个与 WebRTC 相关的 Blink 运行时特性（`RTCPeerConnection` 仍然保留）、WebRTC 的 IP 处理的其他取值、UDP 的端口范围，哪一种都没能阻止它。WebRTC 也不经过权限的确认。
