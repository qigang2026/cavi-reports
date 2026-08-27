# MD 数据需求清单

**文档版本**：v1.2
**创建日期**：2026-08-27
**修订日期**：2026-08-27（v1.2：明确 4 个真实数据源 + AI 润色不编数 + 标杆 Versa 1 个车型）
**目的**：MD 生成涉及的所有字段的**权威定义表**——SKILL、模板、质量保证文档都以此为准
**约定**：
- `系统` = 来自 4 个真实数据源（结构化数据，AI 不可改）
- `AI` = 由 AI 生成（润色 / 翻译，受硬约束）
- `混合` = 系统给原始值 + AI 润色（**AI 不得改数字**）

> **当前标杆**：Versa 2026 · MX · 西语 · 燃油。详细见 `docs/MD-生成-总览.md` §二。

---

## 〇、AI 不可触碰的字段清单（硬约束）

**规则**：以下字段值由系统提供，AI **只能原样填入**，**不得修改、不得润色**：

| 字段组 | 原因 |
|--------|------|
| 价格 / MSRP / official | 用户决策依据，错一位数就报错 |
| 月供 / 利率 | 同上 |
| 销量 / 排名 | 客观数据 |
| CAVI 评分（综合 + 4 维度）| 系统算法输出 |
| 评论原文 | 真实用户声音 |
| 质保年限 / 里程 | 厂家承诺 |
| 任何数字、日期、版本号 | — |

**校验机制**：见 `docs/MD-生成-质量保证.md` §5.4。

---

## 一、Frontmatter 字段

文件顶部 YAML 段，**所有字段必填**（除非标注"可选"）。

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `title` | string | AI | 车名 + 年款（润色）| `Nissan {model} {year} 购车指南` |
| `series_id` | int | **系统** | CAVI 系统车系 ID = 356 | **必填，无兜底** |
| `model` | string | **系统** | 车型名（小写、连字符）= `nissan-versa` | **必填，无兜底** |
| `year` | int | **系统** | 年款 = 2026 | **必填，无兜底** |
| `market` | enum | **系统** | `MX` | **必填，无兜底** |
| `lang` | enum | **系统** | `es` | **必填，无兜底** |
| `energy_type` | enum | **系统** | `燃油` | **必填，无兜底** |
| `body_type` | string | **系统** | `Sedán` | **必填，无兜底** |
| `device` | enum | **系统** | `pc` / `h5` | `pc` |
| `generated_at` | date | **系统** | 生成时间（ISO 8601）| 自动填 |
| `source` | string | **系统** | 数据来源标识 | `CAVI (AutoCava AI)` |
| `cavi_score` | decimal | **系统** | CAVI 综合评分 0-5 | **必填**（AI 不可改）|
| `cavi_recommend` | int | **系统** | 推荐率（%）0-100 | **必填**（AI 不可改）|
| `cavi_verdict` | string | **混合** | 系统给 raw + AI 润色 ≤ 50 字 | `"Sin datos suficientes"` |
| `spec_strip` | list<obj> | **系统** | 首屏 3 核心数据 | `[]` + 跳过此段 |
| `key_specs` | list<obj> | **系统** | 卖点章 3-4 核心数据 | `[]` + 跳过此段 |
| `annual_cost` | object | **系统** | 年度用车成本 | `null` + 跳过此段 |
| `finance_cards` | object | **系统** | 4 张金融卡 | `null` + 跳过段 02 |
| `competitors` | list<obj> | **系统** | 4 张并排卡含本车 | `[]` + 跳过此段 |
| `next_cards` | list<obj> | **系统** | 3 张行动卡 | `[]` + 跳过此段 |

---

## 二、Hero 字段

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `hero.eyebrow` | string | **系统** | `EXPERTO · CAVI` | `EXPERTO · CAVI` |
| `hero.title` | string | **系统** | 车名 + 年款 | `<model_name> {year}` |
| `hero.subtitle` | string | **AI** | 一句副标题（≤ 50 字）| `¿Vale la pena?` |
| `hero.desc` | text | **混合** | 系统给关键数（CAVI/销量/排名）+ AI 拼装 50-80 字 | `"Sin datos suficientes"` |
| `hero.image_url` | url | **系统** | 车图 CDN URL | `null` + 隐藏车图 |
| `hero.cavi_score` | decimal | **系统** | CAVI 评分（与 frontmatter 一致）| **必填** |
| `hero.monthly_sales` | int | **系统** | 月销量 | `null` |
| `hero.rank_text` | string | **混合** | 系统给数据 + AI 拼装排名描述 | `null` |

**AI 拼装规则**：`hero.desc` 中出现的数字必须等于系统给的原始值（CAVI 评分、销量等），AI 只负责把"7,486 ventas, CAVI 4.6"拼成一句通顺的西语。

---

## 三、价格与金融方案（段 02）

### 3.1 `finance_cards.main`（主价卡 · 深色）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `version_name` | string | **系统** | 推荐版本名 `ADVANCE CVT` |
| `price` | int | **系统** | MSRP（整数 MXN）= 374990 |
| `currency` | enum | **系统** | `MXN` |
| `source` | string | **系统** | 经销价来源 |
| `price_base` | int | **系统** | 基础价 |
| `bonus_trade_in` | int | **系统** | 交换补贴（负数）|
| `price_effective` | int | 计算 | = base + bonus |

### 3.2 `finance_cards.bank`（BBVA · 卡 2）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `bank_name` | string | **系统** | `BBVA` |
| `term` | int | **系统** | 期限（月）= 36 |
| `monthly` | int | **系统** | 月供 = 6244 |
| `rate` | decimal | **系统** | 年利率（%）= 12.9 |
| `loan_ratio` | int | **系统** | 贷款比例（%）= 70 |
| `down_payment` | int | 计算 | 首付金额 |
| `loan_amount` | int | 计算 | 贷款额 |
| `total_interest` | int | 计算 | 总利息 |

### 3.3 `finance_cards.factory`（Nissan 厂家 · 卡 3）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `provider` | string | **系统** | `Nissan` |
| `term` | int | **系统** | 期限（月）= 48 |
| `monthly` | int | **系统** | 月供 = 5899 |
| `rate` | string | **系统** | `0%` |
| `tag` | string | **系统** | `0% primer año` |
| `plan` | string | **系统** | `Credissan` |
| `cat` | string | **系统** | `14.8% sin IVA` |
| `down_payment` | int | 计算 | 首付金额 |

### 3.4 `finance_cards.trade_in`（交换补贴 · 卡 4）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `amount` | int | **系统** | 补贴金额 = 15000 |
| `condition` | string | **系统** | `Si entregas tu auto actual` |
| `vigencia` | date | **系统** | 截止日期 |
| `applies_to` | string | **系统** | 适用版本 `ADVANCE+` |
| `extra_bono` | int | **系统** | 额外补贴 -8000 |

### 3.5 `finance_cta_bar`（章节内 cta-bar）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `title` | string | **AI** | 标题（≤ 20 字）|
| `subtitle` | string | **AI** | 副标题 |
| `button_text` | string | **系统** | `Calcular →` |
| `url` | url | **系统** | finance apply 链接 |

---

## 四、CAVI 评分（段 04）

### 4.1 `cavi_score` 顶级字段（已在 frontmatter）

详见 §一。

### 4.2 `cavi_dimensions`（4 维度细分）

> **Versa 标杆的 4 维度锁定**（西语 + 中文 label 都保留，西语优先）：

```yaml
cavi_dimensions:
  - { slug: cajuela,   label: "Cajuela",   label_zh: "后备厢", value: 4.8, stars: 5 }
  - { slug: consumo,   label: "Consumo",   label_zh: "油耗",   value: 4.6, stars: 5 }
  - { slug: seguridad, label: "Seguridad", label_zh: "安全",   value: 4.5, stars: 4 }
  - { slug: ruido,     label: "Ruido",     label_zh: "噪音",   value: 3.9, stars: 3, is_weakness: true }
```

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `slug` | enum | **系统** | `cajuela` / `consumo` / `seguridad` / `ruido` |
| `label` | string | **系统** | 西语标签 |
| `label_zh` | string | **系统** | 中文标签（备用）|
| `value` | decimal | **系统** | 0-5 |
| `stars` | int | 计算 | = round(value) |
| `is_weakness` | bool | **系统** | ruido 维度为 true |

### 4.3 `featured_reviews`（精选评论 2-3 条）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `stars` | int | **系统** | 1-5 |
| `content` | text | **系统** | 评论原文（≤ 100 字）|
| `version` | string | **系统** | 评论者版本 |
| `mileage_km` | int | **系统** | 行驶里程 |
| `location` | string | **系统** | 评论者城市 |

### 4.4 `cavi_meta`（元信息）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `review_count` | int | **系统** | 评论人数 = 3,240 |
| `monthly_sales` | int | **系统** | 月销量 = 7,486 |
| `rank_text` | string | **系统** | 排名 `sedán #1 en México` |

---

## 五、用车成本（段 05）

```yaml
annual_cost:
  total: 17300
  currency: "MXN"
  items:
    - { slug: fuel,        label: "Combustible", amount: 8000 }
    - { slug: insurance,   label: "Seguro",      amount: 4000 }
    - { slug: maintenance, label: "Mantenimiento",amount: 2500 }
    - { slug: depreciation,label: "Depreciación",amount: 2000 }
    - { slug: tax,         label: "Tenencia",    amount: 800 }
  assumptions:
    annual_km: 15000
  saving_tips:
    - "Mantén presión de neumáticos en 32 PSI — ahorra 3% combustible"
```

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `total` | int | **系统** | 年度总成本 |
| `currency` | enum | **系统** | `MXN` |
| `items[].amount` | int | **系统** | 5 项金额 |
| `assumptions.annual_km` | int | **系统** | 假设年行驶里程 |
| `saving_tips` | list<string> | **AI** | 2-3 条省钱建议（AI 写，不涉数字）|

**5 项必含 slug**（顺序固定）：`fuel` / `insurance` / `maintenance` / `depreciation` / `tax`

---

## 六、购车保障（段 06）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `warranty.years` | int | **系统** | 整车质保年限 |
| `warranty.mileage_km` | int | **系统** | 整车质保里程 |
| `warranty.details` | text | **系统** | 质保细节 |
| `maintenance.included_count` | int | **系统** | 保养包次数 |
| `maintenance.mileage_km` | int | **系统** | 保养包里程 |
| `service_network.count` | int | **系统** | 网点数 |
| `service_network.description` | text | **系统** | 服务网络描述 |
| `support_features` | list<string> | **系统** | 3-4 个 Tag |

---

## 七、主要竞品（段 07）

> 系统给**结构化数据**（车名/起售价/车图），**AI 写 pros/cons**（≤ 60 字）。

```yaml
competitors:
  - { tag: "SELF",        name: "Nissan Versa",  price: 309990, image_url: "...", pros: "Mejor precio · cajuela 482L · 6 airbags std", cons: "Ruido carretera · garantía 3 años" }
  - { tag: "ALTERNATIVA", name: "VW Virtus",     price: 322490, image_url: "...", pros: "...", cons: "..." }
  - { tag: "ALTERNATIVA", name: "Kia K3 Sedán",  price: 304900, image_url: "...", pros: "...", cons: "..." }
  - { tag: "PREMIUM",     name: "Mazda 2 Sedán", price: 301900, image_url: "...", pros: "...", cons: "..." }
```

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `tag` | enum | **系统** | `SELF` / `ALTERNATIVA` / `PREMIUM` |
| `name` | string | **系统** | 车名 |
| `price` | int | **系统** | 起售价 |
| `image_url` | url | **系统** | 车图 CDN |
| `pros` | text | **AI** | 优点（≤ 60 字，**不许编数字**）|
| `cons` | text | **AI** | 缺点（≤ 60 字，**不许编数字**）|

**结构固定 4 条**：1 条本车 + 3 条竞品。

---

## 八、行动入口（段 08）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `type` | enum | **系统** | `whatsapp` / `test_drive` / `cavi_ai` |
| `label` | string | **系统** | 卡片标题（西语）|
| `sub` | text | **系统** | 卡片副标题 |
| `url` | url | **系统** | 跳转链接 |

---

## 九、首屏（段 0）

### 9.1 Spec strip（3 核心数据）

```yaml
spec_strip:
  - { label: "Mejor consumo", value: "18.81", unit: "km/L",  sub: "SENSE MT" }
  - { label: "Cajuela",       value: "482",   unit: "L",     sub: "2 maletas + equipaje" }
  - { label: "Ventas julio",  value: "7,486", unit: "unid.", sub: "sedán #1 en México" }
```

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `label` | string | **系统** | 数据标签 |
| `value` | string | **系统** | 数据值（字符串以保留格式）|
| `unit` | string | **系统** | 单位 |
| `sub` | string | **系统** | 副说明 |
| `icon` | enum | **系统** | `fuel` / `trunk` / `trending` |

**固定 3 条**（首屏 3 卡的核心数据）。

### 9.2 Version selector（6 个版本）

```yaml
versions:
  - { code: "sense-base-mt", name: "SENSE BASE MT", msrp: 309990, official: 374900, monthly: 5162, is_recommended: false }
  - { code: "sense-mt",      name: "SENSE MT",      msrp: 316990, official: 382900, monthly: 5278, is_recommended: false }
  - { code: "sense-cvt",     name: "SENSE CVT",     msrp: 341990, official: 406900, monthly: 5694, is_recommended: false }
  - { code: "advance-mt",    name: "ADVANCE MT",    msrp: 363990, official: 428900, monthly: 6061, is_recommended: false }
  - { code: "advance-cvt",   name: "ADVANCE CVT",   msrp: 374990, official: 439900, monthly: 6244, is_recommended: true }
  - { code: "exclusive-cvt", name: "EXCLUSIVE CVT", msrp: 406990, official: 470900, monthly: 6777, is_recommended: false }
```

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `code` | string | **系统** | 版本 code |
| `name` | string | **系统** | 版本名 |
| `msrp` | int | **系统** | 经销价 |
| `official` | int | **系统** | 官方价（≥ msrp）|
| `monthly` | int | **系统** | 估算月供 |
| `is_recommended` | bool | **系统** | 恰好 1 个 true |

### 9.3 Hero CTA

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `text` | string | **系统** | `Consultar planes` |
| `url` | url | **系统** | finance apply 链接 |
| `style` | enum | **系统** | `yellow-block` |

---

## 十、关键约束总结

| 约束 | 说明 |
|------|------|
| **段 02..08 必须有** | 8 段是结构骨架，缺一段视为生成失败 |
| **西语 eyebrow 字面锁定** | `02 · PRECIO Y FINANCIAMIENTO` 等 7 个字符串不能改 |
| **AI 不得编数字** | 价格/销量/评分/里程等系统字段 AI 不可触碰（见 §〇）|
| **跨字段一致** | cavi_score 在 frontmatter / Hero / 段 04 必须一致 |
| **价格是整数** | 不要带 `$`/`MXN`/`，` 千分位（模板渲染时加）|
| **slug 命名规范** | `snake_case` |
| **数据快照必存** | 每次生成保留 fetched_at + cache_ttl 用于回放 |

---

*相关文档：*
- *总览：`docs/MD-生成-总览.md`*
- *数据来源：`docs/MD-生成-数据源与可靠性.md`*
- *质量保证：`docs/MD-生成-质量保证.md`*
- *AI 骨架：`skills/cavi-guide-gen/assets/standard-template-v3.md`*
