# CAVI 购车手册标准模板 v3.0

**版本**：3.0
**更新日期**：2026-08-27
**用途**：AI 生成 MD 文档的标准骨架，映射前端 HTML 8 段结构

> v3.0 升级：章节数从 9 Tab 减为 **8 段**（首屏 + 02..08 七段内容），章节命名对齐西语 `block-eyebrow` 标签。详见 `docs/CAVI 报告式页面重构方案.md` v2.0。

---

## 设计原则

1. **章节固定**：前端按 8 段顺序渲染，必须有对应 MD 章节
2. **变量占位符**：`{{变量名}}` 格式，AI 填充实际数据
3. **可选区块**：用 `{{optional_xxx}}` 标记无数据时可删除
4. **禁止修改**：章节顺序、层级、变量名不可变更
5. **西语标签**：每个内容段都有固定西语 `block-eyebrow`，前端按字面渲染

---

## Frontmatter 规范

```yaml
---
title: Nissan Versa 2026 车辆解读报告
series_id: 356
model: nissan-versa
year: 2026
market: MX
lang: zh
energy_type: 燃油
body_type: Sedán
generated_at: 2026-08-27
source: CAVI (AutoCava AI)

# CAVI v2.0 新增字段
cavi_score: 4.6           # CAVI 综合指数
cavi_recommend: 89        # 推荐率（%）
cavi_verdict: Balance precio/espacio/seguridad inmejorable
cavi_dimensions:          # 4 维度细分
  cajuela: 4.8
  consumo: 4.6
  seguridad: 4.5
  ruido: 3.9
spec_strip:               # 首屏 3 核心数据
  - { label: 马力, value: "118 HP" }
  - { label: 油耗, value: "5.3 L/100km" }
  - { label: 月销量, value: "7,486" }
key_specs:                # 卖点章 3-4 核心数据
  - { label: 马力, value: "118 HP" }
  - { label: 油耗, value: "5.3 L/100km" }
  - { label: 后备厢, value: "480 L" }
  - { label: 中控屏, value: "7 寸" }
annual_cost: 17300
finance_cards:
  main: { price: 374990, currency: "MXN", source: "distribuidor" }
  bank: { monthly: 6244, term: 36, rate: "12.9%", loan_ratio: "70%" }
  factory: { monthly: 5899, term: 48, rate: "0%", tag: "0% primer año" }
  trade_in: 15000
competitors:
  - { name: "VW Virtus", price: 322490, tag: "ALTERNATIVA", pro: "Manejo alemán", con: "Consumo mayor" }
  - { name: "Kia K3 Sedán", price: 304900, tag: "ALTERNATIVA", pro: "Garantía 5 años", con: "Resale inferior" }
  - { name: "Mazda 2 Sedán", price: 301900, tag: "PREMIUM", pro: "Skyactiv KODO", con: "Cajuela 440L" }
next_cards:
  - { type: whatsapp, label: "Hablar con un asesor", sub: "< 5 min respuesta" }
  - { type: test_drive, label: "Agendar prueba de manejo", sub: "280+ distribuidores" }
  - { type: cavi_ai, label: "Preguntar a Cavi (AI)", sub: "Asistente 24/7" }
---
```

---

## 段 0：首屏（Hero + Spec strip + Trim selector + Compare bar）

> 无 `block-eyebrow`，作为 Hero 区直接渲染。

```
# {{title}}

> **{{one_liner}}**

---

## 身份识别

| 维度 | 数据 |
|------|------|
| 品牌 | {{brand}} |
| 车系 | {{series}} |
| 年款 | {{year}} |
| 能源类型 | {{energy_type}} |
| 车身形式 | {{body_type}} |
| 月销量 | {{monthly_sales}} |

### 用户价值定位

!!! success "适合人群"
{{target_users}}

---

## Spec strip（首屏 3 核心数据）

| 核心数据 | 数值 |
|----------|------|
{{#each spec_strip}}
| {{this.label}} | **{{this.value}}** |
{{/each}}

---

## Trim selector（版本选择）

{{trim_selector_data}}

---

## Compare bar（对比条）

{{compare_bar_data}}

---

## Hero CTA

> [Consultar planes]({{finance_apply_url}}) — 黄底按钮，直链 finance apply
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{one_liner}}` | string | 一句话定位 |
| `{{brand}}` | string | 品牌名 |
| `{{series}}` | string | 车系名 |
| `{{year}}` | number | 年款 |
| `{{energy_type}}` | string | 能源类型 |
| `{{body_type}}` | string | 车身形式 |
| `{{monthly_sales}}` | string | 月销量 |
| `{{target_users}}` | list | 适合人群列表 |
| `{{spec_strip}}` | list | 3 核心数据（label/value） |
| `{{trim_selector_data}}` | block | 版本选择数据 |
| `{{compare_bar_data}}` | block | 对比条数据 |
| `{{finance_apply_url}}` | string | finance apply 直链 |

---

## 段 02：价格与金融方案（PRECIO Y FINANCIAMIENTO）

> Frontmatter `block-eyebrow`: `02 · PRECIO Y FINANCIAMIENTO`
> Frontmatter `block-title`: `Precio y planes`

```
## 02 · PRECIO Y FINANCIAMIENTO

> Precio y planes

### 主价卡（distribuidor）

| 项目 | 金额 |
|------|------|
| 经销价 | **{{finance_cards.main.price}} {{finance_cards.main.currency}}** |
| 经销来源 | {{finance_cards.main.source}} |
| Precio base | {{price_base}} |
| Bono intercambio | −{{bonus_trade_in}} |
| Precio efectivo | {{price_effective}} |

### 3 补充卡

#### {{finance_cards.bank.monthly}}/月 · {{finance_cards.bank.term}} 期（BBVA 等银行方案）

| 项目 | 数值 |
|------|------|
| 月供 | **{{finance_cards.bank.monthly}} {{finance_cards.main.currency}}** |
| 期数 | {{finance_cards.bank.term}} 个月 |
| 利率 | {{finance_cards.bank.rate}} APR |
| 贷款比例 | {{finance_cards.bank.loan_ratio}} |

#### {{finance_cards.factory.monthly}}/月 · 厂家金融

| 项目 | 数值 |
|------|------|
| 月供 | **{{finance_cards.factory.monthly}} {{finance_cards.main.currency}}** |
| 期数 | {{finance_cards.factory.term}} 个月 |
| 利率 | {{finance_cards.factory.rate}}（首年贴息） |
| 标签 | {{finance_cards.factory.tag}} |

#### Bono intercambio

| 项目 | 数值 |
|------|------|
| 交换补贴 | **−{{finance_cards.trade_in}} {{finance_cards.main.currency}}** |
| 条件 | Si entregas tu auto actual |

### cta-bar（章节内 CTA）

> **¿Calcular tu cuota?** · Compara hasta 3 bancos · 1 min
> [Calcular →]({{finance_apply_url}})
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{price_base}}` | string | 基础价 |
| `{{bonus_trade_in}}` | number | 交换补贴 |
| `{{price_effective}}` | string | 实际价 |
| `{{finance_cards}}` | object | 4 个金融卡详情 |
| `{{finance_apply_url}}` | string | finance apply 链接 |

---

## 段 03：核心卖点（CORE SELLING POINTS）

> Frontmatter `block-eyebrow`: `03 · CORE SELLING POINTS`
> Frontmatter `block-title`: `Lo que destaca`

```
## 03 · CORE SELLING POINTS

> Lo que destaca

### 3-4 个核心数据指标

{{#each key_specs}}
- **{{this.value}}** {{this.label}}
{{/each}}

### 1 段洞察描述

> "{{insight_quote}}" — CAVI
>
> {{insight_description}}

{{optional_user_impression}}
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{key_specs}}` | list | 3-4 核心数据（label/value） |
| `{{insight_quote}}` | string | 洞察原话 |
| `{{insight_description}}` | string | 50-80 字洞察描述 |
| `{{optional_user_impression}}` | block | 可选：试驾印象 |

---

## 段 04：用户口碑评价（CAVI · RESEÑAS）

> Frontmatter `block-eyebrow`: `04 · CAVI · RESEÑAS`
> Frontmatter `block-title`: `Reseñas de usuarios`

```
## 04 · CAVI · RESEÑAS

> Reseñas de usuarios

### CAVI 综合评分

> **CAVI {{cavi_score}} / 5.0** · {{cavi_recommend}}% recomienda
>
> "{{cavi_verdict}}" — CAVI
>
> Sobresale: **{{cavi_dimensions.cajuela}} cajuela**, **{{cavi_dimensions.consumo}} consumo**, **{{cavi_dimensions.seguridad}} seguridad**. Áreas: ruido ({{cavi_dimensions.ruido}}).

### 元信息

| 指标 | 数值 |
|------|------|
| 评论人数 | {{review_count}} propietarios |
| 月销量 | {{monthly_sales}} ventas |
| 排名 | {{rank_in_segment}} |

### 4 维度细分

| 维度 | 评分 | 星级 |
|------|------|------|
| Cajuela | {{cavi_dimensions.cajuela}} | ★★★★★ |
| Consumo | {{cavi_dimensions.consumo}} | ★★★★★ |
| Seguridad | {{cavi_dimensions.seguridad}} | ★★★★☆ |
| Ruido | {{cavi_dimensions.ruido}} | ★★★☆☆（短板）|

### 精选评论

{{#each featured_reviews}}
> "{{this.content}}"
> — {{this.version}} · {{this.mileage}} km · {{this.location}}

{{/each}}
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{cavi_score}}` | number | CAVI 综合分 |
| `{{cavi_recommend}}` | number | 推荐率（%） |
| `{{cavi_verdict}}` | string | 一句结论 |
| `{{cavi_dimensions}}` | object | 4 维度评分 |
| `{{review_count}}` | number | 评论人数 |
| `{{monthly_sales}}` | number | 月销量 |
| `{{rank_in_segment}}` | string | 细分市场排名 |
| `{{featured_reviews}}` | list | 精选评论 2-3 条 |

---

## 段 05：用车成本（COST）

> Frontmatter `block-eyebrow`: `05 · COST`
> Frontmatter `block-title`: `Costo anual`

```
## 05 · COST

> Costo anual

按年行驶 {{annual_km}} km 计算

### 年度用车成本

> **{{annual_cost}}/年** （折合 {{monthly_cost}}/月）

### 5 项并列明细

| 项目 | 金额 | 占比 |
|------|------|------|
| 燃油费 | {{annual_fuel}} | {{fuel_pct}}% |
| 保险费 | {{annual_insurance}} | {{insurance_pct}}% |
| 维护费 | {{annual_maintenance}} | {{maintenance_pct}}% |
| 折旧 | {{annual_depreciation}} | {{depreciation_pct}}% |
| 购置税 | {{annual_tax}} | {{tax_pct}}% |

### 省钱建议

{{cost_saving_tips}}
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{annual_km}}` | number | 年行驶里程 |
| `{{annual_cost}}` | number | 年度总成本 |
| `{{monthly_cost}}` | number | 月度成本 |
| `{{annual_fuel}}` 等 | number | 5 项金额 |
| `{{fuel_pct}}` 等 | number | 5 项占比（%） |
| `{{cost_saving_tips}}` | list | 2-3 条省钱建议 |

---

## 段 06：购车保障与服务覆盖（PROTECTION）

> Frontmatter `block-eyebrow`: `06 · PROTECTION`
> Frontmatter `block-title`: `Cobertura post-compra`

```
## 06 · PROTECTION

> Cobertura post-compra

### 整车质保

> **{{warranty_years}} 年 / {{warranty_mileage}} km**

{{warranty_details}}

### 保养包含

> **{{maintenance_count}} 次 / {{maintenance_mileage}} km**

### 服务网络

> 覆盖 **{{service_network_count}}+** 网点

### 支持项目

{{#each support_features}}
- {{this}}
{{/each}}
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{warranty_years}}` | number | 质保年限 |
| `{{warranty_mileage}}` | number | 质保里程 |
| `{{warranty_details}}` | block | 质保细节 |
| `{{maintenance_count}}` | number | 保养次数 |
| `{{maintenance_mileage}}` | number | 保养里程 |
| `{{service_network_count}}` | number | 网点数 |
| `{{support_features}}` | list | 3-4 项 Tag |

---

## 段 07：主要竞品怎么选（COMPETITORS）

> Frontmatter `block-eyebrow`: `07 · COMPETITORS`
> Frontmatter `block-title`: `¿Qué comparar?`

```
## 07 · COMPETITORS

> ¿Qué comparar?

### 3 张并排卡

{{#each competitors}}
#### {{this.tag}} · {{this.name}}

> Desde **{{this.price}}** MXN

- **+** {{this.pro}}
- **−** {{this.con}}

{{/each}}
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{competitors}}` | list | 3 个竞品（name/price/tag/pro/con） |
| `{{tag}}` | enum | `ALTERNATIVA`（同级）/ `PREMIUM`（越级） |

---

## 段 08：想进一步了解？（SIGUIENTE PASO）

> Frontmatter `block-eyebrow`: `08 · SIGUIENTE PASO`
> Frontmatter `block-title`: `¿Listo para decidir?`

```
## 08 · SIGUIENTE PASO

> ¿Listo para decidir?
>
> Elige cómo continuar

### 3 张 next-card

{{#each next_cards}}
#### {{this.label}}

> {{this.sub}}

[Continuar →]({{this.url}})

{{/each}}
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{next_cards}}` | list | 3 张 next-card（type/label/sub/url） |
| 预期 3 个 type | enum | `whatsapp` / `test_drive` / `cavi_ai` |

---

## 配套全局元素（不属于 8 段，但所有报告必备）

### Hero CTA

> [Consultar planes]({{finance_apply_url}}) — 黄底按钮，直链 finance apply

### Fixed bottom

> 窄 WhatsApp 图标（56px）+ 橙色 "Llamar" 按钮

### Footer

```
*📊 数据来源：[AutoCava CAVI](https://www.autocava.com.mx/cavi) · {{series}} Series ID: {{series_id}} · 生成时间: {{generated_at}}*
```

---

## 可选区块说明

| 区块 | 段 | 说明 | 使用条件 |
|------|-----|------|----------|
| `{{optional_user_impression}}` | 03 | 试驾印象 | 有用户高频提到的话时保留 |
| `{{sales_data_section}}` | 00 | 销量数据 | 有月销量时保留 |

---

## 生成检查清单

- [ ] 所有 `{{变量}}` 已替换为实际值
- [ ] Frontmatter 完整（含 cavi_score / cavi_recommend / cavi_dimensions / spec_strip / key_specs / finance_cards / competitors / next_cards）
- [ ] **8 段**章节齐全（首屏 + 02..08）
- [ ] 每个内容段都有正确的西语 `block-eyebrow`
- [ ] 4 卡价格（主价卡 + 3 补充卡）+ cta-bar
- [ ] 4 维度 CAVI 细分（含 1 个短板）
- [ ] 3 张并排竞品卡（ALTERNATIVA / PREMIUM 标签）
- [ ] 3 张 next-card（whatsapp / test_drive / cavi_ai）
- [ ] 表格格式正确
- [ ] 无未填充的可选区块
- [ ] 行动入口链接有效
