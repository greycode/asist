# Windows 上，Agent 的 CLI 用带 Job Object 的启动程序来运行

Windows 上，ASIST 不直接启动 Agent 的 CLI，而是从内置的 `asist-agent-launcher.exe` 启动。launcher 创建以 token 命名的 Job Object，从标准输入收到 `start` 之后，从一开始就把 CLI 创建在 Job 里。CLI 启动的进程即使父子关系断开，也进入同一个 Job。无论 CLI 自己结束、被 `--stop` 停止，还是 ASIST 的进程结束，launcher 都在 Job 空了之后才结束。因此 ASIST 可以把 launcher 结束视为 CLI 及其后代已全部消失。重启之后，从保存的 token 决定 Job 的名字，只停止那个 Job。Windows 没有进程组和发往进程组的信号，只沿着父子关系追溯会漏掉父进程先结束的孙进程，PID 又会被很快复用，所以靠 PID 和时间无法确认是不是自己的进程。macOS 上那套用进程组和环境变量的 token 来确认的办法，则照旧使用。

## 未采纳的方案

- **用 `taskkill /T /F` 停止。** 只能追溯还活着的父子关系，会漏掉自己结束的 CLI 留下的孙进程。
- **从 main 用 FFI 操作 Job Object。** 从 `spawn` 之后到放进 Job 之前的这段时间里，CLI 就能创建出子进程。要防住这一点，终究还是需要启动程序。
- **用 PowerShell 的 `Get-CimInstance Win32_Process` 查。** 启动要花时间，而且无法确认父子关系已断的后代是不是自己的。
- **把 ASIST 和 launcher 用命名管道连起来，用管道传停止指令和 ASIST 的退出。** ASIST 的退出只要等 ASIST 进程的句柄就能知道，停止指令用按名字打开 Job 的 `--stop` 就够了。

## 已知限制

- 因为没有相当于 SIGTERM 的东西，停止 CLI 时不给它留清理的时间。CLI 留在 worktree 里的临时文件和锁文件，会在合入或丢弃时由 git 的检查发现。
- 进入 Job 的，只有 CLI 及其后代自己创建的进程。像任务计划程序、WMI、COM 服务器那样，委托 Windows 的服务或已经在运行的其他进程创建的进程，会在 Job 外被创建，停止时也会留下。
- 用 npm 装的 CLI 只有 `.cmd`，用不了。`.cmd` 只能通过 `cmd.exe` 启动，因为 `cmd.exe` 会按自己的方式解释 ASIST 传入的参数里的引号和括号。
