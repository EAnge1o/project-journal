# Project Journal（项目迭代日志）

> A skill for SKILL.md-compatible coding agents (ZCode / Claude Code / Codex …): auto-write a decision log for every work session into `JOURNAL.md`, so you — and future agent sessions — never re-tread dead ends.

git 只记录"改了什么"，不记录"为什么这么改、试过什么失败了"。两周后回看项目，失败路径全忘了；agent 换个会话，同样的坑再踩一遍。

这个 skill 在每次会话收尾时，把下面的内容沉淀到项目根目录的 `JOURNAL.md`：

- **目标**：这轮要解决什么
- **结果**：完成 / 部分完成
- **改动**：模块级摘要（不流水账）
- **试过但放弃**：方案 → 为什么不行（最有价值的字段）
- **决策**：关键选择 + 理由
- **坑**：踩到的坑、注意事项
- **下一步**：具体到文件和动作

日志只 `commit` `JOURNAL.md` 这一个文件（不 push、不碰你未提交的代码），迭代记录和代码历史天然绑定。

## 安装

```bash
git clone https://github.com/EAnge1o/project-journal ~/.agents/skills/project-journal
```

或只想在某个项目里用，放进项目的 `.agents/skills/` 即可。任何读取 `SKILL.md` 格式的 agent 都能用。

## 用法

| 场景 | 对 agent 说 |
|---|---|
| 收尾留痕 | "收工" / "记录一下" / "保存进度" |
| 恢复上下文 | "继续上次" / "上次做到哪了" |

`JOURNAL.md` 最新条目永远在最上面。开工时 agent 会先读最新一条的「下一步 / 坑 / 试过但放弃」，从上次中断处接续，而不是重新踩坑。

## 日志长什么样

```markdown
## 2026-11-03 00:57 · 修复爬虫被 403 拒绝的死循环

**目标**：解决目标站返回 403 后无限重试的问题
**结果**：已解决（本地验证通过）
**改动**：app.py 改用 requests.Session 复用连接，补 User-Agent，重试加上限
**试过但放弃**：只换 User-Agent → 依然 403，该站还校验会话态
**决策**：Session + UA + 有限重试，因为根因是缺会话头且原循环无退出条件
**坑**：requests 默认 UA 会被该站直接拒；while True 重试必须带上限
**下一步**：把 fetch 拆成可复用的 client 类；给重试逻辑补单测
```

## 设计原则

- 手动触发，不为鸡毛蒜皮的小会话留垃圾记录
- 只提交日志文件本身，绝不 push、绝不动用户的 WIP
- 新条目永远插在最上面，旧条目是不可改写的历史
