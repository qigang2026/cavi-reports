# MD 生成 · 质量保证手册

**文档版本**：v1.0
**创建日期**：2026-08-27
**目的**：定义 MD 文档生成完成后的**验收标准**，确保每份报告合格
**关联**：`docs/MD-数据-需求清单.md`（字段表）· `docs/MD-生成-数据源与可靠性.md`（数据可靠）

---

## 一、验收流程

```
生成 MD
   ↓
[自动校验 ① 字段完整性]    ← 必填字段、字段类型、字段范围
   ↓ pass
[自动校验 ② 跨字段一致性]  ← CAVI 唯一性、推荐版本唯一性、价格相等
   ↓ pass
[自动校验 ③ 结构完整性]    ← 8 段都在、西语 eyebrow 字面、变量无残留
   ↓ pass
[自动校验 ④ 风格合规]      ← 西语 eyebrow、AI 文案长度、敏感词
   ↓ pass
[人工抽样审核 ⑤ 5%]        ← 完整审阅
   ↓ pass
发布 / 失败回滚
```

**任何一步失败 → 阻塞发布，进入失败案例库**

---

## 二、字段完整性校验

### 2.1 必填字段清单

**frontmatter 必填（缺一即失败）**：

| Key | 校验 |
|-----|------|
| `title` | 必填，string |
| `series_id` | 必填，int |
| `model` | 必填，string，snake_case |
| `year` | 必填，int，4 位 |
| `market` | 必填，enum（MX/CN/CO/AR）|
| `lang` | 必填，enum（es/zh/en）|
| `energy_type` | 必填，enum |
| `body_type` | 必填，string |
| `template_id` | 必填，string |
| `cavi_score` | 必填，decimal 0-5（**关键字段**）|
| `cavi_recommend` | 必填，int 0-100 |
| `cavi_verdict` | 必填，string |
| `spec_strip` | 必填，list<3> |
| `key_specs` | 必填，list<3-4> |
| `finance_cards` | 必填，object（**关键字段**）|
| `competitors` | 必填，list<4> |
| `next_cards` | 必填，list<3> |

**段字段必填**：

| 段 | 必填字段 | 失败处理 |
|----|---------|---------|
| Hero | `title`, `image_url`, `cavi_score` | 阻塞（cavi_score 缺失）|
| Spec strip | 3 条完整 | 阻塞 |
| Version | 6 条完整 + 1 个 `is_recommended` | 阻塞 |
| Price 卡 1（主价）| `price` | **阻塞** |
| Price 卡 2（银行）| `monthly`, `term`, `rate` | 跳过银行卡位置显示"联系经销商" |
| Price 卡 3（厂家）| `monthly`, `term`, `rate` | 跳过 |
| Price 卡 4（交换）| `amount` | 跳过 |
| CAVI 评分 | `cavi_score` + 4 维度 | 阻塞（cavi 总分缺失）|
| CAVI 评论 | `featured_reviews[]` | 隐藏评论卡片 |
| Cost | `annual_cost.total` | 整段跳过 |
| Protection | `warranty.years` | 整段跳过 |
| Competitors | 4 条 | 阻塞（无竞品 = 无决策辅助）|
| Next steps | 3 条 | 阻塞 |

### 2.2 字段类型校验

| 类型 | 校验规则 |
|------|---------|
| `int` | 整数；价格类为正整数 |
| `decimal` | 0-5（CAVI 评分）|
| `string` | 非空；`title` ≤ 100 字；`cavi_verdict` ≤ 50 字 |
| `text` | 非空；`hero.desc` 30-100 字；`insight` 30-100 字 |
| `url` | 必须以 `https://` 开头 |
| `list<N>` | 恰好 N 条 |
| `list<M-N>` | M-N 条 |
| `enum` | 在白名单内 |
| `bool` | true/false |

### 2.3 字段范围校验

| 字段 | 范围 |
|------|------|
| `cavi_score` | 0-5 |
| `cavi_recommend` | 0-100 |
| `cavi_dimensions[].value` | 0-5 |
| `cavi_dimensions[].stars` | 1-5 |
| `versions[].msrp` | > 0 |
| `versions[].official` | ≥ versions[].msrp |
| `competitors[].price` | > 0 |
| `finance_cards.*.monthly` | > 0 |
| `finance_cards.*.rate` | 0-100 |
| `annual_cost.total` | > 0 |
| `pct` | 0-100 |
| `review_count` | ≥ 0 |

---

## 三、跨字段一致性校验

### 3.1 一致性规则清单

| ID | 规则 | 涉及字段 | 失败处理 |
|----|------|---------|---------|
| C1 | **CAVI 总分唯一** | frontmatter.cavi_score = hero.cavi_score = cavi_summary.s | **阻塞**，告警 |
| C2 | **CAVI 推荐率唯一** | frontmatter.cavi_recommend = cavi_meta.recommend | **阻塞** |
| C3 | **推荐版本唯一** | `versions[].is_recommended=true` 恰好 1 个 | 修复（多余 → 留分数最高的）|
| C4 | **主价卡 = 推荐版本 MSRP** | `finance_cards.main.price` = `versions[recommended].msrp` | **阻塞** |
| C5 | **主价卡版本名 = 推荐版本** | `finance_cards.main.version_name` = `versions[recommended].name` | **阻塞** |
| C6 | **4 维度数量 = 4** | `cavi_dimensions` 长度 = 4 | **阻塞** |
| C7 | **spec_strip 数量 = 3** | `spec_strip` 长度 = 3 | **阻塞** |
| C8 | **key_specs 数量 3-4** | `key_specs` 长度 ∈ [3,4] | **阻塞** |
| C9 | **competitors 数量 = 4** | `competitors` 长度 = 4 | **阻塞** |
| C10 | **next_cards 数量 = 3** | `next_cards` 长度 = 3 | **阻塞** |
| C11 | **竞品首条是本车** | `competitors[0].tag = "SELF"` | **阻塞** |
| C12 | **next_cards type 不重** | 3 条的 type 各不相同 | **阻塞** |
| C13 | **1 个 is_weakness** | 4 维度中 0 或 1 个 `is_weakness=true` | 警告（不是阻塞）|
| C14 | **annual_cost 占比之和 ≈ 100** | Σ items.pct = 100 ±1 | 警告 |
| C15 | **价格非负** | 所有 price/amount ≥ 0 | **阻塞** |

### 3.2 自动校验脚本

```python
# 简化示意
def validate_consistency(md_data):
    errors = []
    
    # C1: CAVI 总分
    scores = [
        md_data['cavi_score'],
        md_data['hero']['cavi_score'],
        # 段 04 提取
    ]
    if len(set(scores)) > 1:
        errors.append(('C1', 'CAVI score inconsistent'))
    
    # C3: 推荐版本唯一
    rec_versions = [v for v in md_data['versions'] if v['is_recommended']]
    if len(rec_versions) != 1:
        errors.append(('C3', f'Expected 1 recommended, got {len(rec_versions)}'))
    
    # C4: 主价卡 = 推荐版本 MSRP
    rec = rec_versions[0]
    if md_data['finance_cards']['main']['price'] != rec['msrp']:
        errors.append(('C4', 'Main price != recommended version MSRP'))
    
    # ... 其他规则
    
    return errors
```

---

## 四、结构完整性校验

### 4.1 8 段必须在

```python
REQUIRED_SECTIONS = [
    ('Hero',         r'^# .+'),                              # 文档标题
    ('Spec strip',   r'## 身份识别|## Spec strip'),          # 首屏内容
    ('Version',      r'## 版本|## Versiones'),                # 版本选择
    ('02 Price',     r'## 02 · PRECIO Y FINANCIAMIENTO'),    # 字面 eyebrow
    ('03 Selling',   r'## 03 · CORE SELLING POINTS'),
    ('04 CAVI',      r'## 04 · CAVI · RESEÑAS'),
    ('05 Cost',      r'## 05 · COST'),
    ('06 Protection',r'## 06 · PROTECTION'),
    ('07 Competitors',r'## 07 · COMPETITORS'),
    ('08 Next',      r'## 08 · SIGUIENTE PASO'),
]
```

**任一缺失 → 阻塞**。

### 4.2 西语 block-eyebrow 字面锁定

```python
SPANISH_EYEBROWS = {
    '02': '02 · PRECIO Y FINANCIAMIENTO',
    '03': '03 · CORE SELLING POINTS',
    '04': '04 · CAVI · RESEÑAS',
    '05': '05 · COST',
    '06': '06 · PROTECTION',
    '07': '07 · COMPETITORS',
    '08': '08 · SIGUIENTE PASO',
}

# 校验：模板渲染时，eyebrow 必须从 SPANISH_EYEBROWS 取
# 任何"PRÉCIO"/"FINANCIAMIENTO 2"等变体 = 失败
```

**未来扩展**：中文模板对应 `02 · 价格与金融方案` 等；待中文模板族正式上线后，加入 `CHINESE_EYEBROWS` 字典。

### 4.3 变量残留检测

```python
VARIABLE_PATTERN = r'\{\{[\w_]+\}\}|\{\{optional_[\w_]+\}\}'

# MD 中任何 {{xxx}} 残留 = 失败
# 例：{{cavi_score}} 还在 = AI 没替换 = 阻塞
```

### 4.4 章节层级

- 文档标题：唯一一个 `#`
- 段标题：固定 `##` 层级
- 子章节：`###` 层级
- 不允许 `#` 出现 2+ 次
- 不允许 `####` 及更深（保持简洁）

---

## 五、风格合规校验

### 5.1 AI 文案长度

| 字段 | 字数限制 | 校验 |
|------|---------|------|
| `title` | ≤ 100 | `len(title) ≤ 100` |
| `hero.subtitle` | ≤ 50 | |
| `hero.desc` | 30-100 | |
| `cavi_verdict` | ≤ 50 | |
| `insight` | 30-100 | |
| `saving_tips[]` | 每条 ≤ 50 | |
| `competitors[].pros` | ≤ 60 | |
| `competitors[].cons` | ≤ 60 | |
| `next_cards[].sub` | ≤ 80 | |

### 5.2 敏感词检测

**禁止词**（西语 + 中文 + 英文）：

```
[西语]          [中文]                [英文]
basura          垃圾                 trash
estafa          智商税                scam
estafador       割韭菜                scammer
mierda          坑                   rip-off
malo            辣鸡                  junk
engañar         智商                  sucker
```

检测 → 阻塞 + 告警。

### 5.3 客观性校验（竞品对比）

- 竞品 `pros` 和 `cons` 必须**同时存在**（每个竞品卡片有优有缺）
- 不允许 `pros` 含绝对化词汇："最好"、"最强"、"完美"等
- 不允许贬低性词汇（见 §5.2 列表）

### 5.4 翻译一致性

- 同一概念在 MD 中必须用同一翻译（"月供"不能一会儿"monthly payment"一会儿"cuota mensual"）
- 数字格式统一：千分位 `,` / 小数 `.` / 货币符号

---

## 六、人工审核规范

### 6.1 抽样规则

- **每批 5%**（最少 1 份）由人工完整审阅
- 抽样优先：高 CAVI 评分车 / 新上线车 / 重大更新后
- 抽样触发：自动校验任一 warning 即进入必抽

### 6.2 审核清单（人工）

| 维度 | 关注点 |
|------|--------|
| **事实性** | 数字是否与厂商一致？CAVI 评分是否合理？ |
| **可读性** | 文案是否通顺？AI 翻译是否别扭？ |
| **完整性** | 段 02..08 是否有缺漏？字段是否填全？ |
| **客观性** | 竞品对比是否公正？pros/cons 是否合理？ |
| **风格** | 是否符合"专业但亲切"语气？ |
| **业务** | 是否符合 AUTOCAVA 业务定位？ |

### 6.3 审核结果分类

| 类别 | 处理 |
|------|------|
| ✅ 通过 | 发布 |
| ⚠️ 小问题 | 修复后重生成 + 重校验，无需重新审核 |
| ❌ 阻塞问题 | 重新生成 + 完整重审 |

---

## 七、失败案例库

### 7.1 案例格式

```yaml
case_id: C-2026-08-27-001
discovered_at: 2026-08-27
discovered_by: 人工审核 / 自动校验
severity: P0 / P1 / P2
title: 主价卡与推荐版本 MSRP 不一致

context:
  series_id: 356
  market: MX
  template_id: pc-mx-fuel-sedan
  generation_date: 2026-08-27

what_happened: |
  finance_cards.main.price = 374900
  versions[is_recommended=true].msrp = 374990
  差额 90 MXN

root_cause: |
  finance API 返回的价格包含交换补贴，versions API 返回的是 MSRP。
  二者计算口径不同。

fix: |
  finance_cards.main.price 改为 versions[recommended].msrp
  优惠项在 main.price_base / bonus_trade_in / price_effective 体现

prevention: |
  在 SKILL 中显式声明：main.price = versions[recommended].msrp
  增加单元测试覆盖此 case

status: 已修复 / 待修复 / 已知
```

### 7.2 已知 case 列表（待建）

> 启动初期暂无历史 case。后续每次失败都录入此库，作为团队学习材料。

---

## 八、发布准入

### 8.1 发布条件（同时满足）

- ✅ 自动校验 ①②③④ 全部 pass
- ✅ 抽样审核（5%）全部通过
- ✅ 失败案例库中无 P0 未修复 case 命中本份报告
- ✅ 数据快照保存完整

### 8.2 发布失败处理

| 失败级别 | 处理 |
|---------|------|
| 字段缺失（必填）| 重生成 |
| 跨字段不一致 | 修复数据 → 重生成 |
| 西语 eyebrow 错 | 修复模板 → 重生成 |
| AI 文案违规 | AI 重写该字段 |
| 风格不通过 | AI 重写 → 重新校验 |

### 8.3 回滚

发布后 24h 内发现 P0 问题：
- 立即下线该报告
- 回滚到上一版本（如果有）
- 录入失败案例库
- 复盘后批量修复同类

---

## 九、关键指标（验收维度）

| 指标 | 公式 | 目标 |
|------|------|------|
| **自动校验通过率** | 一次通过 / 总生成数 | > 95% |
| **人工审核通过率** | 人工通过 / 抽样数 | > 90% |
| **跨字段一致率** | 一致 / 总校验 | > 99.5% |
| **字段完整率** | 必填非空 / 必填总数 | > 99% |
| **西语 eyebrow 命中率** | 字面正确 / 总段数 | 100% |
| **变量残留数** | `{{xxx}}` 出现次数 | 0 |
| **P0 失败率** | P0 case / 总生成数 | < 0.5% |
| **回滚率** | 回滚数 / 发布数 | < 0.1% |

---

## 十、相关文档

- *字段定义：`docs/MD-数据-需求清单.md`*
- *数据来源：`docs/MD-生成-数据源与可靠性.md`*
- *端到端流程：`docs/MD-生成-总览.md`*
- *AI 骨架：`skills/cavi-guide-gen/assets/standard-template-v3.md`*
