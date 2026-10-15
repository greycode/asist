# Mac 的应用用 Developer ID 签名并公证、从这台 Mac 放到 Release，Windows 的应用不签名、由 GitHub Actions 放到同一个 Release，两者的更新都在下次退出时装入

要分发的 Mac 应用在这台 Mac 上执行 `npm run release` 构建，用 Developer ID Application 签名并公证，放到公开仓库的 GitHub Release 里。内置 git 的源码也放进同一个 Release。应用用 electron-updater 确认 Release，在后台获取新版本，下次退出时替换。只有用户在“このアプリについて”里按“今すぐ再起動”时，以及启动失败时，才立即替换。已发布的版本出问题时，不删除 Release，也不回到前一个版本，而是提高版本号，发布修好的版本。Electron 准备好之后在启动过程中失败的应用会报出错误，被关闭时与其他退出一样结束，并停掉那次启动里已经开始运行的一切。然后，重新做一次只做更新的启动。这次启动不开窗口也不开服务，确认新版本，有就获取并装入，再打开新版本。获取期间用通知告知。没有就原样退出。这是为了让使用每次启动都失败的版本的用户也能收到修复后的版本。macOS 把麦克风的许可与签名绑定，Squirrel.Mac 只装入满足与当前版本相同签名要求的版本。因此，从交给别人的第一个构建起就用同一个 Developer ID 签名。之后再改证书，用户的许可会失效，自动更新也收不到。在对话或 Agent 任务运行途中擅自重启，说话途中的语音和手头的工作就会中断。electron-updater 只装入版本号比当前版本更新的版本，所以没有回到前一个版本的办法。

Windows 的安装程序在 GitHub Actions 的 windows-latest 上构建，与 Mac 的文件放在同一个 Release。`npm run release` 先用 Mac 的文件创建草稿 Release，Windows 的工作流把安装程序、latest.yml、内置的 Git for Windows 源码放进这个草稿，然后发布 Release。Windows 的安装程序没有签名，所以没有理由把私钥留在手边。

Windows 的应用也在没有签名的状态下，像 Mac 一样自行更新。获取到的版本在下次退出时由安装程序装入，不显示界面。按“今すぐ再起動”时，立即装入，然后再打开一次 ASIST。在没有签名的期间，能用来信任更新的依据只有两点：从 GitHub 经 HTTPS 获取，以及与 latest.yml 里写的 sha512 一致。latest.yml 也放在同一个 Release 里，所以 GitHub 账号或能写入 Release 的权限一旦被夺走，就能向所有 Windows 用户分发不良的更新。Mac 上 Squirrel.Mac 要求同一个 Developer ID 的签名，所以仅仅夺走 GitHub 账号并不能分发。接受这一风险，让 Windows 的自动更新先于签名开始。

## 未采纳的方案

- **在 CI 里构建 Mac 的应用并公证。** 这样一来，证书的私钥和 Apple ID 的凭据就要放进 GitHub 的 Secrets。分发量不大，所以选择了不从这台 Mac 的钥匙串里拿出来。
- **获取到就立即重启装入。** 正在说话时或任务运行中，应用就会关闭。
- **Windows 的安装程序也在这台 Mac 上构建。** Agent 的 launcher 和麦克风辅助程序要用 MSVC 构建，git 和 uv 也要用构建机器操作系统的版本，所以在 Mac 上做不出 Windows 的应用。
- **启动失败时，不确认更新就退出。** 使用每次启动都失败的版本的用户就收不到修复后的版本。用户就得自己到 Release 页面找到新版本再重装。
- **在拿到签名之前，Windows 的应用不自行更新。** 用户得自己发现新版本，重新运行安装程序。修复后的版本能不能送到，取决于用户会不会去看 Release。

## 已知限制

- 本地执行 `dist:mac` 的构建不生成 dmg 也不生成 zip，所以不带自动更新的配置，不会从 Release 更新自己。本地执行 `dist:win` 的安装程序也只从发布工作流接收 GitHub 的公开配置，同样不会更新自己。
- 在读取 main 的模块的途中，或在 Electron 准备好之前失败时，Electron 会报出自己的错误，不确认更新。使用这种启动失败持续出现的版本的用户，就得从 Release 重新安装。
- 以个人身份加入 Apple Developer Program，所以应用的签名里会出现加入者的姓名。
- 把以前用 Apple Development 签名安装的应用换成用 Developer ID 签名的版本时，麦克风的许可要重新做一次。
- electron-updater 只有在应用里的 app-update.yml 有发行方名称时，才验证 Windows 更新的签名。因此，即使开始给 Windows 的安装程序签名，使用无签名版本的用户装入下一次更新时也不会验证签名。
