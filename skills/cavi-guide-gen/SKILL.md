---
name: cavi-guide-gen
description: 生成 AUTOCAVA CAVI 购车报告 MD 文档。Versa 2026 标杆场景下，从 4 个真实数据源（CAVI API / CAVI 评分 / 销售 / 金融 API）获取数据，由 AI 润色自然语言部分，输出符合 v3.0 模板的 MD 文档。
version: 3.1
input:
  series_id: int (required)  # 当前仅支持 356（Versa 2026）
  force_update: bool         # 强制重生成，默认 false
  priority: enum             # high / normal / low，默认 normal
output:
  file_path: string          # 输出路径，如 "reports/es/nissan-versa-2026.md"
  status: enum               # success / failed / partial
  data_version: string       # 数据快照版本
  validation: object         # 校验结果（含 ai_compliance_check）
---

# CAVI 购车报告生成工具 v3.1

> v3.1 升级：聚焦 Versa 2026 标杆 + AI 润色不编数 + 4 个真实数据源。

## ⚠️ 核心原则（不可违反）

### AI 不得触碰的字段

以下字段由系统提供，AI **只能原样填入**：

- ❌ **价格 / MSRP / official**（如 374,990）
- ❌ **月供 / 利率**（如 6,244 / 12.9%）
- ❌ **销量 / 排名**
- ❌ **CAVI 评分**（综合 4.6 + 4 维度）
- ❌ **评论原文**
- ❌ **质保年限 / 里程**
- ❌ **任何数字、日期、版本号**

### AI 可以做的

- ✅ 润色：把"118 HP · 5.3L/100km"变成"动力够用且省油"
- ✅ 拼装：把系统分 + 评论拼成"一句洞察"（≤ 50 字）
- ✅ 翻译：把系统给的英文 raw 数据译成西语（**保留数字字面**）
- ✅ 竞品 pros/cons：基于系统数据写"+"和"-"（≤ 60 字，**不许编数字**）

**违反上述规则 = E203 错误 = 重生成 + 报警**（P0 级别）

---

## 工作流程

### Step 1: 读取标准模板 v3.0

```bash
读取 assets/standard-template-v3.md
```

### Step 2: 确定文件名

**文件名规则**：`{model}-{year}.md`（标杆固定）

示例：`reports/es/nissan-versa-2026.md`

> 标杆固定 = 文件名固定

### Step 3: 获取 4 个真实数据源数据（后端负责）

> AI Agent 不直接调用 API，由后端统一拉取并组装成字段表。

**4 个数据源**：
1. **CAVI 内部 API**：车系 / 6 版本 / 规格 / MSRP / official / 月供估算
2. **CAVI 评分系统**：综合 4.6 / 推荐率 89% / 4 维度（cajuela/consumo/seguridad/ruido）
3. **销售系统**：月销量 7,486 / 排名 sedán #1 / 评论人数 3,240
4. **金融 API**：BBVA 36 / Nissan 厂家 48 / 交换补贴

### Step 4: 按字段表填数据

读取 `docs/MD-数据-需求清单.md` 作为权威字段表，按"系统 / 混合 / AI"分类填数据：

| 类型 | 行为 |
|------|------|
| **系统字段** | 原样填入，**不修改** |
| **混合字段** | 系统值保留，AI 拼装成自然语言（数字不许变）|
| **AI 字段** | AI 自由生成，遵守风格规范 |

**关键规则**：
- 必填字段缺失 → 触发 E102，**不生成**报告
- 价格 / 版本缺失 → 阻塞
- 兜底策略见 `docs/MD-生成-数据源与可靠性.md` §五

### Step 5: 8 段结构填充

按 `assets/standard-template-v3.md` 的 8 段结构填充：

```
段 0  首屏（Hero + Spec strip + Trim selector + Compare bar）
段 02 PRECIO Y FINANCIAMIENTO
段 03 CORE SELLING POINTS
段 04 CAVI · RESEÑAS
段 05 COST
段 06 PROTECTION
段 07 COMPETITORS
段 08 SIGUIENTE PASO
```

### Step 6: 校验

按 `docs/MD-生成-质量保证.md` 5 步校验：
1. 字段完整性
2. 跨字段一致性（15 条）
3. 结构完整性（8 段 + eyebrow 字面）
4. **AI 字段合规**（不许改数字）
5. 人工抽样审核

### Step 7: 输出

```yaml
# 同步模式
status: success
md_file_path: reports/es/nissan-versa-2026.md
data_version: v20260827-1430
generated_at: 2026-08-27T14:30:00Z
duration_ms: 4200
validation:
  auto_pass: true
  fields_complete: "99.5%"
  consistency_check: "pass"
  ai_compliance_check: "pass"
```

---

## 风格规范

### 语气

- 专业但亲切，像懂车的朋友在讲解
- 西语，**用西语术语而非英语**（如 `enganche` 而非 `down payment`）

### 句子

- 简短，避免长句，超过 20 字考虑拆分

### 数据呈现

- **系统给的数字必须原样出现**（不改、不省略位数）
- 可以加修饰词（"Top #1"、"Best seller"）
- 单位、币种由模板渲染时加，AI 不带 `$`/`MXN`/`，` 千分位

### 翻译原则

❌ 错误示范：
> Engine max power 118 HP, max torque 149 N·m

✅ 正确示范：
> 118 马力，城市通勤绰绰有余；149 N·m 扭矩，起步轻快

### 敏感词处理

- 不使用：basura、estafa、辣鸡、智商税、junk、scam
- 竞品对比：客观陈述，不恶意贬低
- 风险提示：中立表达

---

## 必填字段检查（Versa 标杆）

| 字段 | 必填 | 来源 |
|------|------|------|
| `series_id` | ✅ | = 356 |
| `model` | ✅ | = "nissan-versa" |
| `year` | ✅ | = 2026 |
| `market` | ✅ | = "MX" |
| `lang` | ✅ | = "es" |
| `energy_type` | ✅ | = "燃油" |
| `body_type` | ✅ | = "Sedán" |
| `cavi_score` | ✅ | = 4.6（系统）|
| `cavi_recommend` | ✅ | = 89（系统）|
| `cavi_dimensions` | ✅ | 4 条（cajuela/consumo/seguridad/ruido）|
| `spec_strip` | ✅ | 3 条 |
| `versions` | ✅ | 6 条，1 个 is_recommended |
| `finance_cards` | ✅ | 4 卡（main / bank / factory / trade_in）|
| `competitors` | ✅ | 4 张（self + 3 竞品）|
| `next_cards` | ✅ | 3 张 |

---

## 模板版本

| 版本 | 文件 | 用途 |
|------|------|------|
| v1.0 | `assets/standard-template.md` | 旧版（不推荐）|
| v2.0 | `assets/standard-template-v2.md` | 中间版（9 Tab，已被替代）|
| **v3.0** | `assets/standard-template-v3.md` | **标准版（推荐使用）** · 8 段结构 + 西语 block-eyebrow |

---

## 错误码（v3.1 新增 E203）

| Code | 含义 |
|------|------|
| E001 | 输入参数缺失或非法 |
| E002 | series_id 不在白名单 |
| E101 | 数据获取失败 |
| E102 | 关键字段缺失（价格/版本/CAVI）|
| E103 | 数据陈旧 |
| E201 | 自动校验不通过 |
| E202 | 跨字段不一致 |
| **E203** | **AI 润色时篡改系统数字**（P0）|
| E301 | AI 调用失败 |
| E302 | AI 返回非法格式 |
| E401 | 超时（> 60s）|
| E500 | 系统错误 |

---

## 变更日志

- v3.0（2026-08-27）：升级到 8 段结构
- v3.1（2026-08-27）：聚焦 Versa 标杆 + AI 润色不编数 + 新增 E203 错误码

---

*相关文档：*
- *字段表：`docs/MD-数据-需求清单.md`*
- *数据源：`docs/MD-生成-数据源与可靠性.md`*
- *质量保证：`docs/MD-生成-质量保证.md`*
- *总览：`docs/MD-生成-总览.md`*
- *SKILL 契约：`docs/MD-生成-SKILL-调用契约.md`*
- *AI 骨架：`assets/standard-template-v3.md`*
