# API 密钥加密保存，包括 Agent 的 CLI 在内，不传给任何子进程

在设置界面保存的 API 密钥，与邮箱密码一样用 safeStorage 加密后保存，不写入明文文件。加密不可用时就不保存，直接报错。密钥直接交给 SDK 的客户端，不放进 main 的环境变量。启动子进程（codex 和 claude、Python 的 worker、语音引擎、原生辅助程序、git）时，也从环境变量中移除 provider 的密钥。Agent 的任务一旦受到提示词注入，就能以用户的权限执行命令，因此环境变量里的密钥可能被原样发到外部。ChatGPT 的登录 token 同样加密保存，不传给任何子进程。codex 和 claude 各自用其 CLI 一侧配置的认证运行。claude 则无论是用 Claude 的套餐还是 API 密钥，都由用户按 claude 自身的步骤登录。ASIST 里有 ChatGPT 的登录，但不做 Claude 的登录，也不读取 claude 的认证信息。这是因为 Anthropic 不允许其他应用提供 Claude.ai 的登录，也不允许收集或保存其认证信息和会话的 token。

## 未采纳的方案

- **照旧以明文保存到 `userData/.env`。** 同一用户下运行的任何进程都能读取。不迁移旧的 `.env`，密钥要由用户在设置界面重新输入。

## 已知限制

- 在 macOS 上，开发版和安装后的应用加密密钥不同，所以在一方保存的密钥，在另一方读不到。开发时优先使用运行目录下 `.env` 里的密钥。
- Windows 的 safeStorage 使用 DPAPI，把加密密钥以该用户下任何人都能解开的形式放在 userData（`%APPDATA%\asist`）的 `Local State` 里。因此，同一用户下运行的任何进程都能取出保存的密钥。在 Windows 上，无法阻止受到提示词注入的任务自己去读取密钥文件。开发版和安装后的应用使用同一个 userData，所以在一方保存的密钥在另一方也能读到。已确认开发版加密的字符串能由另一个 Electron 进程从同一个 userData 解密（Windows 11，2026-09-27）。
