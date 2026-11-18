# 仓库成文惯例（评审硬标准）

评审本仓库的变更时，以下条目为硬违规依据；其余发现按判断性标注。

1. **description 即路由触发器**：SKILL.md frontmatter `description` 必须与技能实际行为一致——新增行为轴（如 Copy Audit）入册时必须同步进 description。漏说 = 路由失灵。
2. **provenance 台账**：SKILL.md 末尾「方法论来源」段的全称断言（如「其余流程均为原文技巧」）必须始终为真——任何非 Anshu 原文的机制入册（SKILL.md 或 README），须在同一变更内于该段登记为本地裁量/修订。
3. **公开仓卫生**：不出现本地私有路径与私有仓引用；内嵌命令在 macOS bash 原样可执行。
4. **状态诚实**：README「状态」节按 已实测 / 修订未实测 / 设计未实测 三档标注，新机制入册时同步加行。
5. **词表单点**：中文营销腔词表只存在于 SKILL.md 的 grep 行一处，CI 从该行提取扫描 README 与 docs——改词表只改 SKILL.md，不另建副本。
