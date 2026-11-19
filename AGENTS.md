# AGENTS.md

面向 agent 的仓库配置入口。硬规则（branch protection、评审口径）见 [docs/GITHUB-FLOW.md](docs/GITHUB-FLOW.md) 与 [docs/CONVENTIONS.md](docs/CONVENTIONS.md)。

## Agent skills

### Issue tracker

Issues 跟踪在本仓库的 GitHub Issues，一律经 `gh` CLI 读写。See `docs/agents/issue-tracker.md`.

### Triage labels

使用默认五标签（needs-triage / needs-info / ready-for-agent / ready-for-human / wontfix），角色名即标签名。See `docs/agents/triage-labels.md`.

### Domain docs

单上下文布局：根 `GLOSSARY.md` + 根 `docs/adr/`（由相关 skill 惰性创建）。See `docs/agents/domain.md`.
