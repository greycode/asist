# Windows 应用不在 Microsoft Store 上分发，最初不带签名分发

Windows 应用通过 GitHub 的 Release 以 NSIS 安装程序分发。不在 Microsoft Store（MSIX）上分发。最初不签名，先申请 SignPath Foundation 面向开源项目的免费签名，申请通过后就签名。没有签名的应用，用户需要从 SmartScreen 的警告界面点“詳細情報”，再点“実行”继续，才能装上，因此这一步骤要写进文档。在没有签名的期间，应用仍会自己从 GitHub 的 Release 更新。那时的风险，写在处理更新的记录里。

## 未采纳的方案

- **以 MSIX 发布到 Microsoft Store。** Store 会重新签名，所以不需要证书，也不会出现警告。但是在 2026-09-27，在验证机上把 ASIST 做成 MSIX 来运行，Windows 会把应用对 AppData 的写入重定向到包专用的位置，uv 因此无法准备好 Python 而失败。codex 的 elevated 的 sandbox 会以另一个用户运行命令，出于同样的原因，会看不到 ASIST 的文件。停掉重定向就能运行，但为此需要受限制的权限 `unvirtualizedResources`，Microsoft 主要把它用于游戏。个人账号能否获准，只有提交审核才知道。此外，Store 的自动更新会连正在运行的 Agent 任务一起把应用关掉。
- **以 MSI/EXE 的形式发布到 Store。** 可以使用现在的 NSIS 安装程序，但和 Store 之外一样需要自己的证书，上架 Store 的好处不多。
- **Azure Artifact Signing。** 个人开发者只能在美国和加拿大使用。
- **EV 证书。** 从 2024 年起，和 OV 证书一样，SmartScreen 的警告不再马上消失。

## 已知限制

- 没有签名的期间，SmartScreen 的信誉积累不起来，每发布一个版本，打开下载的安装程序的人都会看到警告。在启用 Smart App Control 的机器上，则装不上。
- SignPath Foundation 的签名，发布者会显示为“SignPath Foundation”。能签名的，只有由 CI 从公开仓库自动构建出来的东西。
