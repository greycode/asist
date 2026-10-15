# 本地的语音识别与朗读用内置的 speech.cpp 在 GPU 上运行

无论 macOS 还是 Windows，本地语音识别的 Qwen3-ASR（1.7B 与 0.6B）、朗读的 Qwen3-TTS（0.6B 与 1.7B）与 Irodori-TTS，都在自行实现的 speech.cpp worker 中运行 GGUF 模型。speech.cpp 随应用打包，在 Mac 上通过 Metal、在 Windows 上通过 Vulkan 使用 GPU。模型从 Hugging Face 获取，版本与 sha256 都固定。由于不搭建 Python 运行环境，首次准备只需获取模型，也不必为每个 OS 维护另一套推理实现。任何 OS 都用同一模型的同一文件，因此识别质量与声音质量不随 OS 变化。在 Windows 上只对装有独立 GPU 的机器提供，其他机器则给出理由，让用户改用浏览器里的 Whisper 与 live 引擎。

## 未采纳的方案

- **在 Mac 上用 MLX、在 Windows 上用 CUDA 的 torch 运行。** 要为每个 OS 维护另一套实现以及数 GB 的 Python 运行环境。在 Windows 上只限于 NVIDIA 的 GPU、驱动程序 580 以上、RTX 20 以后，而 Qwen3-TTS 在 Windows 上无法运行。
- **用 qwentts.cpp 运行 Qwen3-TTS。** 把按句子分开生成的声音对比着听时，上面带着抖动般的噪声。Qwen3-TTS 的 codec 只看过去的输入，所以 speech.cpp 沿用中途的状态，即使分开生成也输出与一次生成相同的声音波形。
- **随应用打包 llama.cpp 的 CUDA 版本。** CUDA 13 的版本不含 RTX 20 的机器码，当驱动程序比该版本所支持的 CUDA 还新时，启动会失败。CUDA 12 的版本能跑，但首次启动时编译 GPU 代码花了 28 秒。Vulkan 不论驱动程序的版本，按下面的实测速度也够。
- **在 Windows 的集成 GPU 上也运行。** 没有测过。它与 CPU 共用内存，所以要先确认同时运行语音识别与朗读来不来得及，再作决定。
- **为内存少的 Mac 保留 Whisper large-v3-turbo。** Qwen3-ASR 0.6B 用不到 2 GB 就能跑，所以把选项统一成 Qwen3-ASR 一种。
- **用 llama.cpp 的 llama-server 运行语音识别。** 起初是这么做的，但在 ADR 0059 中替换成了 speech.cpp。这样就会持有一套与朗读不同的运行环境，超过约 270 秒的发言会失败，还会出现写出的句子与官方实现不同的发言。
- **在 Windows 上，用 CPU 运行 sherpa-onnx 的日语模型（ReazonSpeech 的 zipformer）。** 不需要 GPU，但会变成与 Mac 不同的模型，识别质量要另行调整。

## 实测

2026-09-29，语音识别用 llama.cpp b11246，朗读用 qwen3-tts-ggml v0.1.1（speech.cpp 以前的名字），权重为 Q8_0（Qwen3-TTS 的 codec 是 F16）。Mac 是 M5（32 GB），Windows 是 RTX 2080（8 GB，驱动程序 591.86）。语音识别的行是 ADR 0059 中替换掉的 llama.cpp 的值，speech.cpp 的值在 ADR 0059。

| | M5、Metal | RTX 2080、Vulkan |
|---|---|---|
| Qwen3-ASR 1.7B、17 秒的发言 | 1.40 秒、3.4 GB | 0.60～0.68 秒、VRAM 3.2 GB |
| Qwen3-ASR 0.6B、同一发言 | 0.58 秒、1.8 GB | 0.36～0.40 秒、VRAM 1.9 GB |
| Qwen3-TTS 0.6B、到第一个声音、生成 1 秒声音的时间 | 0.05 秒、0.35 秒、2.3 GB | 0.07 秒、0.31 秒、VRAM 1.6 GB |
| Qwen3-TTS 1.7B、同上 | 0.08 秒、0.49 秒、3.3 GB | 0.08 秒、0.36 秒、VRAM 2.7 GB |

- 在 8 GB 的 RTX 2080 上，加上桌面使用的 1.9 GB，语音识别 1.7B 与朗读 0.6B 共 6.7 GB，朗读换成 1.7B 则为 7.7 GB。
- Qwen3-ASR 0.6B 把“いいえ”听成“いや”，1.7B 没有听错。
- qwen3-tts-ggml 在 F32 权重下输出了与官方实现相同的声音符号序列。Q8_0 下从中途变成别的序列，但听不出区别。
- Qwen3-TTS 0.6B 有时会在声音之前生成最长 1 秒左右的静音，1.7B 几乎没有。
- 2026-10-01，在 speech.cpp v0.3.0 中，M5 上 Qwen3-TTS 0.6B 到第一个声音为 0.04 秒、生成 1 秒声音的时间为 0.37 秒、内存为 1.8 GB，1.7B 为 0.06 秒、0.47 秒、2.8 GB。装入新版本后的首次启动因为要组装 Metal 内核，到准备完成为止花了 15.6 秒。

## 已知限制

- 没有独立 GPU 的 Windows 机器无法使用本地的语音识别与朗读（Qwen3-TTS 与 Irodori-TTS）。
- 装入新版本的应用或 GPU 驱动程序后的首次启动，Mac 上要组装 Metal 内核、Windows 上要组装 Vulkan 着色器，只有那一次的准备时间会变长。
- 内置的附和语音是用 Qwen3-TTS 0.6B 生成的，用 1.7B 朗读时也播放同一个。
- speech.cpp 是为 ASIST 制作的，对 Qwen3-ASR、Qwen3-TTS、Irodori-TTS 的新版本会自行跟进。
