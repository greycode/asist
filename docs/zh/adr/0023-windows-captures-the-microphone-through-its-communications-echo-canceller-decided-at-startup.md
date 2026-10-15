# 在 Windows 上，麦克风经由 Windows 的通话用回声消除采集，是否可用只在启动时决定一次

在 Windows 上，内置的 `asist-mic.exe` 以 WASAPI 的通话用（Communications）分类打开默认麦克风来采集。这样，Windows 会施加通话音效处理（麦克风驱动程序自带的，没有则用 Windows 11 24H2 以后的 Voice Clarity），该回声消除以送到默认扬声器的声音为参照。因此，不仅能抵消 Chromium 播放的朗读，还能抵消用 Windows 语音合成朗读的系统声音以及其他应用的声音。getUserMedia 的 `echoCancellation: true` 只能抵消 Chromium 自身播放的声音，所以在 Windows 上也准备了与 Mac 的 voice processing 作用相同的路径。与辅助程序的交互（48kHz、单声道、float32、退出码）与 Mac 相同，main 与 renderer 不按 OS 区分。是否使用该路径，由 main 在启动时运行 1 次 `asist-mic.exe --check` 来决定：只有在该流的音频效果列表中，Windows 把回声消除回答为“有効”时才使用。把结果作为 capabilities 的 `nativeMic`。回声消除不在列表中或无效的机器、没有麦克风的机器、确认本身就失败的机器，从一开始就用 getUserMedia 采集。

## 未采纳的方案

- **使用 Voice Capture DSP（`CLSID_CWMAudioAEC`）。** 输出限于 16kHz，无法交给接收 48kHz 帧的 DeepFilterNet。作为参照的输出侧没有声音在响就不工作，所以还必须一直播放静音的流。
- **总是运行辅助程序，一旦发现回声消除没有生效就切到 getUserMedia。** 这样每次打开麦克风都要先等失败再切换，也违反“不为失败添加替代行为（fallback）”这一方针。该机器上能否使用，要在启动时确认一次，并作为该 OS 与该机器的路径定下来。

## 已知限制

- 决定只在启动时做一次。启动之后即使换成带回声消除的麦克风，也要重新启动应用才会用辅助程序。
- 运行过程中辅助程序停止时，比如换到的默认麦克风上回声消除无效时，与 macOS 一样由 renderer 切换到 getUserMedia。
- 启动时的确认只打开默认麦克风，不开始采集，但窗口出现最多延迟 5 秒。即使在设置里关掉回声消除，每次启动仍会确认。
- 即使 Windows 回答“有効”，能抵消多少从扬声器播放的朗读，还没有测过。与 Chromium 的 `echoCancellation: "all"`（在 Windows 11 上参照整个系统的输出）相比，需要决定把哪一个作为 Windows 的默认。
