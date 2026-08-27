# MD 数据需求清单

**文档版本**：v1.1
**创建日期**：2026-08-27
**修订日期**：2026-08-27（v1.1：删除"模板族"概念，改为单一展现样式 + 设备载体）
**目的**：MD 生成涉及的所有字段的**权威定义表**——SKILL、模板、质量保证文档都以此为准
**约定**：
- `系统` = 来自 CAVI API/销售/银行/评论库（结构化数据，不需 AI）
- `AI` = 由 AI 生成（自然语言，受风格规范约束）
- `混合` = 系统给原始值 + AI 润色

> **项目只有 1 个展现样式**（西语），不支持多模板族。详见 `docs/MD-生成-展现样式规范.md`。

---

## 一、Frontmatter 字段

文件顶部 YAML 段，**所有字段必填**（除非标注"可选"）。

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `title` | string | AI | 车名 + 年款，如 `Nissan Versa 2026 购车指南` | `Nissan {model} {year} 购车指南` |
| `series_id` | int | 系统 | CAVI 系统车系 ID，如 `356` | **必填，无兜底** |
| `model` | string | 系统 | 车型名（小写、连字符），如 `nissan-versa` | **必填，无兜底** |
| `year` | int | 系统 | 年款，4 位数字，如 `2026` | **必填，无兜底** |
| `market` | enum | 系统 | 市场代码：`MX` / `CN` / `CO` / `AR` | **必填，无兜底** |
| `lang` | enum | 系统 | 语言代码：`es`（当前唯一支持）| **必填，无兜底** |
| `energy_type` | enum | 系统 | `燃油` / `混动` / `纯电` / `插混` | **必填，无兜底** |
| `body_type` | string | 系统 | `Sedán` / `SUV` / `Hatchback` / `Pickup` | **必填，无兜底** |
| `device` | enum | 系统 | 渲染设备：`pc` / `h5`（决定加载哪个 HTML 模板）| `pc` |
| `generated_at` | date | 系统 | 生成时间（ISO 8601）| 自动填 |
| `source` | string | 系统 | 数据来源标识 | `CAVI (AutoCava AI)` |
| `cavi_score` | decimal | 系统 | CAVI 综合评分 0-5 | `null` + 兜底文案 |
| `cavi_recommend` | int | 系统 | 推荐率（%）0-100 | `null` |
| `cavi_verdict` | string | 混合 | 一句洞察（≤ 30 字）| `"Sin datos suficientes"` |
| `spec_strip` | list<obj> | 系统 | 首屏 3 核心数据 | `[]` + 跳过此段 |
| `key_specs` | list<obj> | 系统 | 卖点章 3-4 核心数据 | `[]` + 跳过此段 |
| `annual_cost` | object | 系统 | 年度用车成本（详细见 §五）| `null` + 跳过此段 |
| `finance_cards` | object | 系统 | 4 张金融卡（详细见 §三）| `null` + 跳过段 02 |
| `competitors` | list<obj> | 系统 | 4 张并排卡含本车（详细见 §六）| `[]` + 跳过此段 |
| `next_cards` | list<obj> | 系统 | 3 张行动卡（详细见 §七）| `[]` + 跳过此段 |

---

## 二、Hero 字段

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `hero.eyebrow` | string | 系统 | 顶部小标签，如 `EXPERTO · CAVI` | `EXPERTO · CAVI` |
| `hero.title` | string | 系统 | 车名 + 年款 | `<model_name> {year}` |
| `hero.subtitle` | string | AI | 一句副标题（≤ 50 字）| `¿Vale la pena?` |
| `hero.desc` | text | AI | 描述段 50-80 字（含 CAVI 评分、销量等关键数）| `"Sin datos suficientes"` |
| `hero.image_url` | url | 系统 | 车图 CDN URL | `null` + 隐藏车图 |
| `hero.cavi_score` | decimal | 系统 | CAVI 评分（同 frontmatter，但显式）| `null` + 隐藏评分徽章 |
| `hero.monthly_sales` | int | 系统 | 月销量 | `null` |
| `hero.rank_text` | string | AI | 销量排名描述，如 `sedán #1 en México` | `null` |

---

## 三、价格与金融方案（段 02）

### 3.1 `finance_cards.main`（主价卡 · 深色）

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `version_name` | string | 系统 | 推荐版本名，如 `ADVANCE CVT` | **必填** |
| `price` | int | 系统 | 推荐版本 MSRP（整数 MXN）| **必填** |
| `currency` | enum | 系统 | 货币代码：当前 `MXN` | `MXN` |
| `source` | string | 系统 | 经销价来源 | `precio de lista {品牌} {市场}` |
| `price_base` | int | 系统 | 基础价（同 price）| = price |
| `bonus_trade_in` | int | 系统 | 交换补贴（负数）| `0` |
| `price_effective` | int | 系统 | 实际价 = base + bonus | 计算得出 |

### 3.2 `finance_cards.bank`（BBVA 等银行方案 · 卡 2）

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `bank_name` | string | 系统 | 银行名 | `BBVA` |
| `term` | int | 系统 | 期限（月）| **必填** |
| `monthly` | int | 系统 | 月供 | **必填** |
| `rate` | decimal | 系统 | 年利率（%），如 `12.9` | **必填** |
| `loan_ratio` | int | 系统 | 贷款比例（%），如 `70` | **必填** |
| `down_payment` | int | 系统 | 首付金额 | 计算 = price × (1 - loan_ratio) |
| `loan_amount` | int | 系统 | 贷款额 | 计算 = price - down_payment |
| `total_interest` | int | 系统 | 总利息 | 计算 = monthly × term - loan_amount |

### 3.3 `finance_cards.factory`（厂家金融 · 卡 3）

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `provider` | string | 系统 | 厂家名，如 `Nissan` | **必填** |
| `term` | int | 系统 | 期限（月）| **必填** |
| `monthly` | int | 系统 | 月供 | **必填** |
| `rate` | string | 系统 | 利率描述，如 `0%` | **必填** |
| `tag` | string | 系统 | 利率标签，如 `0% primer año` | `null` |
| `plan` | string | 系统 | 计划名 | `null` |
| `cat` | string | 系统 | CAT（如 `14.8% sin IVA`）| `null` |
| `down_payment` | int | 系统 | 首付金额 | 计算 |

### 3.4 `finance_cards.trade_in`（交换补贴 · 卡 4）

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `amount` | int | 系统 | 补贴金额 | **必填** |
| `condition` | string | 系统 | 条件说明 | `Si entregas tu auto actual` |
| `vigencia` | date | 系统 | 截止日期 | `null` |
| `applies_to` | string | 系统 | 适用版本 | `null` |
| `extra_bono` | int | 系统 | 额外补贴（可负）| `0` |

### 3.5 `finance_cta_bar`（章节内 cta-bar）

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `title` | string | AI | 标题（≤ 20 字）| `¿Calcular tu cuota?` |
| `subtitle` | string | AI | 副标题 | `Compara hasta 3 bancos · 1 min` |
| `button_text` | string | 系统 | 按钮文字 | `Calcular →` |
| `url` | url | 系统 | 按钮链接 | `https://www.autocava.com.mx/finance/apply?...` |

---

## 四、CAVI 评分（段 04）

### 4.1 `cavi_score` 顶级字段（已在 frontmatter）

详见 §一。

### 4.2 `cavi_dimensions`（4 维度细分）

**结构**：4 个对象，每个含 slug / label / value / stars / 可选 is_weakness。

```yaml
cavi_dimensions:
  - { slug: cajuela,   label: "Cajuela",   value: 4.8, stars: 5 }
  - { slug: consumo,   label: "Consumo",   value: 4.6, stars: 5 }
  - { slug: seguridad, label: "Seguridad", value: 4.5, stars: 4 }
  - { slug: ruido,     label: "Ruido",     value: 3.9, stars: 3, is_weakness: true }
```

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `slug` | enum | 系统 | 维度标识符 | **必填** |
| `label` | string | 系统 | 显示标签（西语，与 `lang=es` 对齐）| **必填** |
| `value` | decimal | 系统 | 维度评分 0-5 | **必填** |
| `stars` | int | 系统 | 星级 1-5 | `round(value)` |
| `is_weakness` | bool | 系统 | 是否短板（用于红色高亮）| `false` |

**当前展现样式的 4 维度**（燃油车 Sedan 类）：

| slug | 西语 label |
|------|----------|
| `cajuela` | Cajuela（后备厢） |
| `consumo` | Consumo（油耗） |
| `seguridad` | Seguridad（安全） |
| `ruido` | Ruido（噪音） |

> **未来扩展**（其他车型类别）：PM 按车型类别评审 4 维度定义。例如电动车 SUV 可能改为"外观/内饰/智驾/售后"。新增维度定义需要 PM 评审。

### 4.3 `featured_reviews`（精选评论 2-3 条）

```yaml
featured_reviews:
  - { stars: 5, content: "Llevo casi un año y consumo 18 km/L...", version: "2024 ADVANCE CVT", mileage_km: 15000, location: "CDMX" }
```

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `stars` | int | 系统 | 星级 1-5 | **必填** |
| `content` | text | 系统 | 评论内容（≤ 100 字）| **必填** |
| `version` | string | 系统 | 评论者版本 | **必填** |
| `mileage_km` | int | 系统 | 行驶里程 | `null` |
| `location` | string | 系统 | 评论者城市 | `null` |

### 4.4 `cavi_meta`（元信息）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `review_count` | int | 系统 | 评论人数 |
| `monthly_sales` | int | 系统 | 月销量 |
| `rank_text` | string | AI | 排名描述 |

---

## 五、用车成本（段 05）

```yaml
annual_cost:
  total: 17300           # 年度总成本（整数，本地货币）
  currency: "MXN"
  monthly: 1442          # = total / 12
  items:
    - { slug: fuel,        label_es: "Combustible", label_zh: "燃油",       amount: 8000, pct: 46 }
    - { slug: insurance,   label_es: "Seguro",      label_zh: "保险",       amount: 4000, pct: 23 }
    - { slug: maintenance, label_es: "Mantenimiento",label_zh: "保养",      amount: 2500, pct: 14 }
    - { slug: depreciation,label_es: "Depreciación",label_zh: "折旧",      amount: 2000, pct: 12 }
    - { slug: tax,         label_es: "Tenencia",    label_zh: "购置税",     amount: 800,  pct: 5  }
  assumptions:
    annual_km: 15000
    fuel_price: 24.5     # 本地油价
  saving_tips:
    - "Mantén presión de neumáticos en 32 PSI — ahorra 3% combustible"
    - "Compara seguros en línea cada año — diferencia típica 15%"
```

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `total` | int | 系统 | 年度总成本 | **必填** |
| `currency` | enum | 系统 | 货币 | `MXN` |
| `monthly` | int | 计算 | 月度成本 | = total/12 |
| `items` | list<5> | 系统 | 5 项明细 | 5 项必填 |
| `items[].amount` | int | 系统 | 金额 | **必填** |
| `items[].pct` | int | 计算 | 占比（%）| = amount/total×100 |
| `assumptions.annual_km` | int | 系统 | 年行驶里程假设 | `15000` |
| `saving_tips` | list<string> | AI | 省钱建议 2-3 条 | `[]` |

**5 项必含 slug**（顺序固定，模板按此渲染）：
1. `fuel`
2. `insurance`
3. `maintenance`
4. `depreciation`
5. `tax`

---

## 六、购车保障（段 06）

```yaml
warranty:
  years: 3
  mileage_km: 100000
  details: "Cobertura nacional en 1,500+ agencias"
maintenance:
  included_count: 12
  mileage_km: 24000
service_network:
  count: 1500
  description: "Agencias Nissan en toda la república"
support_features:
  - "Garantía total"
  - "Asistencia vial 24h"
  - "Software OTA"
  - "Red de agencias"
```

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `warranty.years` | int | 系统 | 整车质保年限 | **必填** |
| `warranty.mileage_km` | int | 系统 | 整车质保里程 | **必填** |
| `warranty.details` | text | 系统 | 质保细节 | `null` |
| `maintenance.included_count` | int | 系统 | 保养包次数 | **必填** |
| `maintenance.mileage_km` | int | 系统 | 保养包里程 | **必填** |
| `service_network.count` | int | 系统 | 网点数 | `0` |
| `service_network.description` | text | 系统 | 服务网络描述 | `null` |
| `support_features` | list<string> | 系统 | 支持项目 3-4 个 Tag | `[]` |

---

## 七、主要竞品（段 07）

```yaml
competitors:
  - { tag: "SELF",         name: "Nissan Versa",   price: 309990, image_url: "...", pros: "Mejor precio · cajuela 482L · 6 airbags std", cons: "Ruido carretera · garantía 3 años" }
  - { tag: "ALTERNATIVA",  name: "VW Virtus",      price: 322490, image_url: "...", pros: "Calidad alemana · 126 HP",                   cons: "Consumo mayor · ensamble India" }
  - { tag: "ALTERNATIVA",  name: "Kia K3 Sedán",   price: 304900, image_url: "...", pros: "Garantía 5 años · 8 versiones",               cons: "Resale inferior" }
  - { tag: "PREMIUM",      name: "Mazda 2 Sedán",  price: 301900, image_url: "...", pros: "Skyactiv KODO · ensamble MX",                 cons: "Cajuela 440L · garantía 3 años" }
```

**结构固定 4 条**：1 条本车 + 3 条竞品。

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `tag` | enum | 系统 | `SELF` / `ALTERNATIVA` / `PREMIUM` | **必填** |
| `name` | string | 系统 | 车名 | **必填** |
| `price` | int | 系统 | 起售价 | **必填** |
| `image_url` | url | 系统 | 车图 CDN | `null` |
| `pros` | text | AI | 优点（≤ 60 字）| `"Sin datos"` |
| `cons` | text | AI | 缺点（≤ 60 字）| `"Sin datos"` |

**tag 规则**：
- 第 1 条固定 `SELF`（本车）
- 后续 3 条至少 1 条 `ALTERNATIVA`
- 价格段 ±10% 内为 `ALTERNATIVA`，更贵或越级为 `PREMIUM`

---

## 八、行动入口（段 08）

```yaml
next_cards:
  - { type: "whatsapp",   label: "Hablar con un asesor",   sub: "WhatsApp directo · < 5 min respuesta",  url: "https://wa.me/525527419019" }
  - { type: "test_drive", label: "Agendar prueba de manejo", sub: "280+ distribuidores en México",       url: "https://www.autocava.com.mx/test-drive" }
  - { type: "cavi_ai",    label: "Preguntar a Cavi (AI)",    sub: "Asistente inteligente 24/7",          url: "https://www.autocava.com.mx/cavi" }
```

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `type` | enum | 系统 | `whatsapp` / `test_drive` / `cavi_ai` | **必填** |
| `label` | string | 系统 | 卡片标题（西语：`Hablar con un asesor` / `Agendar prueba de manejo` / `Preguntar a Cavi (AI)`）| **必填** |
| `sub` | text | 系统 | 卡片副标题 | `null` |
| `url` | url | 系统 | 点击跳转链接 | **必填** |

**type 必含**：3 条必须各覆盖 1 个 type，不能重复。

---

## 九、首屏（段 0）

### 9.1 Spec strip（3 核心数据）

```yaml
spec_strip:
  - { label: "Mejor consumo",      value: "18.81", unit: "km/L",  sub: "SENSE MT",  icon: "fuel" }
  - { label: "Cajuela",            value: "482",   unit: "L",     sub: "2 maletas + equipaje",  icon: "trunk" }
  - { label: "Ventas julio",       value: "7,486", unit: "unid.", sub: "sedán #1 en México",  icon: "trending" }
```

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `label` | string | 系统 | 数据标签 | **必填** |
| `value` | string | 系统 | 数据值（字符串以保留格式）| **必填** |
| `unit` | string | 系统 | 单位 | `null` |
| `sub` | string | 系统 | 副说明 | `null` |
| `icon` | enum | 系统 | 图标：`fuel` / `trunk` / `trending` / `power` / `price` / `sales` | `null` |

**spec_strip 固定 3 条**（这是首屏 3 卡的核心数据）。

### 9.2 Version selector（6 个版本）

```yaml
versions:
  - { code: "sense-base-mt", name: "SENSE BASE MT",    msrp: 309990, official: 374900, monthly: 5162, is_recommended: false }
  - { code: "sense-mt",      name: "SENSE MT",         msrp: 316990, official: 382900, monthly: 5278, is_recommended: false }
  - { code: "sense-cvt",     name: "SENSE CVT",        msrp: 341990, official: 406900, monthly: 5694, is_recommended: false }
  - { code: "advance-mt",    name: "ADVANCE MT",       msrp: 363990, official: 428900, monthly: 6061, is_recommended: false }
  - { code: "advance-cvt",   name: "ADVANCE CVT",      msrp: 374990, official: 439900, monthly: 6244, is_recommended: true }
  - { code: "exclusive-cvt", name: "EXCLUSIVE CVT",    msrp: 406990, official: 470900, monthly: 6777, is_recommended: false }
```

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `code` | string | 系统 | 版本内部 code | **必填** |
| `name` | string | 系统 | 版本显示名 | **必填** |
| `msrp` | int | 系统 | 经销价（整数本地货币）| **必填** |
| `official` | int | 系统 | 官方价（不含优惠）| = msrp |
| `monthly` | int | 系统 | 估算月供 | `0` |
| `is_recommended` | bool | 系统 | 是否推荐（1 个 true）| `false` |

**规则**：
- 6 个版本固定
- 恰好 1 个 `is_recommended: true`
- `official ≥ msrp`（优惠不能为负）

### 9.3 Compare bar

```yaml
compare_bar:
  - { name: "VW Virtus",  url: "..." }
  - { name: "Kia K3",     url: "..." }
  - { name: "Mazda 2",    url: "..." }
```

### 9.4 Hero CTA

```yaml
hero_cta:
  text: "Consultar planes"
  url: "https://www.autocava.com.mx/finance/apply?sourceType=1&seriesId=356&brandId=49"
  style: "yellow-block"
```

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `text` | string | 系统 | 按钮文字（西语：`Consultar planes`）|
| `url` | url | 系统 | 直链，参数含 seriesId + brandId |
| `style` | enum | 系统 | `yellow-block`（当前唯一）|

---

## 十、Fixed bottom + 配套全局元素

不属于 8 段内容，但前端模板必备（**全局元素，不写入 MD 文档**）：

| 元素 | 字段 | 类型 | 来源 |
|------|------|------|------|
| Fixed bottom WhatsApp | `whatsapp_number` | string | 系统（市场级配置）|
| Fixed bottom Llamar | `phone_number` | string | 系统（市场级配置）|
| Sub nav | `sub_nav_links` | list | 系统（模板族配置）|
| Footer | `footer_text` | string | 系统 |

> 这些由前端模板自带 + 市场级配置决定，**不进入 MD 文档**。

---

## 十一、关键约束总结

| 约束 | 说明 |
|------|------|
| **段 02..08 必须有** | 8 段是结构骨架，缺一段视为生成失败 |
| **西语 eyebrow 字面锁定** | `02 · PRECIO Y FINANCIAMIENTO` 等 7 个字符串不能改 |
| **4 维度由 PM 评审** | 不同车型类别（燃油 Sedan / 电动 SUV）可换 4 维度；新增需 PM 评审 |
| **必填字段缺失 = 失败** | 不允许兜底到 `"Sin datos"` 后还继续生成（除非该段标记为"可选"）|
| **跨字段一致性** | `cavi_score` 在 frontmatter、Hero、段 04 必须一致 |
| **价格是整数** | 不要带 `$`/`MXN`/`，` 千分位（这些由模板渲染时加）|
| **slug 命名规范** | 用 `snake_case` |

---

*相关文档：*
- *字段权威定义：本文件*
- *数据来源：见 §字段表"来源"列 · 详细策略见 `docs/MD-生成-数据源与可靠性.md`*
- *质量验收：`docs/MD-生成-质量保证.md`*
- *AI 骨架：`skills/cavi-guide-gen/assets/standard-template-v3.md`*
