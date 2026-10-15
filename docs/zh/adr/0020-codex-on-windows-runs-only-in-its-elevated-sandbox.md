# Windows 的 codex 始终在 elevated 的 sandbox 中运行，准备没做好就不运行

在 Windows 上，给 codex 的所有任务（只读的任务、写入的任务、整理记忆的任务以及它们各自的后续）都加上 `-c windows.sandbox="elevated"`。Windows 的 codex 如果在 `windows.sandbox` 里不指定 sandbox，就不会把任何东西关进 sandbox，直接运行。ASIST 用 `--ignore-user-config` 运行 codex，好让任务的运行方式不随用户的模型偏好改变，所以即使用户在自己的配置里选了 sandbox，该配置也到不了任务里。不指定的话，写入的任务会写到文件夹外，也会连上网络。在 macOS 上 codex 的 sandbox 守住的边界，也就是写入的任务要写到文件夹外就必须经过自动审查，以及整理记忆的任务写不到 worktree 外、也连不上网络，这些在 Windows 上都会消失。使用 elevated 的 sandbox 需要一次性的准备来创建一个本地用户，这项准备需要管理员权限。在准备还没完成的机器上，按与找不到 codex 时相同的方式处理，不开始任务，并提示先在终端里启动一次 codex 来准备 Windows 的 sandbox。这样处理，是因为由定时器启动的整理记忆任务不能在没有人在场的时候要求管理员权限。准备是否完成，取决于 codex 在准备后留下的文件（codex 主目录下的 `.sandbox\setup_marker.json`）是否存在。claude 在 Windows 上也靠与 macOS 相同的参数关住，所以不改。

## 未采纳的方案

- **使用 unelevated 的 sandbox。** 不需要管理员权限的准备，但在验证机上所有命令都启动失败。
- **不确认准备是否完成就交给 codex。** 准备还没完成的 `codex exec` 会怎么运行，还没有确认。它有可能在任务中途要求管理员权限。
- **放弃 `--ignore-user-config`，改用用户的 `[windows] sandbox` 配置。** 用户的模型偏好也会影响到任务。另外，没有写这项配置的用户，其任务会在没有被关住的状态下运行。

## 实测

2026-09-27，在 Windows 11 的验证机上，用与 ASIST 相同的参数，从 %APPDATA% 下的文件夹运行 codex-cli 0.157.1。让 Agent 做 5 件事：在文件夹内写入、在文件夹外写入、获取 https://example.com、读取文件夹外的文件、在文件夹内运行 node 脚本。

- 不指定 sandbox 运行写入的任务（`--approve-for-me`）时，5 件都能做到。
- 给写入的任务加上 elevated 的 sandbox 后，向文件夹外写入被拒，获取因为连不上而失败，剩下的 3 件都能做到。
- 不指定 sandbox 运行整理记忆的参数时，包括在文件夹内写入在内，所有命令都以“blocked by policy”被拒绝。
- 给整理记忆的参数加上 elevated 的 sandbox 后，向文件夹外写入被拒，获取因为连不上而失败，剩下的 3 件都能做到。
- 在 unelevated 的 sandbox 下，所有命令都以 `CreateProcessAsUserW failed: 5`（拒绝访问）未能启动。

## 已知限制

- elevated 的 sandbox 和 macOS 的 workspace-write 一样，任何位置的文件都能读。整理记忆的任务也能读取文件夹外的 API 密钥文件。网络是关闭的。
- 整理记忆的任务可以改写自己 worktree 里的验证用脚本。在 codex 中该脚本也在同一个 sandbox 里运行，所以即使改写，它能做的事也不会变多。
- 准备是否完成由 codex 的文件判断，所以 codex 一旦改变该文件的位置或名字，即使准备已经完成，也无法开始 codex 的任务。
- ASIST 不做准备。用户需要先在终端里启动一次 codex。
