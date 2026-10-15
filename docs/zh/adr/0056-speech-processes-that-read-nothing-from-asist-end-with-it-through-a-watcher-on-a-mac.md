# 不从 ASIST 读取任何东西的语音进程，在 Mac 上用监视 shell 让它随 ASIST 一起结束

ASIST 启动的 VOICEVOX 和 AivisSpeech 引擎不从 ASIST 读取任何东西，所以 ASIST 没有走退出流程就结束时，在 Mac 上它们的父进程会变成 launchd，继续运行下去，一直占着模型的内存。除了强制结束和崩溃之外，开发中停掉启动的应用时也会发生。于是在 Mac 上，把它们放在自己的进程组里启动，并配上和 Agent 一样的监视 shell。监视 shell 读取只有 ASIST 写入的管道；进程自己结束时，ASIST 写一行，监视 shell 也随之结束；一行都没来管道就关闭时，就向那个进程组发送 SIGTERM。无法启动监视 shell 时，就不启动那个进程，以免造出 ASIST 挂掉后会留下的进程。从标准输入读取请求的 speech.cpp worker、Python worker、麦克风辅助程序在 ASIST 结束后管道关闭就会自己结束，所以不配监视 shell。语音识别的 llama-server 起初也按这个规定启动，但在 ADR 0059 里取消了随应用打包，语音识别改为在 speech.cpp 的 worker 里运行。Windows 上什么也不做。因为 libuv 会把没有加 `detached` 启动的子进程放进随 ASIST 一起关闭的 Job Object 里。

## 未采纳的方案

- **下次启动时找出上次留下的进程并停掉。** 下次启动之前，留下的进程会一直占用内存。
- **用 shell 包住引擎，shell 读到标准输入结束就停掉引擎。** ASIST 持有的是 shell 的进程，要得知引擎挂掉、要在停止时把信号送到引擎，都还需要在 shell 上再加一层机制。
- **即使启动不了监视 shell，也照样启动进程。** 等于明知 ASIST 挂掉后会留下进程，还去造它。和 Agent 一样，不启动。

## 实测

- 2026-10-05，在 Apple M5 的 macOS 26.2 上，用 `kill -9` 停掉接入麦克风并加载了全部模型的 ASIST 主体后，10 秒后只剩 llama-server。speech.cpp 的 worker、3 个 Python worker、麦克风辅助程序都已经结束。在另一个 worktree 里开发中启动的 llama-server，父进程一直是 launchd，运行了 2 天 14 小时。
- 同一天，同一台 Mac 上，用 `kill -9` 停掉像 ASIST 那样启动 VOICEVOX 和 AivisSpeech 引擎的 Node 进程后，两个引擎在 12 秒后父进程都仍是 launchd，继续运行。按这个规定启动时，两者都随父进程一起结束。
- 同一天，在 RTX 2080 的 Windows 11 上，用 `Stop-Process -Force` 停掉加载了模型的 ASIST 主体后，10 秒后包括 llama-server 在内的子进程全部结束，VRAM 回到 1682 MB。

## 已知限制

- 监视 shell 发送的只有 SIGTERM。SIGTERM 停不掉的进程会留下。
- 这些进程在自己的进程组里运行，所以开发中在终端里按 Ctrl-C，SIGINT 也送不到，要等 Electron 结束后由监视 shell 停掉。
- 这个规定之前的构建留下的进程不会停。
