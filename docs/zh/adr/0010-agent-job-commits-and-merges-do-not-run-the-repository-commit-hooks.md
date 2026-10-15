# Agent 任务改动在汇总提交和合入时，不运行仓库的提交钩子

在 worktree 中隔离的任务结束后，ASIST 把 Agent 留下的改动在 worktree 里汇总成一个提交，获得批准后再把它合入用户的仓库。两个提交都用 write-tree、merge-tree 和 commit-tree 在工作树之外创建，只移动分支。工作树在 worktree 中仍是汇总提交的内容，不会改变；在用户的仓库中只是以 fast-forward 前进到合入的那个提交。因此，仓库的 pre-commit、prepare-commit-msg、commit-msg、pre-merge-commit、post-commit 钩子都不会运行。给 git commit 加上 `--no-verify` 也只能停下 pre-commit 和 commit-msg，prepare-commit-msg 仍会运行。钩子通常在 worktree 和仓库中是共用的，所以 worktree 的提交也会运行用户的钩子。合入不得在用户的仓库里中途停住。用 git merge 合入时，如果钩子失败，合入就会停在半途，任务的改动处于已暂存状态。这时用户只要执行 git commit，未经任何人确认的改动就会被提交。worktree 的提交中钩子失败时，任务的改动会留在既无法合入也无法确认的状态。另外，很多钩子是用来检查提交消息格式的，commitlint 会拒绝“asist: …”这样的消息，而调用 npx 的钩子，在从 Finder 打开的应用那条狭窄的 PATH 里找不到 npx。在这类仓库里，只要让钩子运行，任务的改动就一次也无法合入。

## 未采纳的方案

- **运行钩子，钩子拒绝时就撤销合入。** 仓库会恢复原状，但在有检查消息格式的钩子、或者有调用 npx 的钩子的仓库里，合入一次也不会成功。
- **只有 worktree 的提交用 git commit 的 `--no-verify` 创建。** prepare-commit-msg 不会停下来，所以在那里调用 npx 的仓库里，worktree 的提交每次都会失败。

## 已知限制

- post-merge 钩子在 fast-forward 时会运行。其结果不影响合入的成败。
- 用 pre-merge-commit 钩子跑测试来拦住合入的用法，在 ASIST 的合入中不成立。ASIST 合入的只有用户看过差异并批准过的改动。
