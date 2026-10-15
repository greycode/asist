# 把用 ChatGPT 套餐对话的路径做成与 OpenAI 不同的提供商“ChatGPT”，不切换到 API 密钥

用 ChatGPT 的 Plus 和 Pro 套餐对话的路径，不做成 OpenAI 提供商内部的一个切换，而做成名为“ChatGPT”的独立提供商，在界面上排在 OpenAI 的正下方。套餐可用的路径与 API 密钥的路径，在请求形式（把 store 设为 false 并用流接收、把函数工具归到 namespace 里、不发送 max_output_tokens）和计费方式（没有按 token 计的价格，包含在套餐的用量里）上都不同。合并成一个提供商的话，模型的选择、计费页面、错误文案，每一处都会加进判断是哪条路径的条件分支。登录按照 OpenAI 为在用户计算机上运行的开源应用给出的步骤（Sign in with ChatGPT），在 ASIST 内部实现。达到用量上限时，或套餐不在范围内时，即使有 API 密钥也不自动切到那边。因为会在用户不知道的情况下产生 API 的费用，OpenAI 也要求应用不要悄悄转到别的支付方式。请求通过 Responses API 的 WebSocket 发送，用完的连接留到下次请求再用。

## 未采纳的方案

- **在 OpenAI 这一行里切换 API 密钥和 ChatGPT 套餐。** 按所选的方式来，就要分别处理模型列表、请求的形式、计费方式，上面的条件分支就会全部进到 OpenAI 提供商内部。
- **随应用打包 OpenAI 的官方 DevKit。** 它是非商业许可，装不进以 MIT 许可分发的 ASIST。
- **用 HTTP 发送。** 套餐的路径每次请求等待响应开始的时间都长，如下面的实测所示，比用 WebSocket 复用连接更慢。

## 实测

在 2026-10-05，用这台 Mac 和一个 ChatGPT 账号测量。

- 用一次性的步骤附上约 2,700 token 的指令，把思考深度设为 low，到第一个字到达为止各测 6 次，取中位数。套餐的 gpt-5.6-terra 用 HTTP 是 2190 ms，用 WebSocket 是 1567 ms。gpt-5.6-luna 用 HTTP 是 2785 ms，用 WebSocket 是 1065 ms。API 密钥的 gpt-5.6-terra 用 HTTP 是 845 ms，用 WebSocket 是 658 ms。
- 在开发版的应用里发送不用工具的短对话，到第一句话为止，套餐的 gpt-5.6-terra 是 1.9～3.1 秒，同一个模型用 API 密钥是 1.2～2.3 秒。衔接语的 JSON 用套餐的 gpt-5.6-luna 在 1.6～2.1 秒返回。
- 连续 3 次的对话复用了同一个 WebSocket 连接。

## 已知限制

- 能用套餐的只有 Plus 和 Pro。ID token 里没有套餐的种类，所以 ASIST 无法知道是 Plus 还是 Pro，能不能用要等到请求失败才知道。
- 套餐的路径上，第一句话比同一个模型用 API 密钥时大约慢 1 秒。
- Plus 每 5 小时的用量上限与 ChatGPT 自身及其他应用共享。
- 用套餐发送的对话是否被用于 OpenAI 的学习，OpenAI 的帮助里没有写。
