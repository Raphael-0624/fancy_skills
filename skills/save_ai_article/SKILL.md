---
name: save_ai_article
description: 把一篇（或多篇）文章链接录入飞书「AI相关文章」多维表格。抓取标题、判断分类，填入 标题/链接/分类/来源/日期，其余字段留空。当用户给出文章链接并要求"入库/录入/加入多维表格/存文章"时使用。
---

# save_ai_article

把用户给的文章链接录入指定的飞书多维表格「AI相关文章」。

## 目标表格（固定常量）

- 文档: https://fenbeitong.feishu.cn/wiki/XaVawS7AUiJY4wkRXUTcNKAanTc （标题「AI科普文章」）
- base-token: `WakcbvNJzaXxa0sMh5KcMZotnlf`
- table-id: `tbl0iMGi3EVUCuAG`
- 工具: `lark-cli`（默认 user 身份），命令 `lark-cli base +record-batch-create`

## 要填的字段

只填这 5 个，**其余字段（附件/是否打印/已阅/CMTS/备注/P0P1）一律留空**：

| 字段 | 说明 |
|---|---|
| 标题 | 抓取文章 og:title |
| 链接 | 用户给的原始 URL（含跟踪参数按原样保存） |
| 分类 | 从 `AI趋势 / AI技术 / AI基础 / 其他` 中判断（见下） |
| 来源 | 从 `AI汇报群 / 友商研究群 / 微信 / AI BU / LX` 中取，**默认 AI汇报群**，除非用户指定 |
| 日期 | YYMMDD 格式（如 260713）。用户给什么用什么；明显多打一位等笔误需纠正并说明 |

## 流程

### 1. 解析输入
用户通常给「日期 + 一个或多个链接」的分组，可能还带「来源=xxx」。一个日期下可有多条链接，各自成一行、共用该日期。

### 2. 抓取标题
微信 `mp.weixin.qq.com` 会拦截 WebFetch，**必须用 curl + 移动端 UA** 抓 og:title / og:description：

```bash
UA="Mozilla/5.0 (iPhone; CPU iPhone OS 15_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/15.0 Mobile/15E148 Safari/604.1"
fetch() {
  local url="$1" html title desc
  html=$(curl -sL -A "$UA" "$url" 2>/dev/null)
  title=$(echo "$html" | grep -oE '<meta property="og:title" content="[^"]*"' | head -1 | sed 's/.*content="//;s/"$//')
  desc=$(echo "$html" | grep -oE '<meta property="og:description" content="[^"]*"' | head -1 | sed 's/.*content="//;s/"$//')
  [ -z "$title" ] && title=$(echo "$html" | grep -oE '<title>[^<]*</title>' | head -1 | sed 's/<title>//;s/<\/title>//')
  printf 'T: %s\nD: %s\n' "$title" "$desc"
}
```
- 多条链接可后台并发（`fetch "$u" &` … `wait`）加速。
- 标题里的 HTML 实体（如 `&quot;`）转成对应字符；中文引号用 `“”`。

### 3. 判断分类
参考表内既有习惯：

- **AI趋势** — 行业/市场/投融资、大厂动向与战略、SaaS 存亡与估值、商业模式、企业 AI 价值重估、带 AI 的产品发布。
- **AI技术** — 架构原理与技术拆解、AI Coding 实践、MCP/能力对比报告、AI 产品落地的架构/实战拆解（如「飞猪问一问架构演进」）。
- **AI基础** — Agent / RAG / Skill / MCP / Harness / Prompt / Context / Loop 等概念科普、工程详解、万字综述、评估体系。
- **其他** — 方法论、组织文化、团队变革、以及**无明显 AI 角度**的工具/产品（如纯差旅退订功能、旅行规划小工具）。

拿捏边界时选最贴近的一类，并在回复里简述几条关键判断，方便用户复核改分类。

### 4. 写入
把所有行写成 JSON 文件（scratchpad 目录），再批量创建：

```bash
lark-cli base +record-batch-create \
  --base-token "WakcbvNJzaXxa0sMh5KcMZotnlf" \
  --table-id "tbl0iMGi3EVUCuAG" \
  --json "$(cat /path/to/batch.json)" \
  --jq '{ok, created: (.data.record_id_list | length), ids: .data.record_id_list}'
```

batch.json 结构（rows 顺序对齐 fields，select 字段传选项名字符串）：
```json
{
  "fields": ["标题", "链接", "分类", "来源", "日期"],
  "rows": [
    ["文章标题", "https://...", "AI趋势", "AI汇报群", "260713"]
  ]
}
```

### 5. 汇报
用表格列出 日期 / 标题 / 分类 / 来源 及创建结果，并单独标注任何笔误纠正或边界分类判断，请用户复核。

## 注意
- 分类是判断题：不确定时给推荐并说明理由，让用户一句话就能改。
- 只有 5 个字段要填，别误写系统/附件字段。
- `lark-cli` 默认 user 身份即可，无需 `--as bot`。
