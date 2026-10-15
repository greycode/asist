# uv 与 git 随应用打包，Python 由随应用打包的 uv 在准备时获取

这台计算机上运行的模型的 Python 环境，由随应用打包的 uv 创建。Python 只用 uv 获取的那一份，版本固定下来，Python 的本体和 uv 的缓存都放在 userData 下。不读用户的 uv 设置和 `UV_` 环境变量。若交给用户手头的 Python 或 uv 来管，用哪个 Python 建出环境会因用户而异，删掉那个 Python 后，MaAI 和语义搜索就不再工作。记忆的历史与编辑任务的 worktree 用随应用打包的 git，不读系统和用户的 git 设置。例外是决定工作树文件换行符和符号链接处理方式的 3 项设置，只有这几项取与用户的 git 相同的值。在没有 Command Line Tools 的 Mac 上，`/usr/bin/git` 只会提示安装然后失败，而记忆每次启动都要用 git。一旦读用户的设置，记忆的提交就会因 `commit.gpgsign` 或 git-lfs 的过滤器而失败。

Windows 上没有系统安装的 git，所以也一样随应用打包。在 Windows 上从源码构建 git 需要 MSYS2 环境，Git for Windows 以适合放进应用的形态（MinGit）分发这样的构建，所以也把这个形态随应用打包。uv 在 Windows 上也一样随应用打包。

## 未采纳的方案

- **语音识别使用用户通过 Homebrew 安装的 whisper.cpp 的 `whisper-server`。** 每个用户装的版本和模型都不同，没装的机器上无法运行。
- **Python 也随应用打包。** 应用里的可执行文件要用 hardened runtime 和应用的 Team ID 签名，库的验证会拦下之后才装入的 torch 等扩展模块，不让加载。包反正都要从网络获取，只打包 Python 本体得到的东西很少。

## 已知限制

- 创建编辑任务的 worktree 时也不读用户的 git 设置，所以由 git-lfs 管理的文件在 worktree 里仍是指针。
- 与用户自己用的 uv 之间，Python 和包的缓存都不共享。
- git 是 GPL-2.0，所以分发的应用里要放入许可证正文，并从“このアプリについて”给出指向原始源码的链接。
- MinGit 里除了 git，还装着 MSYS2 的运行时库和 sh。在 Windows 上分发时，要提供源码的范围也包括 Git for Windows 同一 tag 和 MSYS2 各包的源码。
- macOS 和 Windows 上，git 的版本有时会略有差异。
