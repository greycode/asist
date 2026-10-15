# ASIST 文档（简体中文）

本目录是 [docs/](../) 下日文文档的简体中文译本，与原文一一对应，文件名相同；日文原文保留在原位置。改动一处时，另一处也要跟着改。

## 开发文档

- [开发](development.md) — 从源码启动、clone 仓库、CI、构建、发布与真机验证（对应 [docs/development.md](../development.md)）

## 决策记录

每一篇的决策、未采纳的方案、已知限制与实测都译成了中文。原文在同一文件名下的 [docs/adr/](../adr/)。

| 编号 | 决策 |
| --- | --- |
| 0001 | [语言设置分成界面语言、对话语言和地区 3 个](adr/0001-language-settings-are-three.md) |
| 0002 | [只有日语的对话经过专门打磨，其余 10 种语言用共同的做法运行](adr/0002-only-japanese-conversation-is-tuned.md) |
| 0003 | [每次对话都带上 me.md 和 user.md 并各自收在 token 上限内，不持有用来遗忘的机制](adr/0003-me-and-user-md-ride-in-every-prompt-within-a-token-limit-and-nothing-tracks-forgetting.md) |
| 0004 | [把笔记当作用户的文档，对话中的 Agent 只能新建和阅读](adr/0004-notes-are-never-changed-by-the-agent.md) |
| 0005 | [对话中的模型启动和继续的 Agent 任务，以及 worktree 的合入与丢弃，一律先在确认界面取得用户的批准再执行](adr/0005-every-model-started-agent-job-merge-and-discard-asks-the-user.md) |
| 0006 | [API 密钥加密保存，包括 Agent 的 CLI 在内，不传给任何子进程](adr/0006-api-keys-are-encrypted-and-never-passed-to-child-processes.md) |
| 0007 | [uv 与 git 随应用打包，Python 由随应用打包的 uv 在准备时获取](adr/0007-uv-and-git-ship-with-the-app-and-python-is-downloaded-by-uv.md) |
| 0008 | [保存的 JSON 文件带有格式版本，读取时迁移到当前版本](adr/0008-stored-files-carry-a-version-and-are-upgraded-on-read.md) |
| 0009 | [主题用一组 CSS 变量的值表示，只允许卡片标题改变大小](adr/0009-a-theme-swaps-css-variables-and-resizes-only-card-headings.md) |
| 0010 | [Agent 任务改动在汇总提交和合入时，不运行仓库的提交钩子](adr/0010-agent-job-commits-and-merges-do-not-run-the-repository-commit-hooks.md) |
| 0011 | [每轮变化的信息不放进系统提示词，只在变化时附在输入上](adr/0011-what-changes-each-turn-rides-on-the-input-not-the-system-prompt.md) |
| 0012 | [已经开始发送的邮件草稿，不会再发第二次](adr/0012-a-mail-draft-whose-send-started-is-never-sent-again.md) |
| 0013 | [对话轮次不等待历史摘要，上下文接近 API 上限时不回答，而是这样告知](adr/0013-a-turn-never-waits-for-the-history-summary.md) |
| 0014 | [碰到子模块的 Agent 任务，ASIST 不合入，保留 worktree 交给用户](adr/0014-agent-job-that-touched-a-submodule-is-not-merged-by-asist.md) |
| 0016 | [Agent 的任务把 worktree 按磁盘上实际的样子提交，index 设置让 git 看不见的文件也包括在内](adr/0016-agent-job-commits-the-worktree-as-it-is-on-disk-whatever-the-index-hides.md) |
| 0017 | [该 OS 上无法使用的功能，不出现在界面里，也不出现在模型的工具里](adr/0017-features-the-os-cannot-run-are-left-out-of-the-screens-and-the-tools.md) |
| 0018 | [ASIST 的 git 只沿用用户 git 设置中的换行符与符号链接处理，以及忽略文件清单](adr/0018-asist-git-takes-only-the-users-line-ending-symlink-and-ignore-settings.md) |
| 0019 | [Windows 上，Agent 的 CLI 用带 Job Object 的启动程序来运行](adr/0019-agents-on-windows-run-in-a-job-object-of-the-launcher.md) |
| 0020 | [Windows 的 codex 始终在 elevated 的 sandbox 中运行，准备没做好就不运行](adr/0020-codex-on-windows-runs-only-in-its-elevated-sandbox.md) |
| 0021 | [本地的语音识别与朗读用内置的 speech.cpp 在 GPU 上运行](adr/0021-local-speech-runs-on-the-gpu-with-bundled-speech-cpp.md) |
| 0022 | [Windows 应用不在 Microsoft Store 上分发，最初不带签名分发](adr/0022-the-windows-app-is-not-published-on-the-microsoft-store.md) |
| 0023 | [在 Windows 上，麦克风经由 Windows 的通话用回声消除采集，是否可用只在启动时决定一次](adr/0023-windows-captures-the-microphone-through-its-communications-echo-canceller-decided-at-startup.md) |
| 0024 | [Google 日历在使用时读取，既不用变更通知，也不在后台轮询](adr/0024-google-calendar-is-read-when-used-not-pushed-or-polled.md) |
| 0025 | [Google 的 OAuth 客户端从构建环境装入，而不是从仓库](adr/0025-the-google-oauth-client-comes-from-the-build-environment.md) |
| 0026 | [Mac 的应用用 Developer ID 签名并公证、从这台 Mac 放到 Release，Windows 的应用不签名、由 GitHub Actions 放到同一个 Release，两者的更新都在下次退出时装入](adr/0026-mac-releases-are-signed-with-developer-id-windows-releases-are-unsigned-and-updates-install-at-the-next-quit.md) |
| 0027 | [首次设置只准备 MaAI，是否打开交给用户](adr/0027-setup-prepares-maai-but-leaves-it-off.md) |
| 0028 | [无法启动 MaAI 或处理滞后时，用设置的静音长度继续对话，并把这件事只告知一次](adr/0028-maai-that-cannot-start-or-falls-behind-leaves-the-conversation-on-the-fixed-wait.md) |
| 0029 | [设置从概要打开，模型的准备在使用该模型的那一行做](adr/0029-settings-open-on-an-overview-and-prepare-models-where-they-are-used.md) |
| 0030 | [VAD 把只有一句“はい”的回应也送进语音识别，能量判定的固定下限不降低](adr/0030-the-vad-passes-a-lone-short-reply-and-keeps-its-fixed-lower-threshold.md) |
| 0031 | [日语对话默认用 Irodori-TTS 的 3 个声音朗读，其他语言用 Qwen3-TTS 朗读](adr/0031-japanese-is-read-by-irodori-tts-in-three-voices-and-other-languages-by-qwen3-tts.md) |
| 0032 | [live 引擎只采用连回复内容也自己决定的模型，不采用 GPT-Live](adr/0032-a-live-engine-decides-the-reply-itself-so-gpt-live-is-not-offered.md) |
| 0033 | [任务告知的文件位于 git 忽略的位置时，不询问用户、也不清理 worktree，交给用户处理](adr/0033-agent-job-keeps-its-worktree-when-a-reported-file-lies-where-git-ignores-it.md) |
| 0034 | [Agent 把 worktree 切到其他分支的任务，ASIST 不合入，而是告知该分支的名字，交给用户处理](adr/0034-agent-job-whose-agent-switched-the-worktree-to-another-branch-is-not-merged-by-asist.md) |
| 0035 | [地区设为日本时的天气，用罗马字写的地名由表这一侧来查](adr/0035-places-of-japan-named-in-romaji-are-resolved-by-the-weather-table.md) |
| 0036 | [在 Mac 上，按用户登录 shell 的 PATH 查找 Agent 的 CLI，并用同一个 PATH 启动](adr/0036-the-agent-cli-on-a-mac-is-found-and-run-on-the-path-of-the-users-login-shell.md) |
| 0037 | [在 Mac 上，确认 Agent 已经结束，要看 CLI 的进程组、CLI 启动的进程、持有 token 的进程是否一样都不剩](adr/0037-an-agent-on-a-mac-has-ended-only-when-none-of-its-processes-is-left.md) |
| 0038 | [衔接语单独设一个与附和不同的开关，任何对话语言下都显示](adr/0038-the-bridge-phrase-and-the-aizuchi-are-turned-on-and-off-by-separate-switches.md) |
| 0039 | [在 Mac 上，ASIST 退出时向正在运行的 Agent 发送 SIGTERM](adr/0039-an-agent-on-a-mac-is-sent-sigterm-when-asist-ends.md) |
| 0040 | [持续加载失败的语音识别与朗读引擎，把重新启动的间隔从 30 秒逐次加倍到 10 分钟](adr/0040-the-watchdog-waits-longer-between-restarts-of-a-speech-engine-that-keeps-failing-to-load.md) |
| 0041 | [角色为默认值时，不保存文本，按对话语言读取](adr/0041-the-default-persona-is-not-stored-and-follows-the-conversation-language.md) |
| 0042 | [自己的邮件地址只限于账号的地址与设置里写的地址，不从已发送的邮件推测](adr/0042-the-users-other-mail-addresses-are-listed-per-account-and-never-guessed-from-sent-mail.md) |
| 0043 | [日历在 Mac 和 Windows 上都只用 Google Calendar API 读写，不使用 macOS 的 EventKit](adr/0043-the-calendar-is-google-calendar-alone-on-mac-and-windows.md) |
| 0044 | [文件卡片把 zip 当作一个文件显示，不列出它的内容](adr/0044-the-files-card-shows-a-zip-as-a-file-and-does-not-list-its-contents.md) |
| 0045 | [文件卡片里打开的 HTML 页面只读取该文件夹内部，并在与网络隔开的会话中显示](adr/0045-an-html-page-in-the-files-card-is-confined-to-its-folder-and-reaches-no-remote-host.md) |
| 0046 | [笔记放在内存里供搜索，ASIST 之外的改动靠监视笔记文件夹来合入](adr/0046-notes-are-held-in-memory-and-the-folder-is-watched-for-changes-made-outside-asist.md) |
| 0047 | [文件查看器的繁重处理不在 Web Worker 里，而在另一个进程里运行的 iframe 里完成](adr/0047-the-viewers-heavy-work-runs-in-an-iframe-of-its-own-process-not-in-a-web-worker.md) |
| 0048 | [每次打开会话都用通知把当前时刻告诉 Gemini Live，指令文本里不写时刻](adr/0048-gemini-live-is-told-the-time-in-a-note-whenever-a-session-opens.md) |
| 0049 | [全天日程用首日和末日的日期，带时刻的日程用这台计算机的时钟，在与模型的交互和确认界面上呈现](adr/0049-calendar-events-reach-the-model-as-days-when-all-day-and-on-this-computers-clock-when-timed.md) |
| 0050 | [Excel 的工作表按 XML 原样保留，只解析要显示的行](adr/0050-an-excel-sheet-is-kept-as-its-xml-and-only-the-rows-in-view-are-parsed.md) |
| 0051 | [音频波形只对 MP3、AAC、WAV 绘制，FLAC 和 OGG 不显示波形只播放](adr/0051-an-audio-waveform-is-drawn-for-mp3-aac-and-wav-and-not-for-flac-or-ogg.md) |
| 0052 | [记忆连标题在内都用对话的语言写，模板在对话的 11 种语言里都有，整理的指令只有日语和英语](adr/0052-the-memory-is-written-in-the-conversation-language-from-templates-in-every-language.md) |
| 0053 | [CI 只在查看器的测量值远离平常值时才判为失败，细化的预算只作报告](adr/0053-ci-fails-the-viewer-budgets-only-far-from-normal-and-reports-the-tuned-numbers.md) |
| 0054 | [把用 ChatGPT 套餐对话的路径做成与 OpenAI 不同的提供商“ChatGPT”，不切换到 API 密钥](adr/0054-chatgpt-is-a-provider-of-its-own-on-the-users-plan-and-never-switches-to-an-api-key.md) |
| 0054 | [本地的语音模型只在麦克风需要它的期间加载](adr/0054-local-speech-models-load-while-the-microphone-needs-them.md) |
| 0055 | [关闭窗口和最小化时关掉麦克风，窗口恢复后麦克风仍保持关闭](adr/0055-the-microphone-turns-off-when-the-window-goes-to-the-tray-or-is-minimized.md) |
| 0056 | [不从 ASIST 读取任何东西的语音进程，在 Mac 上用监视 shell 让它随 ASIST 一起结束](adr/0056-speech-processes-that-read-nothing-from-asist-end-with-it-through-a-watcher-on-a-mac.md) |
| 0057 | [测试运行时把字典里的所有消息换成键，字典的抄写由该测试自身的失败来发现](adr/0057-tests-run-with-every-message-replaced-by-its-key.md) |
| 0058 | [需要准备的本地语音模型在启动时只通知一次，不属于任何固定的文件在启动时删除](adr/0058-speech-models-that-need-preparing-are-told-once-at-launch-and-files-no-pin-names-are-removed.md) |
| 0059 | [本地语音识别在与朗读相同的 speech.cpp worker 里运行，不随应用打包 llama.cpp](adr/0059-local-speech-recognition-runs-in-a-speech-cpp-worker-and-llama-cpp-is-not-bundled.md) |
| 0060 | [语音识别也能选 FastConformer 的 3 个模型，只列出能识别对话语言的模型，自动仍然保持 Qwen3-ASR](adr/0060-fastconformer-recognition-models-are-listed-by-the-conversation-language-and-auto-stays-qwen3-asr.md) |
| 0061 | [本地朗读只丢掉人声之前的完全静音，从第一个声音起等 0.2 秒后开始播放](adr/0061-local-speech-drops-only-the-silence-before-its-first-sound-and-starts-0-2-s-after-it.md) |
| 0062 | [本地语音模型的固定从随应用打包的 speech 的清单读取，文件放在以内容 SHA-256 命名的文件夹里](adr/0062-local-speech-model-pins-come-from-the-bundled-speech-catalog-and-files-are-kept-by-their-sha256.md) |
| 0063 | [长发言按静音分成 10 秒以下再转写](adr/0063-long-utterances-are-transcribed-in-parts-cut-at-pauses.md) |
