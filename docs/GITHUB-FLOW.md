# 开发流程（GitHub Flow）

本仓走 GitHub Flow 单人版：main 始终可发布，一切改动经 PR 进入。规则由 branch protection 强制，不靠自觉。

## 硬规则（branch protection 强制）

- **直推 main 被拒**——对管理员同样生效（enforce admins），agent 与人一律走 PR
- **PR 必须等 `checks` CI 绿**（词表扫描 + 链接检查；strict 模式：分支须包含 main 最新提交）
- **禁 force push、禁删 main、保持线性历史**

## 一次改动的完整路径

1. 从 main 开描述性命名的分支（`docs/…` / `feat/…` / `fix/…`）
2. 小步 commit（conventional commits：`feat:` / `fix:` / `docs:` / `setup:` 等），随手 push 分支
3. 开 **draft PR** 分享进展；自查完成 → mark ready for review
4. CI 绿 → **squash merge**（一个 PR 一个提交，提交标题即 PR 标题）
5. 分支随合并自动删除（delete branch on merge）
6. 里程碑打 tag 发 release：`gh release create vX.Y.Z --generate-notes`

## 为什么 squash-only

单人仓 + agent 代 commit：PR 内中间提交是工作过程，合并后 main 的每个提交 = 一个完整语义单元，历史可回滚、可信任。仓库已关闭 merge commit / rebase merge，只留 squash。

## 参考

- [GitHub Flow 官方文档](https://docs.github.com/en/get-started/using-github/github-flow)
- 评审口径（description 一致性 / provenance / 公开仓卫生 / 状态诚实 / 词表单点）见 [CONVENTIONS.md](CONVENTIONS.md)
