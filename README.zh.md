<p align="center">
  <img src="website/public/img/og-en.png" alt="ASIST —— 只要开口，日程和邮件就都办好了。" width="720" />
</p>

<h1 align="center">ASIST</h1>

<p align="center">
  面向 Mac 和 Windows 的实时助手
</p>

<p align="center">
  <a href="https://asist-agent.com">官网</a> ·
  <a href="https://github.com/nyosegawa/asist/releases/latest">下载</a> ·
  <a href="https://asist-agent.com/en/docs/">文档</a> ·
  <a href="README.md">English</a> ·
  <a href="README.ja.md">日本語</a>
</p>

<p align="center">
  <a href="https://github.com/nyosegawa/asist/actions/workflows/ci.yml"><img src="https://img.shields.io/github/actions/workflow/status/nyosegawa/asist/ci.yml?branch=main&style=flat-square&label=CI" alt="CI" /></a>
  <a href="https://github.com/nyosegawa/asist/releases/latest"><img src="https://img.shields.io/github/v/release/nyosegawa/asist?style=flat-square" alt="最新版本" /></a>
  <img src="https://img.shields.io/badge/macOS-14%2B%20%C2%B7%20Apple%20Silicon-lightgrey?style=flat-square" alt="macOS 14 或更高版本，Apple Silicon" />
  <img src="https://img.shields.io/badge/Windows-11%20%C2%B7%20x64-lightgrey?style=flat-square" alt="Windows 11，x64" />
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-green?style=flat-square" alt="MIT License" /></a>
</p>

ASIST 是一个你对着说话的助手，它用自己的声音回答。天气、日程和邮件以卡片的形式摆在对话旁边；耗时的资料查找和文件修改，在你批准之后交给 Agent（codex 或 claude 的 CLI）。对话用的模型由你从 Anthropic、OpenAI、Google、Cerebras 中挑选。

**你的数据留在你自己的电脑上。** 麦克风收音、记忆、笔记和对话历史都在这台电脑里。对话的文字只发给你选定的模型服务商，卡片只向它需要的天气或新闻服务查询。ASIST 不向开发者发送任何数据。

<p align="center">
  <img src="website/public/screens/en/home.webp" alt="ASIST 的主界面：中间是对话，左边是汇率卡片，右边是天气卡片，下方是 Dock" width="860" />
</p>

<table>
  <tr>
    <td width="50%"><img src="website/public/screens/en/calendar.webp" alt="日历迷你应用，显示一周的日程" /><br /><sub>日历可以查看、新建和修改 Google 日历里的日程。说一句“给我看看下周的安排”就能打开。</sub></td>
    <td width="50%"><img src="website/public/screens/en/agent.webp" alt="Agent 任务界面，有任务列表和一个任务的日志" /><br /><sub>Agent 任务。可以看进度，打开产出的结果。</sub></td>
  </tr>
  <tr>
    <td width="50%"><img src="website/public/screens/en/memory.webp" alt="记忆迷你应用，显示着 ASIST 写的一篇日记" /><br /><sub>记忆。每天一篇日记，还有它记得的事。</sub></td>
    <td width="50%"><img src="website/public/screens/en/mail.webp" alt="邮件迷你应用，显示收件箱和一个邮件串" /><br /><sub>邮件。负责朗读、总结和起草回复。</sub></td>
  </tr>
</table>

## 能做什么

- **开口就行。** 你话音一落，它就先用简短的一句应声；你中途插话，它会停下朗读。日语对话里，你说话时它还会适时附和。
- **答案摆在卡片上。** 天气、日程、邮件草稿、待办、汇率、新闻、地图、计时器等，在它出声回答的同时排列在对话旁边。
- **迷你应用。** Agent 任务、待办、笔记、邮件、记忆和日历都能从 Dock 打开，也能用一句话打开：“在日历里打开下周的安排。”
- **麻烦事交给 Agent。** 资料查找和文件修改交给 codex 或 claude 的 CLI，你可以看着进度，也能打开产出的结果。开始之前它一定会先问你。
- **它记得住。** 每天它把当天的对话整理成关于你的记忆，也写自己的日记。第二天的对话就从那里开始。
- **11 种语言。** 界面和对话支持 11 种语言。日语对话连附和与停顿都专门打磨过。

## 开始使用

你需要一台 Apple Silicon 的 Mac（macOS 14 或更高版本），或者一台 x64 的 Windows 11 电脑，再加上 Anthropic、OpenAI、Google、Cerebras 中任意一家的对话模型 API 密钥。在 Windows 上，要让语音识别和 Qwen3-TTS 在这台电脑里跑，需要一块支持 Vulkan 的独立显卡；没有独显，就用浏览器里的 Whisper 或 Live API 语音引擎来对话，也可以直接打字。日历需要 Google 账号。Agent 任务需要 codex 或 claude 的 CLI，两者都能用 ChatGPT 或 Claude 套餐登录，也能用 API 密钥。

1. 从 [Releases](https://github.com/nyosegawa/asist/releases/latest) 下载。Mac 上下载 `ASIST-arm64.dmg`，把 ASIST 拖进“应用程序”。Windows 上下载 `ASIST-Setup-x64.exe` 并打开；安装包还没有签名，SmartScreen 弹出警告时先点“更多信息”，再点“仍要运行”。
2. 打开 ASIST，在首次设置里选好语言、模型、声音和麦克风。
3. 开始说话。新版本会自动送达，下次退出时安装，Mac 和 Windows 都一样。

带截图的分步说明、麦克风权限、连接日历、准备 Agent 的 CLI，都在[入门指南](https://asist-agent.com/en/docs/start/)。

## 安全使用

ASIST 的回答和卡片由语言模型生成，可能有错，要紧的内容请自己核对。改日程、发邮件和 Agent 任务，都只在你批准之后才执行。批准过的任务由你自己的 codex 或 claude CLI 以该 CLI 的权限运行，可能改写你选定文件夹里的文件，所以批准之前先看清任务内容。API 用的都是你自己的密钥，费用由各家服务商直接向你收取，请分别设好额度上限。对话内容，以及选用 Live API 语音引擎时的音频，会发给你选定的服务商。开始使用前请先读[安全使用 ASIST](https://asist-agent.com/en/docs/privacy/safety/)；ASIST 以 MIT License 分发，不提供任何担保。

## 隐私

| 位置 | 内容 |
|---|---|
| 只在这台电脑上 | 麦克风收音、语音活动检测、附和判定、记忆检索。设置、记忆、笔记、待办、对话历史，以及收取的邮件 |
| 你选定的模型服务商 | 对话的文字、每次对话都会带上的两份记忆文档，以及记忆中与该次对话相关的部分。选用 Live API 语音引擎时，还有麦克风的音频 |
| 卡片数据的来源 | 只有卡片要用的那几个词，例如天气的地点、新闻的话题、货币 |
| codex 或 claude CLI 背后的服务 | 你批准的任务的提示词，以及 CLI 读取的文件 |

API 密钥和邮箱密码加密保存，macOS 上用钥匙串里的密钥，Windows 上用 DPAPI，不会传给任何子进程，Agent 的 CLI 也不例外。在 Windows 上，任何以你的用户身份运行的程序都能解密它们。完整的发送目的地清单在[隐私与数据](https://asist-agent.com/en/docs/privacy/)。

## 文档

面向用户的文档在 [asist-agent.com/en/docs](https://asist-agent.com/en/docs/)。

| 想了解 | 看这里 |
|---|---|
| 安装、首次设置、权限、Agent 的 CLI | [入门指南](https://asist-agent.com/en/docs/start/) |
| 怎么和 ASIST 说话、卡片、迷你应用 | [使用 ASIST](https://asist-agent.com/en/docs/usage/) |
| 外观、语言与地区、模型、声音、Agent | [设置](https://asist-agent.com/en/docs/settings/) |
| 数据放在哪里、发送到哪里 | [隐私与数据](https://asist-agent.com/en/docs/privacy/) |
| 出问题的时候 | [疑难解答](https://asist-agent.com/en/docs/troubleshooting/) |
| 用到的模型、外部数据和许可证 | [参考](https://asist-agent.com/en/docs/reference/models/) |
| 从源码运行、构建、发布 | 本仓库的 [docs/zh/development.md](docs/zh/development.md) |
| 设计决策及其理由 | 本仓库的 [docs/zh/adr/](docs/zh/adr/) |

中文译文的索引在 [docs/zh/README.md](docs/zh/README.md)。

## 开发

```bash
npm install
npm run dev      # 从源码启动
npm test
```

除了 Node.js 和 npm，Mac 上还需要 Xcode Command Line Tools，Windows 上需要 Visual Studio 的“使用 C++ 的桌面开发”工作负载。构建、CI、检查界面和发布的步骤见 [docs/zh/development.md](docs/zh/development.md)，用 Agent 开发时的规则见 [AGENTS.md](AGENTS.md)。

缺陷报告和功能需求欢迎发到 [Issues](https://github.com/nyosegawa/asist/issues)。安全漏洞请不要发公开 Issue，按 [SECURITY.md](SECURITY.md) 的步骤告知。

## 许可证

采用 [MIT](LICENSE)。应用里打包的 git 是 GPL-2.0，其源码附在每个版本的发布里。ASIST 用到的模型和数据的许可证在[参考](https://asist-agent.com/en/docs/reference/models/)。
