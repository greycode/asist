# 日历在 Mac 和 Windows 上都只用 Google Calendar API 读写，不使用 macOS 的 EventKit

ASIST 的日历是为使用 Google 日历的人做的。日程在 Mac 和 Windows 上都通过 Google Calendar API，从用户登录的 1 个 Google 账号读写，不使用 macOS 的 EventKit。Windows 上没有相当于 EventKit 的 OS 机制，要在 Windows 上提供日历就只能用 Google Calendar API 做，Mac 也用同一套实现，界面、文案、给模型的说明、测试就不必按 OS 分开。用 EventKit 的话，macOS 的许可会与签名绑定，从 shell 启动的应用上许可不生效，要恢复许可状态还得用 `tccutil`，许可的处理也很费事。Mac 的日历 ID 无法与 Google 的日历 ID 对应，所以在把设置升到版本 6 时丢弃选中的日历，关闭集成。用户登录 Google，重新选择日历。

## 未采纳的方案

- **Mac 继续用 EventKit，只有 Windows 用 Google Calendar API。** iCloud、Exchange 的日历在 Mac 上也能读，但读写的实现会变成 2 套，许可状态、同步的说明、错误的文案、界面的区分都要按 OS 各自维护下去。

## 已知限制

- 不在 Google 账号里的日历（iCloud、Exchange 等）不会出现在 ASIST 里。
- `calendar.events` 是 Google 划为敏感的权限，是通过 Google 的审核在用的。修改同意界面的设置，或者追加权限，都要重做审核。
