# Agent 的任务把 worktree 按磁盘上实际的样子提交，index 设置让 git 看不见的文件也包括在内

worktree 里隔离的任务结束后，ASIST 把 Agent 留下的 worktree 按文件在磁盘上的样子提交。git 里有让它可以不必去看文件的设置。仓库的 core.ignoreStat 生效时，git 在创建 worktree 的时候会给所有文件加上 assume-unchanged。Agent 自己也能用 update-index 加上 assume-unchanged 或 skip-worktree。fsmonitor 的钩子有时会漏掉变更通知，core.checkStat=minimal 和 core.trustctime=false 会看漏把大小和更新时间改回去的改写。原样相信这些设置去读，编辑就看不见，任务会被当作没有变更而收拾掉，编辑也跟着 worktree 一起消失。所以 ASIST 用关掉这些设置的 git 读取，磁盘上存在的文件不管带着什么设置，都按实际的样子计数。assume-unchanged 不过是让人不去看文件的标记，所以对磁盘上没有的文件也要摘掉，并把该文件计为删除。skip-worktree 也是 git 加给 sparse checkout 不检出的文件上的设置，所以对磁盘上没有的文件不摘掉，也不计为删除。sparse checkout 若不检出该文件，它就是没有变化的文件，保留 index 里已暂存的内容。Agent 的 git apply --cached、git restore --staged、git read-tree 这类把没有检出的文件重新暂存的命令也会摘掉 skip-worktree。因此，sparse checkout 不检出、却也没有带着该设置的文件在磁盘上不存在时，也重新加上设置，不计为删除。除此之外，也就是该文件在 sparse checkout 的范围之内、sparse checkout 已关闭，或记录该范围的文件不存在时，分辨不出是 Agent 藏起了删除，还是只是没有检出，因此不提交，把 worktree 留下，并把理由写进任务的日志。Agent 在没有检出的文件的位置上放了别的文件，而一旦提交，那些文件就会被删掉时，也同样不提交。读取时用任务 index 的副本，不改写任务的 index。改写 index 的只有提交，届时摘掉把磁盘上的文件藏起来的设置。在合入之前确认用户的仓库有没有变化时，也用关掉同样设置的 git 读取，不改写用户的 index。这是为了不让合入悄悄覆盖 fsmonitor 看漏的用户的编辑。

## 未采纳的方案

- **原样相信仓库和 index 的设置去读。** 编辑看不见，worktree 被删掉，Agent 的工作就丢了。
- **对非 sparse 的 worktree，摘掉所有 skip-worktree。** 在这类 worktree 上加上 skip-worktree 的不只是 Agent。用仓库整体的设置启用 sparse checkout，用户在自己的工作树里把它关掉时，或 Agent 没有重新检出就关掉时，没有检出的文件全都会被当作删除提交。
- **对 sparse 的 worktree 也摘掉所有 skip-worktree。** 没有检出的文件全都会被当作删除提交。
- **对 sparse checkout 的仓库不收拾任务。** sparse 的仓库里，任何任务都无法合入。
- **把范围之内却不存在的文件当作删除。** 只是改了范围还没有重新检出时，就会把 Agent 并没有做过的删除排进待合入之列。
- **每次读取时从任务的 index 摘掉设置。** 每次合入前做确认，index 都会变，别的 git 锁着 index 时，确认就会失败。在 sparse checkout 被临时关掉的期间做确认，设置会一直保持摘掉的状态，恢复之后所有文件都看起来被删除。
- **在 worktree 里执行 git sparse-checkout reapply 再读。** 这会改写 worktree 的文件，Agent 删掉的文件会回来，提交的东西与 Agent 留下的并不一致。

## 已知限制

- 碰上分不清某个文件是没有检出还是被删除的任务，在用户于 worktree 里重新检出，或改成按删除处理之前，任务不会收拾。
- Agent 删掉文件之后，把 sparse checkout 的范围收窄到排除该文件，git 会给该文件加上 skip-worktree，删除就不进提交。git 自己也把该文件当作没有变化。
- Agent 在 sparse checkout 的范围之外写入并暂存的文件，之后即使从磁盘上删掉，也不作为删除提交，而是保留。要删掉范围之外的文件，需要 git rm --sparse。
- 在没有检出的文件夹的位置放一个符号链接，指向持有该处应有的全部文件名的文件夹，这些文件就会被计为存在于磁盘上，无法阻止它们在提交里被该链接取代而消失。这是因为 lstat 会沿路径中途的链接走下去，git 自己的 status 也这样处理。
- 用户的仓库里设了 core.trustctime=false 或 core.checkStat=minimal，并且有 ctime 已变的文件时，合入前的确认每次合入都会重新读这些文件的内容，不把结果写回 index。每 1 GB 约花 1.6 秒，但结果是对的。
