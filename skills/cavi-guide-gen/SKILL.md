---
name: cavi-guide-gen
description: 生成 AUTOCAVA CAVI 购车手册。从内部系统获取车型数据，按照标准模板生成 Markdown 文档，供前端渲染展示。
version: 2.0
input:
  series_id: string (required)  # 车系 ID，如 "356"
  market: string (required)    # 市场代码，如 "MX"
  lang: string (optional)       # 语言，默认 "es"
  force_update: boolean         # 强制更新，默认 false
output:
  file_path: string            # 输出路径，如 "zh/nissan-versa-2026.md"
  series_id: number           # 车系 ID
  model: string               # 车型名
---

# CAVI 购车手册生成工具 v2.0

生成让用户"看懂和理解"的车辆解读报告，核心是**翻译参数为用户价值**。

## 约束规则

### 必须遵守

1. **标题层级**：`#` 只出现一次（文档标题），`##` 为一级章节，`###` 为二级章节
2. **表格格式**：使用标准 Markdown 表格，`|` 分隔，对齐符 `|` 不少于 3 个
3. **变量替换**：所有 `{{变量名}}` 必须替换为实际数据，禁止保留占位符
4. **货币格式**：使用 `$` 前缀，如 `$309,990`，不加单位缩写
5. **数字单位**：km/L、HP、N·m 等保持原样，不转换
6. **禁止**：不生成代码块（除版本配置差异的列表）、不生成 PlantUML、不生成 Mermaid

### 风格规范

| 维度 | 规范 |
|------|------|
| **语气** | 专业但亲切，像懂车的朋友在讲解 |
| **句子** | 简短，避免长句，超过 20 字考虑拆分 |
| **数据** | 先说结论，再说数据支撑 |
| **对比** | 用"比...更..."句式，如"比竞品省油 15%" |
| **推荐** | 给出明确版本推荐，说明理由 |

### 翻译原则

❌ 错误示范：
> 发动机最大功率 118 HP，最大扭矩 149 N·m

✅ 正确示范：
> 118 马力，城市通勤绰绰有余；149 N·m 扭矩，起步轻快

### 敏感词处理

- 不使用：丐版、辣鸡、智商税、割韭菜
- 竞品对比：客观陈述，不恶意贬低
- 风险提示：中立表达，如"需要注意"而非"坑"

## 工作流程

### Step 1: 读取标准模板 v2.0

```bash
读取 assets/standard-template-v2.md
```

### Step 2: 确定文件名

**文件名规则**：`{series_id}-{车型名}-{年款}.md`

示例：`zh/nissan-versa-2026.md`

> 输出目录为 `reports/{lang}/`，如 `reports/es/`、`reports/zh/`，文件命名不再带 `356-` 前缀和 `-es`/`-zh` 后缀（语言通过目录区分）。

- series_id：来自 CAVI API
- 车型名：小写、连字符分隔
- 年款：4 位数字

### Step 3: 获取车型数据

从 CAVI 内部 API 获取车型数据：

```
GET /api/v1/series/{series_id}
```

返回数据包括：
- 车型基本信息（品牌、车系、年款）
- 版本列表及价格
- 金融方案（月供计算）
- 车辆参数（尺寸、动力、油耗等）

**内部数据源，无需外部爬取。**

### Step 4: 按模板填充数据

1. 读取 `assets/standard-template-v2.md`
2. 按照 Tab 顺序逐个填充章节
3. 替换 `{{变量名}}` 为实际数据
4. 删除未使用的可选区块（如 `{{optional_xxx}}`）

### Step 5: 生成文档

**输出路径**：
```
/Users/qigangye/Mstar/购车手册/reports/{lang}/{series_id}-{车型名}-{年款}.md
```

**示例**：`/Users/qigangye/Mstar/购车手册/reports/zh/nissan-versa-2026.md`

## Frontmatter 规范

```yaml
---
title: Nissan Versa 2026 购车手册
series_id: 356              # CAVI 系统车系 ID
model: nissan-versa         # 车型名（小写、连字符）
year: 2026
market: MX                  # 市场：MX=墨西哥
lang: es                    # 语言：es=西班牙语
energy_type: 燃油
body_type: Sedán
generated_at: 2026-08-27
---
```

## 快速检查清单

生成报告后确认：
- [ ] 已读取 `assets/standard-template-v2.md` 骨架
- [ ] Frontmatter 完整
- [ ] 9 个 Tab 章节齐全（概览、参数、金融、卖点、口碑、成本、保障、竞品、选购）
- [ ] 所有 `{{变量}}` 已替换为实际值
- [ ] 表格格式正确
- [ ] 行动入口链接有效

## 模板版本

| 版本 | 文件 | 用途 |
|------|------|------|
| v1.0 | `assets/standard-template.md` | 旧版（不推荐） |
| **v2.0** | `assets/standard-template-v2.md` | **标准版（推荐使用）** |

### v2.0 模板与前端 Tab 映射

AI 生成时必须遵循此映射关系：

| Tab | MD 章节 | 说明 |
|-----|---------|------|
| 概览 | `# {{title}}` + `## 身份识别` | 车型基本信息、适合人群 |
| 参数 | `## 详细参数` | 基本信息、动力、安全、科技 |
| 金融 | `## 价格与金融方案` | 版本价格、月供计算 |
| 卖点 | `## 核心卖点` + `## 版本配置差异` | 亮点、各版本配置 |
| 口碑 | `## 用户口碑` | 真实用户评价 |
| 成本 | `## 用车成本` | 油耗、保险、保养 |
| 保障 | `## 购车保障与服务覆盖` | 质保、售后网络 |
| 竞品 | `## 竞品车型` | 对比表格、优劣分析 |
| 选购 | `## 选购指南` + `## 行动入口` | 推荐版本、行动按钮 |

### 使用 v2.0 模板

```bash
读取 assets/standard-template-v2.md
```
