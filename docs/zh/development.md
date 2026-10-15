# 开发

使用 Electron、React、TypeScript、electron-vite、Vitest。开发时的规则在 [AGENTS.md](../../AGENTS.md) 里，字符串写在哪里在 [ui-text 技能](../../skills/ui-text/SKILL.md) 里，设计的判断及其理由在 [docs/adr](adr/) 里。

```bash
npm run typecheck
npm test
npm run build
```

`npm test` 首先准备随应用打包用的 git（macOS 上是编译，Windows 上是获取 MinGit），然后获取随应用打包用的 uv 和测试要用的 Electron。

记忆整理的确认脚本（`resources/skills/` 里的 Python）的测试，与整理用的 Agent 一样，用内置的 uv 以 `uv run --no-project` 运行，使用该计算机上已有的 Python。因为不让 uv 获取 Python，所以找不到 Python 时测试失败。CI 的 `test` 和 `test-windows` 使用下面的 Python 3.12。

`tests/vap-worker.test.ts` 实际运行 MaAI 的 worker（`resources/vap_worker.py`），所以使用装有 numpy 的 Python。环境变量 `ASIST_VAP_PYTHON` 存在时用那个 Python，不存在时用 PATH 里的 `python3` 或 `python`。本地找不到时跳过这个测试，CI 里找不到时判定为失败。CI 的 `test` 和 `test-windows` 先装入与应用环境相同的 Python 3.12 和 numpy 2.5.2，再执行 `npm test`。

`tests/local-tts-live.test.ts` 从 ASIST 的朗读客户端经由随应用打包的 speech worker，让 Irodori-TTS 的 3 个声音和 Qwen3-TTS 0.6B 朗读日语句子，并输出准备完成所需的时间和到第一个声音的时间。因为需要模型文件，平时跳过。升级 speech.cpp 时，把按应用相同布局（`<owner>--<name>/<SHA-256>/<文件名>`）放置了 `speech-models/` 的文件夹传给 `SPEECH_LIVE_USER_DATA`，用 `npx vitest run tests/local-tts-live.test.ts --silent=false --reporter=verbose` 运行。文件夹里可以用 `cp -c` 复制本地文件放进去。从 speech.cpp 0.8.2 起，speech.cpp 自身的缓存也是相同的布局。

模型文件的固定，从与随应用打包的 speech 相同版本的清单文件读取。测试使用复制了 speech.cpp 0.8.1 的 `tools/catalog/catalog.json` 的 `tests/fixtures/speech-models.json`。升级 speech.cpp 时，换成该版本的清单。

`tests/local-asr-live.test.ts` 从 ASIST 的语音识别客户端经由随应用打包的 speech worker，让 Qwen3-ASR 的 1.7B 和 0.6B 以及 FastConformer 的 3 个模型转写录音，再与各自的官方实现在指定语言下写出的句子比对。FastConformer 的官方句子是 NeMo 写出的句子。也会确认模型的语言列表与文件列表一致。还会输出准备完成所需的时间、转写和中间转写花的时间，以及在 Mac 上转写完 ASIST 一次发送的最长 20 秒的发言之后 worker 的内存。因为需要模型文件和录音，平时跳过。升级 speech.cpp 时，把放着 `speech-models/` 的文件夹传给 `SPEECH_LIVE_USER_DATA`，把放着 16 kHz 的 WAVE 文件和官方句子的 `requests.json` 的文件夹传给 `ASR_LIVE_CLIPS`，用同样的方式运行。没有文件或 `requests.json` 里没有对应行的模型跳过。`requests.json` 的形式写在测试文件的开头。

在 Windows 上，`tests/memory-curation-service.test.ts` 确认记忆文件夹用 8.3 形式的短名称（形如 `C:\Users\RUNNER~1\…` 的名称）书写时也能把整理结果合入。测试在系统盘的 `%LOCALAPPDATA%\Temp` 里创建文件夹，并向 Windows 询问它的短名称。卷设置为不生成短名称时，给出理由并跳过这个测试。存在环境变量 `ASIST_REQUIRE_SHORT_NAMES=1` 时（CI 的 `test-windows`），不跳过，判定为失败。

## 在 Windows 上 clone

在 Windows 11（x64）上作业时，clone 之前先完成以下事项。

- 在“設定”→“システム”→“開発者向け”中启用开发者模式。
- 在 Git for Windows 中启用符号链接和长路径。

```powershell
git config --global core.symlinks true
git config --global core.longpaths true
git clone git@github.com:nyosegawa/asist.git
```

`.claude/skills` 和 `.agents/skills` 是指向 `skills/` 的符号链接。开发者模式和 `core.symlinks=true` 缺少任何一个，两者都会变成写着 `../skills` 的文本文件，Claude Code 和 Codex 就一个技能也读不到。用 `dir .claude` 显示出 `<SYMLINKD>` 时，说明是链接。如果变成了文本文件，就启用开发者模式并执行 `git config core.symlinks true`，再用 `git checkout -- .claude/skills .agents/skills` 重新创建。

换行符由仓库的 `.gitattributes` 统一成 LF，在任何 OS 上都一样。不需要修改 Git for Windows 的换行符设置（`core.autocrlf`）。

## 从源码启动

需要 Node.js、npm、Xcode Command Line Tools。编译麦克风的原生 helper 和 git，获取 uv，随应用打包。

```bash
npm install
npm run dev
```

在 Windows 11（x64）上，需要 Node.js 22、按上面的“在 Windows 上 clone”设置好的 Git for Windows，以及 Visual Studio 的“C++ によるデスクトップ開発”。Visual Studio 只装 Build Tools 也可以。准备脚本获取 MinGit 代替 git，并用 `vswhere` 找到的 Visual Studio 的 `cl.exe` 编译两个程序：运行 Agent 的 CLI 的 `asist-agent-launcher.exe`，以及让麦克风经由 Windows 的回声消除采集声音的 `asist-mic.exe`。已在 Visual Studio Community 2017 和 CI（windows-latest）的 Visual Studio 2026 上验证过。命令与 macOS 相同。应用把设置和记忆写在 `%APPDATA%\asist`，把日志写在 `%APPDATA%\asist\logs`。Windows 只显示具有与开始菜单快捷方式相同的 AppUserModelID 的应用的通知。`npm run dev` 启动的应用没有快捷方式，所以通知要在安装后的应用上确认。

在 Windows 上，npm 脚本在 `cmd.exe` 中运行，Claude Code 的 Bash 工具在 Git Bash 中运行。不安装 Python。随应用打包的 uv 会获取固定版本的 Python。要在 Windows 机器上把 ASIST 的行为也确认到位时，还要预先安装以下各项。

- **GPU 驱动程序。** 本地的语音识别和 Qwen3-TTS 通过 Vulkan 运行，需要独立 GPU。用 `resources\speech\speech.exe devices` 确认 ASIST 所看到的 GPU 的名称、种类和显存。更新 GPU 驱动程序之后的第一次启动要构建 Vulkan 着色器，所以准备需要几十秒。
- **Claude Code。** 用官方的原生安装程序（`irm https://claude.ai/install.ps1 | iex`）安装，把安装到的 `%USERPROFILE%\.local\bin` 加入用户的 PATH。用 npm 安装的 claude 只有 `.cmd`，ASIST 无法启动它，而且它还在的话在终端里也会被先找到，所以用 `npm uninstall -g @anthropic-ai/claude-code` 删除。确认 `Get-Command claude -All` 只列出原生的那个。
- **Codex CLI。** 安装 PowerShell 7（`winget install --id Microsoft.PowerShell --exact`），再从其中用官方安装程序（`irm https://chatgpt.com/codex/install.ps1 | iex`）安装。它会装到 `%LOCALAPPDATA%\Programs\OpenAI\Codex\bin`。2026-09-27 时，在 Windows 自带的 PowerShell 5.1 上安装程序读不到 `OSArchitecture` 而停止，winget 的 `OpenAI.Codex`（0.157.1）从链接启动时以 `the CLI package does not match this platform or executable` 停止。安装后，在终端里启动一次 `codex`，准备 Windows 的 sandbox。准备完成之前，ASIST 不会开始 codex 的任务。
- **GitHub CLI。** 用 `gh auth login`，git 的方式选 SSH，在浏览器中认证。只有 SSH 密钥无法使用 gh 的 API。
- **Google Chrome。** 用于 `scripts/cdp` 的截图和测量。

像 `npm run dev` 这样未打包的应用，读取启动时所在执行目录的 `.env`。从父进程继承的环境变量优先于 `.env`。打包后的应用无论从哪里启动都不读 `.env`。即使环境变量里有 `ELECTRON_RENDERER_URL`，也打开包内的页面。

API 密钥在环境变量（未打包的应用中还包括执行目录的 `.env`）里有就用它，没有就用设置里保存的密钥。环境变量里的 provider 密钥无法在设置里保存。在设置里保存的密钥用 Electron 的 safeStorage 加密，不以明文写入。加密的密钥在 macOS 上由钥匙串保护，在 Windows 上由 DPAPI 保护，放在 userData 的 `Local State` 里。无法使用加密时不保存，报错处理。在 macOS 上，开发版和安装后的应用密钥不同，所以一方保存的密钥另一方读不到。读不到时会报错，所以在设置里重新输入。在 Windows 上，开发版和安装后的应用使用相同的 userData 和相同的密钥，所以一方保存的密钥另一方也能读到。

如果两个构建固定的语音模型文件不同，却使用同一个 userData，那么后启动的一方会把只有对方构建使用的模型文件从 `speech-models/` 中删除。交替启动开发版和安装后的应用，每次都要重新准备。

地图卡片需要 Google 的 Maps Embed API 密钥。密钥在构建时嵌入。在 Google Cloud 中启用 Maps Embed API 并创建密钥，写进仓库根目录的 `.env`，然后执行 `npm run dev` 或 `npm run dist:mac`。

```bash
RENDERER_VITE_GOOGLE_MAPS_EMBED_KEY=...
```

这个密钥会出现在 iframe 的 URL 里，所以能从分发的应用中取出。在密钥的“API の制限”中只允许 Maps Embed API，不要在同一项目里启用其他付费 API。正式版的应用从 `file://` 打开，不会发送 Referer，所以无法使用网站限制。Google 的条款不允许更改地图的归属显示，因此卡片的地图没有加颜色滤镜。

### 连接 Google 日历

日历在 macOS 和 Windows 上都通过 Google Calendar API 读写。登录用的 OAuth 客户端在构建时嵌入应用。从仓库运行时，把 Google Cloud 项目（按 issue #81 的步骤创建的那个）的桌面应用客户端值写进仓库根目录的 `.env`。

```bash
ASIST_GOOGLE_CLIENT_ID=...apps.googleusercontent.com
ASIST_GOOGLE_CLIENT_SECRET=...
```

用 `npm run dev` 启动，在设置的“カレンダーとメール”里的“カレンダー”中按“Google でログイン”。浏览器会打开同意界面，勾选权限的 2 个项目的两项。登录后开启日历联动，选择要显示的日历和新日程的保存目标。

客户端 ID 和密钥由 `electron.vite.config.ts` 在构建时嵌入 main 进程。`npm run dev` 读取启动时的 `.env`，`npm run dist:mac` 等读取执行命令的 shell 的环境变量或 `.env`。在 CI 中，只向构建应用的 `build` 和 `build-windows` 的 job 从仓库的 secret 以相同名称传入。没有客户端而构建出的应用也能启动，日历界面也会出现，但第一次使用日历时（例如打开设置的日历项目时）会变成告知没有客户端的错误。分发的应用由 `npm run release` 和 Windows 的发布工作流确认客户端存在之后再构建。

登录的 refresh token 与 API 密钥一样用 safeStorage 加密，保存到 `userData/google-calendar.json`。在 macOS 上，开发版和安装后的应用密钥不同，所以一方登录的内容另一方读不到。设置里会显示读不到，因此重新登录，或者登出。

## CI

GitHub Actions（`.github/workflows/ci.yml`）在每次向 main push 和每次 pull request 时，同时运行下面的 job。应用的 job 在 Apple Silicon 的 macOS 和 x64 的 Windows 上运行。

| job | 确认的事项 |
| --- | --- |
| `test` | `npm run typecheck`、`npm run i18n -- check`、`npm test`。测试在 CPU 3 个的 runner 上分成 3 个 worker 运行（`--maxWorkers=3`） |
| `viewer-budgets` | `npm run demo:viewer-budgets -- --time-factor 3`。文件卡片的各个查看器没有退回到把大文件整个读取、整个绘制的方式。判定失败的只是与平时的值相差很大时，例如到显示为止 5 秒、主线程停住 2 秒、内存增加 1.5 GB 等。每个 case 的预算（更严格的值）只报告，时间按共享的 runner 乘 3 倍来比较。生成的示例文件以生成它的脚本的版本为名称留在缓存里，之后不再重新生成。但 WAV 和 zip 无法压缩、体积又大，所以不保留，每次花 3 秒左右重新生成。没有缓存时，用 Homebrew 安装生成 MP3 的 `lame` |
| `fit` | `npm run demo:fit`。在 11 种语言和所有主题下，卡片和界面的文字都放得下。把主题分到 2 台机器上（`--shard 1/2` 和 `2/2`）同时检查 |
| `build` | 用 `npm run dist:mac:unsigned` 一直构建到包含原生 helper、git、uv 的未签名应用，并确认应用里的 git 和 uv 能运行 |
| `test-windows` | 在 Windows 上 `npm test`。把测试文件分到 2 台机器上（`--shard 1/2` 和 `2/2`）同时运行。临时文件夹（`TEMP` 和 `TMP`）放在不生成 8.3 形式名称的 runner 的 D: 盘。类型和字典不依赖 OS，所以只用 `test` 确认 |
| `build-windows` | 用 `npm run dist:win:dir` 构建 Windows 的应用，确认应用里的 git、uv、`asist-agent-launcher.exe`、`asist-mic.exe` 能运行，`ASIST.exe` 通过 ASAR 的检查并启动 |
| `website` | 在 Ubuntu 上构建网站（`website/`），确认所有页面的链接和图片的指向 |
| `result` | 其他 job 全都成功，或者按各自的条件被跳过。没有分配到 runner 就结束的 job 也算作失败（`scripts/ci-result.mjs`） |

main 的 ruleset 作为合并条件的只有 `result`。只改 `website/` 内部时，跳过应用的 job。`fit` 只在改了 demo 读取的文件时运行。demo 读取的是 renderer、`src/shared`、demo 的设置、fit 的脚本、依赖等，只改 main 进程、测试或文档，demo 的显示不会变。`viewer-budgets` 在改了文件卡片、卡片外框、查看器、预览页面以及它们 import 的文件、测量的脚本、依赖等时运行，另外每周二早上 6 点（日本时间）运行。这份列表与顺着 import 追出来的结果一致，由 `tests/ci-viewers-list.test.ts` 确认。测量的页面是整个应用，所以列表之外的改动影响了数字时，会在一周一次的运行中知道。列表在 `ci.yml` 的 `changes` job 里。跳过的 job 不算作失败，所以 pull request 可以直接合并。

向 pull request push 新的提交时，会取消还在为前一个提交运行的执行。向 main 的 push 不取消，即使连续合并，也对每个提交确认到底。这是为了失败时能知道是哪个提交弄坏的。

CodeQL（`.github/workflows/codeql.yml`）在每次向 main push 时和每周 1 次，分析进入应用的部分的 JavaScript/TypeScript、Python、Actions。不在 pull request 上运行。只在开发时使用的 `scripts/`、`tests/`、`website/`、`promotions/`、`skills/` 排除在分析对象之外（`.github/codeql/codeql-config.yml`）。

随应用打包的 git 和 uv，以准备它们的 `scripts/resources/` 里 git 和 uv 模块的内容为键做缓存。`VERSION` 文件记录该模块的哈希，准备脚本在内容一致时就直接使用从缓存恢复出来的东西。`npm audit --omit=dev` 在每次向 main push 时和每周二早上 6 点（日本时间）由另一个任务运行。因为依赖的漏洞即使不改代码也会在之后公开。签名构建和公证还没有放进 CI。

## 用浏览器确认界面

只确认界面时，不启动 Electron，直接提供 demo。把固定的数据（`src/renderer/src/demo/fixtures/`）传给应用本体的组件来查看。

```bash
npm run demo
```

| URL | 能看到的东西 |
|---|---|
| `http://localhost:5174/` | 带示例列表的外框。在左侧选择界面或卡片，在上方选择语言和窗口大小。 |
| `/screens/setup?size=l` | 以应用本体窗口的大小（l / m / s）查看初次设置。`calendar`、`settings/voice`、`boot/error` 等也一样。名称在 `src/renderer/src/demo/screens.ts` 里。 |
| `/cards/files-pdf` | 把 1 个卡片示例按 s / m / l / focus 并排查看。`/cards` 是全部。 |
| `/app?say=ドル円のレートを教えて` | 没有外框的应用。用文字搭话来使用。`say=` 的句子与出现的卡片的对应关系在 `src/renderer/src/demo/sayings.ts` 里。 |
| `/i18n?group=settingsVoice` | 界面文案的列表。用 `?q=` 过滤，用 `?langs=ja-JP,en-US` 选择语言。 |

任何 URL 都可以像 `?lang=de-DE` 这样附加语言。在外框的 URL 后加 `/preview` 就是外框的内容，截图脚本直接打开这里。

外观的确认和测量用把 headless Chrome 通过 CDP 驱动的 `scripts/cdp/` 脚本完成。脚本每次运行时，都在 OS 选出的空闲端口上自行启动它所在工作树的 demo 和 Chrome，结束时关闭。不需要事先启动 `npm run demo`，每个 worktree 同时运行也不会冲突。步骤在[外观调试](../../skills/visual-debugging/SKILL.md)里，卡片的设计在[面板卡片步骤](../../skills/panel-card-design/SKILL.md)里。

| 命令 | 做的事 |
|---|---|
| `npm run demo:drive -- --launch <步骤>` | 按顺序执行步骤，做操作、测量、截图。加上 `--port 9222` 后，对用[安装步骤](../../skills/install-mac-app/SKILL.md)的 `--launch --cdp` 启动的应用本体也能使用同样的步骤。 |
| `npm run demo:open` | 让 demo 和 Chrome 保持打开，并显示 Chrome 的端口。适合一边修改界面一边用 `demo:drive -- --port <该端口>` 反复测量时使用。 |
| `npm run demo:cards`、`npm run demo:gallery` | 把卡片按 3 个大小截图。把列表截成 1 张。 |
| `npm run demo:setup` | 把初次设置从头到尾走一遍并截图。 |
| `npm run demo:docs-shots` | 把要放到 README 和文档里的界面用日语和英语重新截到 `website/public/screens/`。 |
| `npm run demo:fit` | 检查在所有语言下卡片和界面的文案是否放得下。大约需要 1 分钟。 |
| `npm run demo:viewer-budgets` | 在临时文件夹里生成大的 PDF、Office 文件、音频等，用文件卡片及其放大显示打开并滚动到最后，测量到显示为止的时间、主线程停住的最长时间、renderer 的内存。超过 `scripts/cdp/scenes/viewer-budgets.mjs` 的 `LIMITS`（case 也可以有自己的值）时以退出码 2 结束。写在 case 里的预算是按开发所用 Mac 上的值定的严格数字，超过也只报告。查看器不显示内容而显示错误或“大きすぎる”时、滚动放大显示后内容不出现时、页面或其中的 frame 崩溃时，也算失败。`--measure all` 不设预算也不设上限，测量所有文件。内存用 `ps` 读取，所以在 Windows 上无法测量，会报错停止。生成 MP3 需要 `lame`，生成 m4a 需要 macOS 的 `afconvert`。 |

使用真实服务的自测要准备好 API 密钥和语音的服务，在构建之后运行。

```bash
ASIST_SELFTEST=1 npx electron .
```

## 重新生成随应用打包的素材

- **Irodori-TTS 和 Qwen3-TTS 的附和音频。** 应用不在运行时合成，而是使用随应用打包在 `resources/aizuchi/<引擎>/<声音>/` 里的东西。因为每个附和都要先用语音识别和耳朵确认之后再放进去。改了附和的文案（`src/shared/aizuchi-bank.ts`）时，或者增加声音时，用 `node scripts/aizuchi-clips/build.mjs <irodori 或 qwen3tts> <声音> [候选的个数] [句子…]` 生成。Qwen3-TTS 只让它读一句短话时有时会连着说几秒，所以脚本把附和接在后续句子的前面让它读几次，在后续句子能被语音识别原样听到的那段区间里切出附和。Irodori-TTS 则让它单独读附和，把前后的静音和最后短的噪声压掉。两者都用语音识别确认附和的文案，把长度最典型的候选做成 clip，把排在前面的候选留在 `.aizuchi-candidates/`（不在 Git 的对象内）。带上合格标记（manifest 的 `reviewed`）的附和，只要不点名句子就不重新生成。朗读和语音识别使用随应用打包的 speech（`node scripts/prepare-resources.mjs dev`），读取应用准备好的 Irodori-TTS 或 Qwen3-TTS 0.6B，以及 Qwen3-ASR 1.7B 的文件。使用放在其他位置的文件时，用 `ASIST_SPEECH_MODELS` 指定那个文件夹。附和按 0.6B 生成，朗读改成 1.7B 时也播放同样的东西。
  - 生成之后，用 `npm run demo` 的 http://localhost:5174/aizuchi 全部听一遍再决定。从候选中选择并设为“合格”，那个 clip 就带上合格标记。判为“だめ”的附和，可以用左侧的“だめな N 件を作り直す”重新生成。等全部合格之后再提交。
- **Irodori-TTS 的声音。** `resources/irodori-voices/<声音>.voice.gguf` 是用随应用打包的 speech 的 `speech voice` 在 CPU 上转换 speech-bench 生成的参考音频（`~/speech-bench-data/references/voice-<声音>.wav`）得到的。声音文件带有生成时所用编解码器的哈希，别的编解码器的模型不会读它。speech.cpp 改变声音文件的形式时也不再读。升级后的 speech.cpp 的清单把 Irodori-TTS 固定到别的编解码器的文件时，以及声音文件的形式改变时，用 `node scripts/irodori-voices.mjs [参考文件夹]` 重新生成。脚本用 `node scripts/prepare-resources.mjs dev` 放好的 speech，读取应用准备好的 Irodori-TTS 的文件（或 `ASIST_SPEECH_MODELS` 的文件夹）。因为在 CPU 上生成，所以从同样的参考会得到同样的文件。增加声音时，要确认短的附和里会不会多出词，再决定（理由见 ADR 0031）。
- **live 的声音示例。** `npm run gen:live-voices` 让 provider 的 TTS 读同样的句子，放到 `src/renderer/src/assets/live-voices/`。
- **Irodori-TTS 和 Qwen3-TTS 的声音示例。** `node scripts/gen-tts-voices.mjs` 生成还没有的示例，放到 `src/renderer/src/assets/tts-voices/<引擎>/<语言>/<声音>.mp3`。Irodori-TTS 只有日语，Qwen3-TTS 是会成为对话语言的 8 种语言，句子在 `src/shared/voice-samples.ts` 里。只使用语音识别听起来与句子一致的读法。与附和一样，使用随应用打包的 speech 和应用准备好的模型。把引擎、语言、声音作为参数传入时，只重新生成那一部分。
- **图标。** 原始图片和生成提示词在 `resources/artwork/` 里，可以用 `python3 scripts/gen-icon.py` 重新生成（需要 Pillow）。
- **天气图片。** 47 个都道府县的风景和 5 种天空状态作为各自独立的图片叠加。生成提示词在 `src/renderer/src/assets/weather/` 的各个 `prompts.json` 里，用 `python3 scripts/check-weather-alpha.py` 检查 PNG 的 alpha。获取和对应表的更新在[天气的规格](../../src/main/services/weather/SPEC.md)里。
- **macOS 许可对话框的文案。** `scripts/macos/permission-texts.mjs` 在构建之前写出 11 种语言的文案。macOS 不用应用的设置，而用系统的语言选择这段文案。

## 网站和介绍视频

介绍页面和文档在 [website/](../../website/) 里，在 https://asist-agent.com 公开。是用 Astro 和 Starlight 构建的、与应用不同的 npm 项目。要发到 YouTube 和 X 的介绍视频及其缩略图在 [promotions/launch-video/](../../promotions/launch-video/) 里。两者都不包含在应用里。

```bash
npm --prefix website install # 安装网站的依赖（只在一开始时做一次）
npm run website              # 在 http://localhost:5194 打开
npm --prefix website test    # 运行把页面转换成 Markdown 的处理的测试
npm run website:build        # 写出到 website/dist，确认所有页面的链接和图片
npm run website:deploy       # 发布到 asist-agent.com（从 main）
npm run promo:video          # 把介绍视频生成到 promotions/launch-video/out/asist-launch-video.mp4
npm run promo:thumbnail      # 把 YouTube 的缩略图生成到 promotions/launch-video/out/youtube-thumbnail.png
```

写文档的位置、翻译方法、发布的步骤在 [website 技能](../../skills/website/SKILL.md)里。

## 构建 macOS 的应用

指定签名的证书后运行。应用输出到 `dist/`。

```bash
CSC_NAME="<姓名> (<Team ID>)" npm run dist:mac
```

证书使用与要分发的版本相同的 Developer ID Application。macOS 把麦克风的许可与签名绑定，所以替换成用别的证书签名的应用后，许可要重新来过。应用启用了库的验证，所以用自签名证书签名时，应用本体和 Electron Framework 的 Team ID 不一致，启动后立刻退出。`CSC_NAME` 传 `security find-identity -v -p codesigning` 列出的名称去掉“Developer ID Application: ”之后的部分。加前缀 electron-builder 会停止，不加则自己选择 Developer ID Application 的证书。省略 `CSC_NAME` 时 electron-builder 从钥匙串选择证书，所以存在自签名证书时必须指定。

这个构建既不生成 dmg 也不生成 zip，所以没有自动更新的设置（`app-update.yml`），不会从 Release 更新自己。

`npm run build` 之后 `scripts/third-party-notices.mjs` 运行，收集进入 bundle 的 npm 包和进入 app.asar 的生产用依赖包的许可证，写进 `build/THIRD_PARTY_NOTICES.txt`。应用的 `Contents/Resources` 里放有它和 ASIST 的 `LICENSE.txt`、Electron 和 Chromium 的许可证，可以从“このアプリについて”打开。如果有包没有写许可证，构建就在那里停止。

要构建未签名的版本，用 `npm run dist:mac:unsigned`。未签名时，麦克风的许可有时在重启之后不保留，所以语音的真机确认使用签名后的应用。分发之前还要用 `npm audit --omit=dev` 确认依赖。

构建出的应用通过 Electron 的 fuse（`electron-builder.yml` 的 `electronFuses`）不接受 `ELECTRON_RUN_AS_NODE`、`NODE_OPTIONS`、`--inspect`，不从 `app.asar` 以外加载应用的代码，`app.asar` 的内容有变动就不启动。这些都是为了防止其他进程顶着 ASIST 的签名，使用用户许可的麦克风。构建之后可以用 `npx @electron/fuses read --app dist/mac-arm64/ASIST.app` 确认值。`--remote-debugging-port` 不会被 fuse 挡住，所以只在用 CDP 驱动应用时加上它启动。

放到开发机的 `/Applications` 里确认之前的步骤在[安装步骤](../../skills/install-mac-app/SKILL.md)里。

## 构建 Windows 的应用

在 Windows 11 的 x64 机器上执行。准备脚本准备的是为构建机器的 git、uv、原生 helper，所以在其他 OS 上会在 `npm run build` 之后停止。

```powershell
npm run dist:win       # 把安装程序生成到 dist\ASIST-Setup-x64.exe
npm run dist:win:dir   # 把不用安装就能运行的应用生成到 dist\win-unpacked
```

证书还没有设置，所以不签名。安装程序不请求管理员权限，按用户安装到 `%LOCALAPPDATA%\Programs\asist`。`electron-builder.yml` 没有让 Windows 带上 GitHub 的公开设置，所以这个安装程序没有自动更新的设置（`app-update.yml`），不会从 Release 更新自己。Electron 的 fuse 与 macOS 相同。`app.asar` 的内容有变动时，`ASIST.exe` 启动后立刻退出。加上 `ELECTRON_ENABLE_LOGGING=1` 启动时，那时会显示 `Integrity check failed for asar archive`。

在 Windows 机器上安装、确认、退出之前的步骤在 [Windows 安装步骤](../../skills/install-windows-app/SKILL.md)里。

## 发布

要分发的 Mac 应用在这台 Mac 上签名和公证，Windows 的安装程序由 GitHub Actions 不签名地构建，放到同一个 GitHub 的 Release 里。用户的应用无论 Mac 还是 Windows，都在启动时和每 6 小时确认新版本，在后台获取，下次退出时替换。用“このアプリについて”的“今すぐ再起動”也可以立刻替换。在 Windows 上，安装程序不显示界面就替换，选“今すぐ再起動”时在替换之后打开 ASIST。Electron 准备好之后启动失败的应用，关闭错误后退出，重新做一次只做更新的启动（`--update-after-failed-start`）。这个启动不打开窗口也不打开服务，确认新版本，有的话获取并替换，然后打开 ASIST。

只在最开始做一次：把 Developer ID Application 的证书放进钥匙串（Xcode 的 Settings → Accounts → Manage Certificates），把公证用的 Apple ID 保存到 notarytool。App 专用密码在 account.apple.com 创建。

```bash
xcrun notarytool store-credentials asist-notary --apple-id <Apple ID> --team-id <Team ID>
```

每次发布时，先合并提高 `package.json` 的 `version` 的 pull request，然后在 main 上执行。

```bash
npm run release
```

`npm run release` 先确认 main 与 origin/main 相同、没有改动，以及那个版本还没有发布。接着确认地图的密钥和 Google 的 OAuth 客户端（`RENDERER_VITE_GOOGLE_MAPS_EMBED_KEY`、`ASIST_GOOGLE_CLIENT_ID`、`ASIST_GOOGLE_CLIENT_SECRET`）存在，并确认 `npm audit --omit=dev`，然后构建 dmg 和 zip，做签名和公证，确认 Gatekeeper 接受。在这之上，把 dmg（`ASIST-arm64.dmg`。名称里不含版本，所以 `releases/latest/download/ASIST-arm64.dmg` 总是指向最新版）、zip、`latest-mac.yml`、随应用打包的 git 的源码（`scripts/resources/git-macos.mjs` 的版本的 tarball）放到 Release 里。选择证书的 `CSC_NAME` 和选择 notarytool 配置文件的 `APPLE_KEYCHAIN_PROFILE`（省略时是 `asist-notary`）都可以改。

放了 Mac 文件的 Release 仍是草稿状态。`npm run release` 接着传入 tag 和提交来运行 `.github/workflows/windows-release.yml`，用 `gh run watch --exit-status` 等待它结束。这个工作流首先确认 Release 是草稿，而且是从那个提交创建的。接着在 windows-latest 上确认 `package.json` 的版本与 tag 相同，填入仓库 Secrets 里的地图密钥和 Google 的 OAuth 客户端（`RENDERER_VITE_GOOGLE_MAPS_EMBED_KEY`、`ASIST_GOOGLE_CLIENT_ID`、`ASIST_GOOGLE_CLIENT_SECRET`），构建 NSIS 的安装程序。npm 和仓库的脚本只在这个构建的 job 里运行，该 job 的 token 只被赋予读取权限。持有可写入 Release 的 token 的 job 不 checkout 任何东西，只运行 gh。GitHub 的公开设置只在此时通过命令行传入，所以只有这个安装程序带有自动更新的设置（`app-update.yml`）和 `latest.yml`。生成的安装程序（`ASIST-Setup-x64.exe`。名称里不含版本，所以 `releases/latest/download/ASIST-Setup-x64.exe` 总是指向最新版）、它的 blockmap、`latest.yml`、随应用打包的 Git for Windows 的源码（`git-for-windows-<版本>.tar.gz`）作为 Actions 的 artifact 交给可写入 Release 的 job。那个 job 再一次确认 Release 仍是草稿、而且是从同一个提交创建的，然后把这些放到草稿里。`npm run release` 确认草稿里 Windows 的文件已全部齐全之后，公开 Release。工作流失败时，停在草稿状态，并显示接下来要做的事。

发布的版本有问题时，不要删除 Release，也不要退回前一个版本，而是提高编号，发布修好的版本。因为自动更新只会装入编号比当前版本新的版本。

## 真机确认

改了行为时，用签名后的应用和真实的服务确认相关的项目。分发之前整体做一遍，把用过的提交、构成、结果留在工作汇报里。

- **初次设置。** 确认选择语言后后续界面变成该语言、认证、模型的准备、麦克风的许可。确认只用文字的方式也能对话。
- **附和和衔接**（日语）。用扬声器试短的发言和长的发言，确认附和、衔接语（在附和之后、正式回答之前响）、打断、不会重新拾取自己的声音。确认问候不响附和，困扰有共情，“〜だっけ”有短的确认，订正有道歉的附和，HUD 显示“相槌: 調べ物 98%”这样的判定，“えっとね、昨日の”这种说到一半之后的停顿里发言不被确定。MaAI 有效时，还要确认说完的话很快确定、含糊时等待时间会延长、朗读中的“うん”不会让朗读停止。
- **非日语对话。** 把对话的语言设为英语、地区设为美国，确认回答用英语返回、天气从 Open-Meteo 以华氏度显示、新闻变成英语圈的、附和分类器和 MaAI 停止、设置里的“声”和“概要”中只属于日语的项消失。确认保存其他 1 个设置后，语言和地区也不会变回去。改回日语，确认附和重新工作。
- **卡片。** 确认卡片的显示和朗读一致，即使回答途中改口，旧的回答和卡片也不会混进来。改变窗口的高度，确认卡片在 3 个大小之间切换时不会在下面被切掉、最小的高度也放得下、放大显示时全部可见。
- **Qwen3-TTS。** 确认长的回答里，第一个句子还没全部生成完就出声、句子与句子之间不会空太久、每个句子的音量一致、朗读中搭话会停止、附和和衔接用同一个声音响、切换到别的引擎时 Qwen3-TTS 的 worker 结束。内存 24GB 以上的 Mac 上，在设置的“声”里把模型换成 1.7B，用其下面的“準備する”准备之后，确认用新模型朗读。
- **被保存的东西。** 确认关闭卡片之后和重启之后计时器仍在走，任务、笔记、对话的历史都留下。
- **记忆。** 整理前一天的对话，确认记忆的合入、`me.md` 和 `user.md` 会进入下一次对话、整理的 Agent 用技能文件夹里的 uv 执行 `count.py` 和 `validate.py` 把两者收在上限内（在没有 Node 和自己 Python 的机器上，以及在装了自己 uv 的机器上）、以前版本的 `instruction.md` 在启动时移到 `me.md` 和 `user.md`、重启之后的检索、过了 0 点之后启动时整理会开始。
- **Agent。** 在验证用的文件夹里，确认从对话委托的任务在开始、继续、合入时每次都出现确认界面，取消则什么都不开始。还要确认 worktree 差异的合入和丢弃。有未提交的改动或冲突时，确认合入停止。确认合入目标的分支名称出现在卡片的差异上方和从对话合入的确认界面上，看过差异之后打开从同一提交创建的另一分支时合入停止。在子模块里提交过的任务无法合入，确认卡片上显示 worktree 的位置，卡片的“捨てる”会弹出确认界面。HEAD 不指向分支时（bisect 途中之类），确认合入停止，任务的分支和 worktree 留下。在 Agent 的界面上，确认所选任务的日志下方列出产物，按下后打开文件卡片。确认产物的 HTML 作为页面打开，同一文件夹的 CSS 和图片生效，页面里的外部链接不在应用内打开。在从 Finder 打开的应用里，确认用 npm 安装的 codex 的任务能运行，停止任务时 Agent 正在运行命令也停止。从终端用 `open -a` 打开的应用会继承终端的 PATH，所以不能用来做这个确认。
- **常驻。** 确认通信和语音的服务恢复时能恢复、用 30 分钟时的 CPU 和内存、从睡眠恢复、托盘和唤出的快捷键（macOS 是 ⌥Space，Windows 是 Alt+Shift+Space）、完全退出之后不留下子进程。
- **日历。** 确认登录 Google、在同意界面取消权限中的一项时不会登录成功、登出后 ASIST 从 Google 账号的“サードパーティ製のアプリとサービス”中消失、要显示的日历和保存目标的选择、重启之后设置和登录状态仍保留。用验证用的日程试添加、修改、删除，确认取消时什么都不变、也会反映到 Google 日历。确认月视图里跨日的全天日程成为 1 条带、周视图里重叠的日程左右分开、当前时刻的线出现在今天。
- **邮件。** 用 Gmail 的应用专用密码添加账号，确认收件箱的收取、IDLE 下的新邮件、重启之后的连接。用验证用的邮件试“未読メールある?”“読んで”“返信して”，确认回复变成草稿卡片、“捨てる”不发送、“送信”送到对方并留在已发送、归档和垃圾箱经过确认界面也反映到服务器。在邮件界面上确认邮箱的切换、搜索、邮件串的显示、从撰写发送和保存草稿、批量标为已读的操作。
- **启动失败时的更新。** 装入带这个机制的版本，并在 Release 上有比它更新的版本时，在 Mac 和 Windows 上都确认。试两种失败：读不到设置的失败，以及服务准备的失败。读不到设置的失败，是把设置和数据文件夹的 `settings.json` 移到别处，放上内容读不了的文件来引起。服务准备的失败，是把同一文件夹的 `jobs.json` 移到别处，放上内容读不了的文件来引起。记忆的整理读取任务历史的地方会让启动停止。两种情况下都要确认：出现启动失败期间以及关闭之后，ASIST 的窗口和托盘图标都不出现，麦克风也不启用。有邮件账号时，还要确认获取期间不出现新邮件通知。确认关闭时出现正在更新的通知。确认在获取期间，Mac 上从 Finder 或 Dock、Windows 上从开始菜单再次打开 ASIST 时，同样的通知会再次出现。在 Mac 上，获取期间 Dock 里出现 ASIST 的图标。最后确认替换成新版本后 ASIST 打开。在 Windows 上，确认读不到设置的失败也会出现通知。弄坏的东西还保持原样，所以新版本上也会出现启动失败，关闭就直接退出。最后把移走的东西移回去。
- **麦克风的许可。** 把签名后的应用重启 2 次，确认许可保留、原生麦克风能工作。语音识别、朗读、Agent 的 CLI 缺失的构成下，也要确认显示设置方法或失败理由。
- **重新加载页面。** 打开设置里的“起動時にマイクをオンにする”，确认启动时麦克风为开。在 macOS 上，麦克风开着时用 ⌘R 重新加载页面，确认新页面即使这个设置为开也从麦克风关闭的状态开始，菜单栏的麦克风显示也消失。选择 Gemini Live 时，确认那个会话也关闭。用 CDP 的 `Page.crash` 让 renderer 崩溃时，确认页面重新加载，麦克风保持关闭也能使用。
- **live 的引擎。** 选择 Gemini Live，确认开始说话时会话打开、回答用 provider 的声音响、天气和日程会出卡片、朗读中搭话会停止、能回答文字的输入、对话停止时按设置的秒数关闭并用下一个声音重新打开、把麦克风设为 OFF 时会话关闭、对话日志里留下转写。把设置的“会話”的“モデル”换成 Gemini 3.8 Live (extended thinking)，确认会话打开、回答能响。API 要求的设置因模型而异，一个模型能打开，不代表另一个也能打开。

在 Windows 上，用安装程序装入的应用确认上面的项目。因为存在 Windows 上没有的行为，以及只有 Windows 才有的行为，所以下面的项目也要确认。

- **安装。** 确认未签名的安装程序会弹出 SmartScreen 的警告、从“詳細情報”选“実行”能装进去、不请求管理员权限就装到 `%LOCALAPPDATA%\Programs\asist`、能从开始菜单启动。
- **窗口和托盘。** 确认标题栏的按钮叠在顶栏的右侧、换主题时按钮的颜色也变、最大化和还原不会让 dock 的高度崩掉。确认托盘图标出现，能从菜单显示、切换麦克风、退出，任务完成等通知出现，按下后应用来到前面。
- **麦克风。** 在 Windows 的设置里拒绝桌面应用的麦克风时，确认会显示理由和打开设置的按钮。用启动日志里 `native-mic: check:` 的行看该机器的麦克风回声消除是否显示为有效。有效时，确认一开麦克风就显示 `asist-mic: ready:` 且声音变成文字、麦克风开着期间其他声音不会变小、更换默认的麦克风或扬声器时采集会重新创建。无效时，确认从一开始就用 getUserMedia 采集，设置的“声”里不出现回声消除的开关。无论哪条路径，都用扬声器放 VOICEVOX、live 的引擎、系统语音，确认朗读中不会重新拾取自己的声音。
- **网络共享。** 在设置的“見せてよいフォルダ”里写 `\\サーバー\共有\フォルダ`，确认那里的 PDF 和图片出现在文件卡片里。用映射了同一文件夹的驱动器（`Z:\フォルダ`）书写时也要确认。
- **听取和 Qwen3-TTS。** 在有独立 GPU 的机器上，确认初次设置中 Qwen3-ASR 和 Qwen3-TTS 能通过 Vulkan 准备好、能用声音对话，以及在设置里能切换到 Qwen3-ASR 的 1.7B 和 0.6B，以及能听取对话语言的 FastConformer 的模型。在任务管理器的“パフォーマンス”里确认收在 GPU 的专用内存内。在没有独立 GPU 的机器上，确认会给出理由，能选择浏览器中的 Whisper 或 live 的引擎，Qwen3-TTS 不出现在可选项里。
- **Agent。** 在 claude 和 codex 两者上确认任务的开始、停止，以及退出应用时 CLI 和它的子进程都不留下。codex 只能在 Windows 的 elevated 的 sandbox 里运行，所以在还没准备好的机器上确认任务不会开始、会显示准备方法，在终端里准备好之后能合入会写入的任务。确认合入的文件的换行符保持仓库原来的换行符。

## 阅读实现

| 位置 | 作用 |
|---|---|
| [src/main/](../../src/main/) | 外部的服务、保存、语音的 worker、Agent 的执行 |
| [src/main/services/brain/](../../src/main/services/brain/) | 对话的循环、提示词、工具、历史、说话对象的组装 |
| [src/main/services/llm/](../../src/main/services/llm/) | 对话模型的调用。每个 provider 的 adapter 转换不依赖 provider 的对话类型（[conversation.ts](../../src/shared/conversation.ts)）与各自 API 的形式。 |
| [src/main/services/live/](../../src/main/services/live/) | Gemini Live 的引擎 |
| [src/main/services/weather/](../../src/main/services/weather/) | 天气。日本气象厅和 Open-Meteo |
| [src/preload/](../../src/preload/) | main 与界面之间的 API。契约在 [src/shared/ipc.ts](../../src/shared/ipc.ts) 里。 |
| [src/renderer/](../../src/renderer/) | React 的界面、麦克风的输入、语音的播放、卡片 |
| [src/shared/](../../src/shared/) | 进程之间共享的类型和逻辑。界面文案的字典在 [i18n/messages/](../../src/shared/i18n/messages/) 里。 |
| [resources/](../../resources/) | 语音和检索的 worker、[整理记忆的 Agent 的步骤](../../resources/skills/memory-curation/SKILL.md)（非日语对话中是[英语的步骤](../../resources/skills/memory-curation-en/SKILL.md)），以及每种对话语言的[记忆模板](../../resources/skills/memory-templates/) |
| [tests/](../../tests/) | Vitest 的测试 |

增加卡片时，在[清单](../../src/shared/panel-catalog.ts)里定义 schema，在 [builtin/](../../src/renderer/src/panels/builtin/) 里实现显示，再注册到 [registry.tsx](../../src/renderer/src/panels/registry.tsx)。传给对话模型的卡片用工具由 schema 生成。

## 只在开发时使用的模型

| 用途 | 模型 |
|---|---|
| 随应用打包的 live 的声音示例 | Google 的 `gemini-2.5-flash-preview-tts` |
| 图标、天气的风景和天空状态的图片 | 图像生成的模型。提示词在 `resources/artwork/` 和 `src/renderer/src/assets/weather/` 里。 |
