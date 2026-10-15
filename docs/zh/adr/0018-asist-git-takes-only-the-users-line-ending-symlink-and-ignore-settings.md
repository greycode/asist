# ASIST 的 git 只沿用用户 git 设置中的换行符与符号链接处理，以及忽略文件清单

ASIST 让内置的 git 在既不读系统设置也不读用户设置的环境中运行。但决定工作树文件如何写出、如何读回的 core.autocrlf、core.eol、core.symlinks 这 3 项，以及决定忽略哪些文件的 core.excludesFile，按与用户 git 相同的值运行。Git for Windows 的安装程序默认在系统设置里写入 core.autocrlf=true。因此在用户 clone 的仓库中，文件以 CRLF 存在于磁盘上，以 LF 进入提交。不读这一设置的 git 会认为所有文件的每一行都变了。于是任何仓库都通不过合入前的检查，任务一次也合入不了。另外，在任务 worktree 的提交里，CRLF 本该由用户 git 改成 LF，却原样进入用户的历史。core.symlinks 不同时，仓库的符号链接会在任务的 worktree 中作为普通文件检出。在 Mac 上，通常的做法是通过 ~/.gitconfig 的 core.excludesFile，让所有仓库忽略 Finder 生成的 .DS_Store 等文件。不读这一设置的 git，会把用户 git 不输出任何内容的仓库看成有未跟踪的新文件，于是在任何任务里都以存在未提交的变更为由拒绝合入，还把任务 worktree 中生成的 .DS_Store 加进任务的提交。git 在没有 core.excludesFile 时会去读的 ~/.config/git/ignore，即使不读用户设置，ASIST 的 git 也会读。这 4 个值放进 ASIST 写入 userData 的设置文件，用 GIT_CONFIG_GLOBAL 指向该文件，在与用户全局设置相同的层级传入。这样，仓库自身 .git/config 里写入的值就会和用户 git 一样被优先采用。core.excludesFile 按写入的原样传值，因此开头的 ~ 由 ASIST 的 git 从与用户 git 相同的 HOME 展开。这些值在启动后第一次使用 git 时只读一次。因为如果每次调用 git 都启动一次用户 git，每次都会多出 1 次 git 调用，而且同一次运行中，合入前的检查与任务的检出还可能按不同的值运行。在 Windows 上，把 PATH 中最先找到的 git.exe（除去 ASIST 内置的那个）当作用户 git，在仓库之外运行它，读取系统与全局层级的值。找不到就什么都不沿用。在 macOS 上不运行用户的 git。因为在没有 Command Line Tools 的 Mac 上，/usr/bin/git 会打开安装对话框。改为用内置的 git 只读全局层级（~/.gitconfig 等）。Apple 的 git 与 Homebrew 的 git 的系统设置都没有决定这 4 项。

## 未采纳的方案

- **完全不读用户的设置。** 在用 Windows 默认设置 clone 的仓库里，任务无法合入。
- **只沿用换行符与符号链接的这 3 项。** 仓库里有用户全局忽略的 .DS_Store 等文件时，任何任务都会被拒绝合入。在用户的 git 里看不到要暂存的东西，因此照着做也修不好。
- **读取用户系统和全局的全部设置。** 如果有 commit.gpgsign，每次提交都要等签名；像 git-lfs 这样的过滤器或钩子所指的程序，在从开始菜单或 Finder 打开的应用的 PATH 里找不到。
- **用 GIT_CONFIG_COUNT 或 -c 传入沿用的值。** 命令行层级优先于仓库设置，所以即使仓库定为 core.autocrlf=false 也会被覆盖，读写文件的方式与用户 git 不同。

## 实测

2026-09-27 在 Windows 11 的验证机上，用与 ASIST 相同的环境变量运行内置的 MinGit 做了确认。

- 对按 Git for Windows 默认设置（`C:/Program Files/Git/etc/gitconfig` 的 core.autocrlf=true）检出的仓库，用户 git 视为没有变更，ASIST 的 git 则把所有文件都视为有变更。在 git diff --stat 中，每一行都成了差异。
- 即使是在开发者模式下能创建符号链接的机器上，MinGit 的 git init 也会写入 core.symlinks=false，仓库的链接在 worktree 中被作为普通文件检出。

2026-10-02 在 Apple M5 的 Mac 上，用与尚未沿用 core.excludesFile 时的 ASIST 相同的环境变量运行内置的 git 2.55 做了确认。

- 用 ~/.gitconfig 的 core.excludesfile 忽略 .DS_Store 时，用户 git 的 status 不输出任何内容，ASIST 的 git 输出了 `?? .DS_Store`。
- 把同一份清单写进 ~/.config/git/ignore 时，ASIST 的 git 也没有输出 .DS_Store。

## 已知限制

- 用户改了设置，也要等到重新启动 ASIST 才会反映。改写 core.excludesFile 所指文件的内容时，从下一次调用 git 起反映。
- 在 macOS 上，用户在 git 系统层级自己写入的值不会沿用。
- 在 Windows 上，把启动 ASIST 时的 PATH 中最先找到的 git.exe 视为用户的 git。对不同终端使用不同 git 的人，ASIST 选中的 git 可能与平时用的 git 不同。
- 用户 git 无法运行时，或者用的是没有 `--show-scope` 的 2.26 之前的 git 时，读不到值，ASIST 的 git 操作全部失败。
- safe.directory 不沿用。git 认为所有者与当前用户不同的仓库（别的账号 clone 的、位于 Windows 的 FAT 或 exFAT 驱动器上的等），ASIST 的 git 打不开，所以以那里为工作位置的可写任务不会开始，并给出 git 的理由。只读任务不使用 git，所以就在该文件夹里照常运行。
- 记忆的仓库也受同样的值影响。在 Windows 上使用 core.autocrlf=true 的用户，整理任务改动的记忆文件会以 CRLF 写到磁盘上。记忆的读取能读 CRLF 的行。用户全局忽略的文件也不会进入记忆的提交。
