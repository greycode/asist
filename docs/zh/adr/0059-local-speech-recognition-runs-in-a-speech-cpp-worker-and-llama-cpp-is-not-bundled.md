# 本地语音识别在与朗读相同的 speech.cpp worker 里运行，不随应用打包 llama.cpp

本地语音识别的 Qwen3-ASR（1.7B 和 0.6B）与朗读一样，在随应用打包的 speech.cpp 的 `speech worker` 里运行。worker 只运行 1 个模型，所以语音识别的 worker 与朗读的 worker 分别启动，对两者的请求都用同一个 worker 协议的客户端发送。模型分别获取 `sakasegawa/Qwen3-ASR-1.7B-GGUF` 和 `sakasegawa/Qwen3-ASR-0.6B-GGUF` 的各 1 个 GGUF 文件，固定 revision 和 sha256。对话的语言以 BCP 47 的 tag（如 `ja`）传给 speech.cpp 的 `language`，与官方实现相同的答案开头（`language Japanese<asr_text>`）由 speech.cpp 写出。不随应用打包 llama.cpp 的 llama-server。用 llama-server 运行，就会多出一套与 speech.cpp 不同的执行环境。需要 ggml-org 的模型和 mmproj 这 2 个文件、回环端口及其密钥、Mac 上的监视（ADR 0056），上下文是 4096 token，长于约 270 秒的发言失败了。llama.cpp 的 Qwen3-ASR 也与官方实现不同，提示词里没有 system 的发言，log-mel 的帧多 1 个，对最后不足 1 秒的音频还会生成用零填充的那部分语音 token。speech.cpp 用官方提示词写出与官方相同的句子，如下面的实测所示，解码速度和转写精度与 llama.cpp 没有差别。

## 未采纳的方案

- **语音识别继续用 llama.cpp 的 llama-server 运行。** 上面说的另一套执行环境和 270 秒的上限会留下，写出与官方实现不同的句子的发言也会留下。解码速度和转写精度如下面的实测，与 speech.cpp 没有差别。

## 实测

- 2026-10-07，Apple M5（32 GB）、Metal、Q8_0。让 speech.cpp 0.7.1 与 ASIST 当时打包的 llama.cpp b11246 交替听同一批 FLEURS 发言，取 5 次（1.7B 为 3 次）的中位数。语言是指定的。

  | 发言 | 0.6B、speech.cpp | 0.6B、llama.cpp | 1.7B、speech.cpp | 1.7B、llama.cpp |
  |---|---|---|---|---|
  | 日语、6.36 秒 | 0.17 秒 | 0.16 秒 | 0.38 秒 | 0.44 秒 |
  | 日语、10.50 秒 | 0.25 秒 | 0.23 秒 | 0.59 秒 | 0.58 秒 |
  | 德语、11.16 秒 | 0.32 秒 | 0.30 秒 | 0.74 秒 | 0.71 秒 |
  | 英语、23.64 秒 | 0.59 秒 | 0.51 秒 | 1.36 秒 | 1.20 秒 |
  | 日语、25.50 秒 | 0.80 秒 | 0.75 秒 | 1.89 秒 | 1.76 秒 |

  解码方面，0.6B 在 1 秒里 146～158 token（llama.cpp 为 147～157），1.7B 为 58～63 token（同样为 58～62）。剩下的差别在编码器和提示词的读取上，llama.cpp 用 Metal 4 的 tensor API 计算矩阵乘法，speech.cpp 因为 ggml 内核的缺陷没有用它。同样的 20 个请求里，llama.cpp 在 0.6B 上有 6 个、在 1.7B 上有 4 个请求写出了与官方不同的句子。
- 同一天、同一台 Mac 上，从 ASIST 的语音识别客户端让 speech.cpp 0.7.1 的 worker 转写 FLEURS 的 7 段日语和英语录音，并指定语言，两种大小除了几乎无声的 0.6 秒录音之外，都与官方句子相同。几乎无声的录音在四舍五入成传给 worker 的 16 位值之后，最大振幅只有 10 级左右，两种大小都写出了与官方不同的句子。不四舍五入直接传，1.7B 写出了与官方相同的句子。耗时测了 3 次，1.7B 对 10.50 秒的发言为 0.62～0.74 秒，对 25.50 秒的发言为 2.1～2.6 秒，0.6B 为 0.27～0.31 秒和 0.87～1.01 秒。在准备完成之前，文件处于 OS 缓存中的状态下 1.7B 为 0.56～0.67 秒，0.6B 为 0.37～0.42 秒；内存在转写完 25.50 秒的发言之后，1.7B 为 2.5 GB，0.6B 为 1.1 GB（2026-09-29 的 llama-server 为 3.4 GB 和 1.8 GB）。
- 2026-10-07，RTX 2080（8 GB）的 Windows 11、Vulkan。把 Common Voice 8.0 的 4,483 段日语片段的语言指定为 `ja`，让 speech.cpp 0.7.0 和 llama.cpp b11246 转写（speech-bench 的 asr-runtimes-on-common-voice-8-ja-2026-10-07）。在允许认可写法的情况下，字符错误率 1.7B 为 4.52%（llama.cpp 为 4.56%），0.6B 为 6.88%（同样为 6.88%）。等待时间的中位数 1.7B 为 0.203 秒（0.192 秒），0.6B 为 0.126 秒（0.121 秒），加载 1.7B 为 2.2 秒（3.2 秒），0.6B 为 2.3 秒（4.2 秒）。同一块 RTX 2080 上，speech.cpp 0.7.1 比 0.7.0 转写 FLEURS 的 5 段发言快 13～25%。

## 已知限制

- 1 个请求一次最多能听 1200 秒，比这更长的音频由 speech.cpp 分段听。1 个部分达到 4096 token 时，就返回到那里为止的文本。
- 中途的转写每次都把最后 5 秒作为新的请求发送。speech.cpp 的 worker 没有读取正在持续送入的音频中间部分的请求。
- speech.cpp 是为 ASIST 做的，也会自行跟进 Qwen3-ASR 的新版本。
