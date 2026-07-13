# fancy_skills

我自己做的一些 [Claude Code](https://claude.com/claude-code) Skills 合集。每个 skill 是一段可复用的工作流，放在这里统一维护、方便在多台机器间同步。

## 目录结构

```
fancy_skills/
├── README.md
└── skills/
    └── <skill_name>/
        └── SKILL.md        # skill 主文件（带 YAML frontmatter）
        └── ...             # 可选：脚本、引用文档等辅助文件
```

- 一个 skill 一个目录，目录名即 skill 名。
- 每个 skill 至少包含一个 `SKILL.md`，开头是 YAML frontmatter：
  ```yaml
  ---
  name: skill_name
  description: 一句话说明「做什么 + 什么时候触发」，Claude 靠它判断是否调用这个 skill。
  ---
  ```
- 需要额外脚本 / 模板 / 参考资料时，放进同一 skill 目录，在 `SKILL.md` 里引用即可。

## 已收录的 skill

| Skill | 说明 |
|---|---|
| [save_ai_article](skills/save_ai_article/SKILL.md) | 把文章链接录入指定飞书多维表格：抓标题、判断分类，填 标题/链接/分类/来源/日期。 |

## 如何使用（安装到 Claude Code）

Claude Code 的个人 skill 目录是 `~/.claude/skills/`。把本仓库的某个 skill 装上有两种方式：

**方式 A：软链接（推荐，改动实时生效）**
```bash
git clone https://github.com/Rockelbel/fancy_skills.git
ln -s "$PWD/fancy_skills/skills/save_ai_article" ~/.claude/skills/save_ai_article
```

**方式 B：直接拷贝**
```bash
cp -r fancy_skills/skills/save_ai_article ~/.claude/skills/
```

装好后在 Claude Code 里输入 `/save_ai_article` 或直接描述任务即可触发。

## 如何新增 / 维护 skill

1. 在 `skills/` 下新建目录 `skills/<新名字>/`，写好 `SKILL.md`（含 frontmatter）。
2. 本地调试：链接或拷贝到 `~/.claude/skills/` 试用，跑通后再回填改动。
3. 在本 README 的「已收录的 skill」表格里补一行。
4. 提交并推送：
   ```bash
   git add skills/<新名字> README.md
   git commit -m "add skill: <新名字>"
   git push
   ```

## 命名约定

- skill 名用小写 + 下划线（如 `save_ai_article`）。
- `description` 要写清「触发条件」，Claude 据此自动选用，别只写功能。
