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

### 0.1 Hero 区

| 字段 (HTML class/id) | 变量 | 类型 | 说明（西语 label 锁定） |
|------|------|------|------|
| `.eyebrow` | `{{hero_eyebrow}}` | string | "EXPERTO · CAVI" |
| `.hero-title` | `{{hero_title}}` | string | `<brand> <series> <year>` 拼装 |
| `.hero-subtitle` | `{{hero_subtitle}}` | string | 一句疑问 ≤ 60 字（如 "¿Vale la pena?"）|
| `.hero-desc` | `{{hero_desc}}` | text | 1 段话，50-80 字 |
| `.hero-cta` | `{{hero_cta_label}}` + `{{finance_apply_url}}` | string | "▶ Ver precio y financiamiento" 按钮 |
| `.hero-link` | `{{hero_link_label}}` | string | "Comparar con VW Virtus / Kia K3 / Mazda3 ›" |
| `.score-badge .num` | `{{cavi_score}}` | number | "4.6" |
| `.score-badge .lbl` | `{{cavi_score_label}}` | string | "CAVI" |

### 0.2 Spec strip（首屏 3 核心数据）

| 字段 | 变量 | 类型 | 说明 |
|------|------|------|------|
| `.spec-card .v` | `{{spec_strip[].value}}` | string | 主数值（带单位 HTML 渲染时插 `<span>`） |
| `.spec-card .l` | `{{spec_strip[].label}}` | string | 副说明（HTML 含 `<b>` 高亮关键词） |
| 数量 | — | — | **固定 3 张** spec-card |

### 0.3 Trim selector（6 版本）

| 字段 (data-attr / class) | 变量 | 类型 | 说明 |
|------|------|------|------|
| `.trim-card.active` | `{{trim_selector_data[].is_recommended}}` | bool | 默认 ADVANCE CVT active |
| `.badge-rec` | `{{trim_selector_data[].badge}}` | string | "RECOMENDADO"（仅推荐版本显示） |
| `data-version` | `{{trim_selector_data[].version_slug}}` | string | 如 `advance-cvt` |
| `data-name` | `{{trim_selector_data[].version_name}}` | string | 如 "ADVANCE CVT" |
| `data-msrp` | `{{trim_selector_data[].msrp}}` | number | 经销价（MXN） |
| `data-official` | `{{trim_selector_data[].official_price}}` | number | 官方价（MXN） |
| `data-monthly` | `{{trim_selector_data[].monthly_payment}}` | number | 估算月供（MXN） |
| `.trim-card .price` | `{{trim_selector_data[].msrp_display}}` | string | 渲染时 `$309,990` 格式 |
| 数量 | — | — | **固定 6 张** trim-card（必须覆盖全系 6 版本） |

### 0.4 Compare bar（4 cells 对比条）

| 字段 | 变量 | 类型 | 说明（西语 label） |
|------|------|------|------|
| `.compare-head .tag` | `{{compare_bar_head_tag}}` | string | "VS COMPETENCIA · ¿CÓMO ESTÁ POSICIONADO?" |
| `.compare-head .more` | `{{compare_bar_head_more}}` | string | "Ver análisis completo ›" |
| `.compare-cell .l` | `{{compare_bar.cells[].label}}` | string | "Precio de entrada" / "Vs. promedio del segmento" / "Ventas julio" / "Recomendación" |
| `.compare-cell .v` | `{{compare_bar.cells[].value}}` | string | 主数值（HTML 含 `<span class="acc">` 强调） |
| 数量 | — | — | **固定 4 cells** |

### 变量说明（汇总）

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{hero_eyebrow}}` | string | "EXPERTO · CAVI"（字面锁定） |
| `{{hero_title}}` | string | `<brand> <series> <year>` 拼装 |
| `{{hero_subtitle}}` | string | 一句疑问 ≤ 60 字 |
| `{{hero_desc}}` | text | 1 段话 50-80 字 |
| `{{hero_cta_label}}` | string | 跳转 finance 段按钮文字 |
| `{{hero_link_label}}` | string | 跳转 compare 段链接文字 |
| `{{cavi_score_label}}` | string | "CAVI"（score-badge 标签） |
| `{{one_liner}}` | string | 一句话定位（兼容旧模板，= hero_subtitle） |
| `{{brand}}` | string | 品牌名 |
| `{{series}}` | string | 车系名 |
| `{{year}}` | number | 年款 |
| `{{energy_type}}` | string | 能源类型 |
| `{{body_type}}` | string | 车身形式 |
| `{{monthly_sales}}` | string | 月销量 |
| `{{target_users}}` | list | 适合人群列表 |
| `{{spec_strip}}` | list<3> | 3 核心数据（label/value） |
| `{{trim_selector_data}}` | list<6> | 6 版本完整数据（见 0.3 字段表） |
| `{{compare_bar}}` | object | 对比条（head + cells<4>） |
| `{{compare_bar_data}}` | block | 旧版别名，向下兼容 |
| `{{finance_apply_url}}` | string | finance apply 直链（hero CTA + 段 02 cta-bar 复用） |

---

## 段 02：价格与金融方案（PRECIO Y FINANCIAMIENTO）

> Frontmatter `block-eyebrow`: `02 · PRECIO Y FINANCIAMIENTO`（**字面锁定**）
> Frontmatter `block-title`: `Precio y planes de financiamiento`
> Frontmatter `block-sub`: `Precio distribuidor + bancos + financiación de marca + bono de intercambio`

### 02.1 主价卡（distribuidor dark 高亮）

| HTML id/class | 变量 | 类型 | 西语 label 锁定 |
|------|------|------|------|
| `#price-lbl` | `{{price_lbl}}` | string | "Precio sugerido · {version_name}" |
| `#price-num` | `{{price_num}}` | string | "$374,990"（含 `<span class="cur">$</span>`） |
| `#price-sub` | `{{price_sub}}` | string | "MXN · precio de lista Nissan México" |
| `#price-base` | `{{price_base}}` | string | "Precio base" → "$374,990" |
| `Bono intercambio` | `{{bonus_trade_in}}` | number | "Bono intercambio" → "−$15,000" |
| `#price-effective` | `{{price_effective}}` | string | "Precio efectivo" → "$359,990" |
| `data-version` | `{{active_version_slug}}` | string | 当前选中版本（联动 trim-card） |

### 02.2 4 张金融卡（price-card）

| 卡类型 | `.lbl` 顶部 | `.price` 主数字 | `.sub` 副标题 | 3 行 row 明细 |
|---|---|---|---|---|
| **main**（dark 高亮）| "Precio sugerido · {version}" | `{{price_num}}` | "MXN · precio de lista" | base / bono / effective |
| **bank** | `{{finance_cards.bank.lbl}}` (如 "Banco · BBVA 36 meses") | `{{finance_cards.bank.monthly}}` ($X/mes) | "Enganche 30% · Tasa 12.9%" | enganche / monto_financiar / interes_total |
| **factory** | `{{finance_cards.factory.lbl}}` ("Financiamiento Nissan") | `{{finance_cards.factory.monthly}}` ($X/mes) | "48 meses · 0% intereses primer año" | enganche / plan_credissan / CAT |
| **trade_in** | `{{finance_cards.trade_in.lbl}}` ("Bono intercambio") | `{{finance_cards.trade_in.amount}}` ($X) | "Si entregas tu auto actual" | vigencia / aplica_a / bono_extra |

### 02.3 4 张金融卡的完整字段定义

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{finance_cards.bank.lbl}}` | string | "Banco · BBVA 36 meses" |
| `{{finance_cards.bank.monthly}}` | number | 月供 "$6,244" |
| `{{finance_cards.bank.sub}}` | string | "Enganche 30% · Tasa 12.9%" |
| `{{finance_cards.bank.enganche}}` | string | "$112,497" |
| `{{finance_cards.bank.monto_financiar}}` | string | "$262,493" |
| `{{finance_cards.bank.interes_total}}` | string | "$39,768" |
| `{{finance_cards.factory.lbl}}` | string | "Financiamiento Nissan" |
| `{{finance_cards.factory.monthly}}` | number | 月供 "$5,899" |
| `{{finance_cards.factory.sub}}` | string | "48 meses · 0% intereses primer año" |
| `{{finance_cards.factory.enganche}}` | string | "$93,748" |
| `{{finance_cards.factory.plan_credissan}}` | string | "48 cuotas" |
| `{{finance_cards.factory.cat}}` | string | "14.8% sin IVA" |
| `{{finance_cards.factory.tag}}` | string | "0% primer año" |
| `{{finance_cards.trade_in.lbl}}` | string | "Bono intercambio" |
| `{{finance_cards.trade_in.amount}}` | number | "$15,000" |
| `{{finance_cards.trade_in.sub}}` | string | "Si entregas tu auto actual" |
| `{{finance_cards.trade_in.vigencia}}` | date | "31 oct 2026" |
| `{{finance_cards.trade_in.aplica_a}}` | string | "ADVANCE+" |
| `{{finance_cards.trade_in.bono_extra}}` | string | "−$8,000" |

### 02.4 cta-bar（章节内 CTA）

| 字段 | 变量 | 说明 |
|------|------|------|
| `.left .h` | `{{cta_h}}` | "¿Quieres calcular tu cuota según tu enganche?" |
| `.left .s` | `{{cta_s}}` | "Compara hasta 3 bancos en 1 minuto · CAVI hace el cálculo por ti" |
| `.btn` | `{{finance_apply_url}}` | "Calcular mi cuota →" 链接 |

---

## 段 03：核心卖点（VENTAJAS CLAVE）

> Frontmatter `block-eyebrow`: `03 · VENTAJAS CLAVE`（**字面锁定**）
> Frontmatter `block-title`: `Lo que realmente destaca del Versa 2026`
> Frontmatter `block-sub`: `4 datos que resumen la propuesta de valor`

### 03.1 4 个 feature-card（核心数据）

| 字段 | 变量 | 类型 | 说明（西语 label 锁定） |
|------|------|------|------|
| `.feature-card:nth-child(1) .v` | `{{key_specs[0].value}}` | string | "18.81 km/L" |
| `.feature-card:nth-child(1) .l` | `{{key_specs[0].label}}` | string | "Consumo combinado SENSE MT · mejor del segmento" |
| `.feature-card:nth-child(2) .v` | `{{key_specs[1].value}}` | string | "482 L" |
| `.feature-card:nth-child(2) .l` | `{{key_specs[1].label}}` | string | "Cajuela · 2 maletas + carreola sin problema" |
| `.feature-card:nth-child(3) .v` | `{{key_specs[2].value}}` | string | "6 airbags" |
| `.feature-card:nth-child(3) .l` | `{{key_specs[2].label}}` | string | "Frontales + laterales + tipo cortina · estándar" |
| `.feature-card:nth-child(4) .v` | `{{key_specs[3].value}}` | string | "3 años" |
| `.feature-card:nth-child(4) .l` | `{{key_specs[3].label}}` | string | "Garantía Nissan · 60,000 km lo que ocurra primero" |

> **固定 4 张** feature-card（不能 3 张也不能 5 张）

### 03.2 quote-grid（CAVI 洞察 + 真实案例）

| 字段 | 变量 | 类型 | 说明 |
|------|------|------|------|
| `.quote-img .tag` | `{{case_tag}}` | string | "CASO · Testimonio real" |
| `.quote-content h3` | `{{insight_quote}}` | string | "Llevo casi un año y todo funciona como debe."（**用户真实引言**） |
| `.quote-content p` | `{{insight_description}}` | string | 50-80 字，含 `<b>` 高亮关键词 |

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{key_specs}}` | list<4> | 4 核心数据（label/value） |
| `{{case_tag}}` | string | "CASO · Testimonio real" |
| `{{insight_quote}}` | string | 洞察原话（用户真实引言） |
| `{{insight_description}}` | string | 50-80 字洞察描述 |
| `{{optional_user_impression}}` | block | 可选：试驾印象 |

---

## 段 04：用户口碑评价（CAVI · RESEÑAS）

> Frontmatter `block-eyebrow`: `04 · CAVI · RESEÑAS`（**字面锁定**）
> Frontmatter `block-title`: `Reseñas de usuarios reales`
> Frontmatter `block-sub`: `3,240 propietarios calificaron · corte julio 2026`

### 04.1 CAVI 综合评分（cavi-top 左）

| 字段 | 变量 | 类型 | 说明 |
|------|------|------|------|
| `.cavi-score .l` | `{{cavi_score_label}}` | string | "CAVI SCORE" |
| `.cavi-score .s` | `{{cavi_score}}` | number | "4.6" |
| `.cavi-score .u` | `{{cavi_score_unit}}` | string | "/ 5.0" |
| `.cavi-score .rec` | `{{cavi_recommend}}` | number | "89% recomienda" |

### 04.2 CAVI 摘要（cavi-top 右）

| 字段 | 变量 | 类型 | 说明 |
|------|------|------|------|
| `.cavi-summary h3` | `{{cavi_verdict}}` | string | "Balance precio/espacio/seguridad inmejorable"（含 `<span class="hl">`） |
| `.cavi-summary p` | `{{cavi_summary_text}}` | text | 4-6 行总结，含 `<b>` 高亮 |
| `.cavi-summary .meta .v` × 3 | `{{monthly_sales}}` / `{{rank_in_segment}}` / `{{review_count}}` | mixed | "7,486" / "#1" / "3,240" |
| `.cavi-summary .meta .l` × 3 | `{{monthly_sales_label}}` / `{{rank_label}}` / `{{review_count_label}}` | string | "Ventas julio" / "Sedán subcompacto MX" / "Reseñas CAVI" |

### 04.3 4 维度 cavi-metrics

| 字段 | 变量 | 类型 | 说明（西语 label 锁定） |
|------|------|------|------|
| `.metric:nth-child(1) .l` | `{{cavi_dimensions.espacio.label}}` | string | "Espacio / cajuela" |
| `.metric:nth-child(1) .v` | `{{cavi_dimensions.espacio.score}}` | number | "4.8" |
| `.metric:nth-child(1) .stars` | `{{cavi_dimensions.espacio.stars}}` | string | "★★★★★" |
| `.metric:nth-child(2) .l` | `{{cavi_dimensions.consumo.label}}` | string | "Consumo real" |
| `.metric:nth-child(2) .v` | `{{cavi_dimensions.consumo.score}}` | number | "4.6" |
| `.metric:nth-child(2) .stars` | `{{cavi_dimensions.consumo.stars}}` | string | "★★★★★" |
| `.metric:nth-child(3) .l` | `{{cavi_dimensions.seguridad.label}}` | string | "Seguridad" |
| `.metric:nth-child(3) .v` | `{{cavi_dimensions.seguridad.score}}` | number | "4.5" |
| `.metric:nth-child(3) .stars` | `{{cavi_dimensions.seguridad.stars}}` | string | "★★★★☆" |
| `.metric:nth-child(4) .l` | `{{cavi_dimensions.ruido.label}}` | string | "Ruido en carretera" |
| `.metric:nth-child(4) .v` | `{{cavi_dimensions.ruido.score}}` | number | "3.9" |
| `.metric:nth-child(4) .stars` | `{{cavi_dimensions.ruido.stars}}` | string | "★★★☆☆" |

> ⚠️ **HTML 实际只展示 4 维度**（espacio/consumo/seguridad/ruido）。v3 模板 v2 写的是 cajuela/consumo/seguridad/ruido，**字段名已修正**：cajuela → espacio（HTML 实际用 "Espacio / cajuela"）

### 04.4 精选评论（2-3 条）

| 字段 | 变量 | 类型 | 说明 |
|------|------|------|------|
| `.review-card .stars` | `{{featured_reviews[].stars}}` | string | "★★★★★" |
| `.review-card .text` | `{{featured_reviews[].text}}` | string | 评论原文（**不修改**） |
| `.review-card .meta` | `{{featured_reviews[].meta}}` | string | "2024 ADVANCE CVT · 15,000 km · Ciudad de México" |

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{cavi_score_label}}` | string | "CAVI SCORE" |
| `{{cavi_score}}` | number | CAVI 综合分（"4.6"） |
| `{{cavi_score_unit}}` | string | "/ 5.0" |
| `{{cavi_recommend}}` | number | 推荐率（%），如 "89" |
| `{{cavi_verdict}}` | string | 一句结论（"Balance precio/espacio/seguridad..."） |
| `{{cavi_summary_text}}` | text | cavi-summary 段落 |
| `{{monthly_sales}}` | number | 月销量（如 "7,486"） |
| `{{monthly_sales_label}}` | string | "Ventas julio" |
| `{{rank_in_segment}}` | string | "Sedán subcompacto MX" / "#1" |
| `{{rank_label}}` | string | "Sedán subcompacto MX" |
| `{{review_count}}` | number | 评论人数（如 "3,240"） |
| `{{review_count_label}}` | string | "Reseñas CAVI" |
| `{{cavi_dimensions}}` | object | **4 维度**评分（espacio/consumo/seguridad/ruido），每个含 label/score/stars |
| `{{featured_reviews}}` | list<2-3> | 精选评论（stars/text/meta） |

---

## 段 05：用车成本（COSTO DE USO）

> Frontmatter `block-eyebrow`: `05 · COSTO DE USO`（**字面锁定**）
> Frontmatter `block-title`: `Costo anual de uso`
> Frontmatter `block-sub`: `Estimado para 20,000 km/año · gasolina, seguro, mantenimiento`

### 05.1 年度用车成本（cost-top）

| 字段 | 变量 | 类型 | 说明（西语 label 锁定） |
|------|------|------|------|
| `.cost-main .l` | `{{annual_cost_label}}` | string | "COSTO ANUAL ESTIMADO" |
| `.cost-main .v` | `{{annual_cost}}` | number | "$28,500 MXN"（带单位） |
| `.cost-main .sub` | `{{monthly_cost_sub}}` | string | "≈ $2,375 MXN / mes · menos que un coffee shop diario" |
| `.cost-stat:nth-child(1) .l` | `{{cost_stat_gasolina_label}}` | string | "Gasolina" |
| `.cost-stat:nth-child(1) .v` | `{{annual_fuel}}` | string | "$14,200/año" |
| `.cost-stat:nth-child(2) .l` | `{{cost_stat_seguro_label}}` | string | "Seguro" |
| `.cost-stat:nth-child(2) .v` | `{{annual_insurance}}` | string | "$9,800/año" |
| `.cost-stat:nth-child(3) .l` | `{{cost_stat_mantenimiento_label}}` | string | "Mantenimiento" |
| `.cost-stat:nth-child(3) .v` | `{{annual_maintenance}}` | string | "$4,500/año" |

### 05.2 4 张 cost-card（5 年累计 + 3 项服务 + 残值）

| 字段 | 变量 | 类型 | 说明 |
|------|------|------|------|
| `.cost-card.hi .l` | `{{cost_5y_label}}` | string | "Costo 5 años" |
| `.cost-card.hi .v` | `{{cost_5y_value}}` | number | "$142,500 MXN" |
| `.cost-card.hi .s` | `{{cost_5y_compare}}` | string | "23% menos que promedio del segmento" |
| `.cost-card:nth-child(2) .l` | `{{servicio_menor_label}}` | string | "Servicio menor" |
| `.cost-card:nth-child(2) .v` | `{{servicio_menor_value}}` | number | "$650 MXN" |
| `.cost-card:nth-child(2) .s` | `{{servicio_menor_detail}}` | string | "Cada 10,000 km · aceite + filtros" |
| `.cost-card:nth-child(3) .l` | `{{servicio_mayor_label}}` | string | "Servicio mayor" |
| `.cost-card:nth-child(3) .v` | `{{servicio_mayor_value}}` | number | "$2,400 MXN" |
| `.cost-card:nth-child(3) .s` | `{{servicio_mayor_detail}}` | string | "Cada 30,000 km · bujías + transmisión" |
| `.cost-card:nth-child(4) .l` | `{{revalorizacion_3a_label}}` | string | "Revalorización 3a" |
| `.cost-card:nth-child(4) .v` | `{{revalorizacion_3a_value}}` | number | "62%" |
| `.cost-card:nth-child(4) .s` | `{{revalorizacion_3a_compare}}` | string | "Mejor que Virtus y K3" |

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{annual_km}}` | number | 年行驶里程（20,000） |
| `{{annual_cost}}` | number | 年度总成本（"$28,500"） |
| `{{annual_cost_label}}` | string | "COSTO ANUAL ESTIMADO" |
| `{{monthly_cost}}` | number | 月度成本（"$2,375"） |
| `{{monthly_cost_sub}}` | string | 副标题（"≈ $2,375 MXN / mes..."） |
| `{{annual_fuel}}` | string | "Gasolina" 行（"$14,200/año"） |
| `{{annual_insurance}}` | string | "Seguro" 行（"$9,800/año"） |
| `{{annual_maintenance}}` | string | "Mantenimiento" 行（"$4,500/año"） |
| `{{cost_5y_value}}` | number | 5 年累计（"$142,500"） |
| `{{servicio_menor_value}}` | number | "Servicio menor"（"$650"） |
| `{{servicio_mayor_value}}` | number | "Servicio mayor"（"$2,400"） |
| `{{revalorizacion_3a_value}}` | number | 3 年残值率（"62"） |
| `{{cost_saving_tips}}` | list | 2-3 条省钱建议（HTML 段 05 未展示，模板保留兼容） |
| `{{fuel_pct}}` 等 | number | 占比（HTML 未展示，兼容） |

---

## 段 06：购车保障与服务覆盖（GARANTÍA Y SERVICIO）

> Frontmatter `block-eyebrow`: `06 · GARANTÍA Y SERVICIO`（**字面锁定**）
> Frontmatter `block-title`: `Cobertura post-compra`
> Frontmatter `block-sub`: `Nissan México · red de distribuidores a nivel nacional`

### 06.1 整车质保（protect-main）

| 字段 | 变量 | 类型 | 说明（西语 label 锁定） |
|------|------|------|------|
| `.protect-main .n` | `{{warranty_years}}` | number | "3" |
| `.protect-main .u` | `{{warranty_mileage_label}}` | string | "AÑOS / 100,000 KM"（年+里程拼装） |
| `.protect-main .s` | `{{warranty_summary}}` | string | "Garantía total Nissan México" |

### 06.2 4 项支持服务（protect-list）

| 字段 | 变量 | 类型 | 说明 |
|------|------|------|------|
| `.protect-item:nth-child(1) .h` | `{{support_features[0].title}}` | string | "Red Nissan en todo México" |
| `.protect-item:nth-child(1) .s` | `{{support_features[0].desc}}` | string | "Más de 280 distribuidores · refacciones originales garantizadas" |
| `.protect-item:nth-child(2) .h` | `{{support_features[1].title}}` | string | "Asistencia vial 24/7" |
| `.protect-item:nth-child(2) .s` | `{{support_features[1].desc}}` | string | "Grúa, gasolina, cerrajero · 3 años incluidos" |
| `.protect-item:nth-child(3) .h` | `{{support_features[2].title}}` | string | "Plan de mantenimiento prepagado" |
| `.protect-item:nth-child(3) .s` | `{{support_features[2].desc}}` | string | "3 servicios incluidos desde $4,990 MXN" |
| `.protect-item:nth-child(4) .h` | `{{support_features[3].title}}` | string | "Nissan Connect" |
| `.protect-item:nth-child(4) .s` | `{{support_features[3].desc}}` | string | "App para localizar, abrir y diagnosticar tu auto" |

> **固定 4 项** protect-item

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{warranty_years}}` | number | 质保年限（"3"） |
| `{{warranty_mileage_label}}` | string | "AÑOS / 100,000 KM"（含拼装逻辑） |
| `{{warranty_mileage}}` | number | 质保里程数字（"100000"，用于 mile 拼装） |
| `{{warranty_summary}}` | string | 质保说明（"Garantía total Nissan México"） |
| `{{warranty_details}}` | block | 质保细节（兼容字段） |
| `{{maintenance_count}}` | number | 保养次数（如 3，兼容） |
| `{{maintenance_mileage}}` | number | 保养里程（兼容） |
| `{{service_network_count}}` | number | 网点数（"280"） |
| `{{support_features}}` | list<4> | 4 项支持（title/desc） |

---

## 段 07：主要竞品怎么选（COMPETENCIA）

> Frontmatter `block-eyebrow`: `07 · COMPETENCIA`（**字面锁定**）
> Frontmatter `block-title`: `Si te interesa el Versa, también estás viendo`
> Frontmatter `block-sub`: `4 sedanes subcompactos · mismo rango de precio`

> ⚠️ **HTML 实际渲染 4 张** compete-card（不是 3 张）：
> - 1 张 self（Versa 本身，tag="TU ELECCIÓN"）
> - 2 张 ALTERNATIVA（同级竞品）
> - 1 张 PREMIUM（越级竞品）
> v3 模板之前 v2 写的是"3 张"，已修正

### 07.1 4 张 compete-card

| 字段 | 变量 | 类型 | 说明 |
|------|------|------|------|
| `.compete-card.self .tag` | `{{competitors[0].tag}}` | enum | `TU ELECCIÓN`（**首张固定为 self**） |
| `.compete-card .name` | `{{competitors[].name}}` | string | "Nissan Versa" / "VW Virtus" / "Kia K3 Sedán" / "Mazda 2 Sedán" |
| `.compete-card .price` | `{{competitors[].price}}` | string | "Desde **$309,990** MXN" |
| `.compete-card .pros` | `{{competitors[].pro}}` | string | "+Mejor precio · cajuela 482L · 6 airbags std" |
| `.compete-card .cons` | `{{competitors[].con}}` | string | "−Ruido carretera · garantía 3 años (estándar)" |
| `.compete-card .img src` | `{{competitors[].image_url}}` | string | CDN 图（CDN URL） |

### 07.2 tag 取值

| tag | 西语 | 含义 |
|------|------|------|
| `TU ELECCIÓN` | "你的选择" | self 卡片（**第 1 张固定**） |
| `ALTERNATIVA` | "同级替代" | 同价位同级竞品 |
| `PREMIUM` | "越级" | 越级对比 |

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{competitors}}` | list<4> | **4 张**竞品卡（首张 self，后 3 张 alternative/premium），每张含 name/price/tag/pro/con/image_url |
| `{{tag}}` | enum | `TU ELECCIÓN`（self）/ `ALTERNATIVA`（同级）/ `PREMIUM`（越级） |

---

## 段 08：想进一步了解？（SIGUIENTE PASO）

> Frontmatter `block-eyebrow`: `08 · SIGUIENTE PASO`（**字面锁定**）
> Frontmatter `block-title`: `¿Listo para decidir? Elige cómo continuar`
> Frontmatter `block-sub`: `CAVI te acompaña hasta que tengas el auto en tu cochera`

### 08.1 3 张 next-card

| 字段 | 变量 | 类型 | 说明（西语 label 锁定） |
|------|------|------|------|
| `.next-card:nth-child(1) .h` | `{{next_cards[0].label}}` | string | "Hablar con un asesor" |
| `.next-card:nth-child(1) .s` | `{{next_cards[0].sub}}` | string | "1 a 1 por WhatsApp o teléfono<br>Sin compromiso · responde &lt; 5 min" |
| `.next-card:nth-child(1) .btn` | `{{next_cards[0].url}}` | string | WhatsApp link + "Iniciar chat →" |
| `.next-card:nth-child(2) .h` | `{{next_cards[1].label}}` | string | "Agendar prueba de manejo" |
| `.next-card:nth-child(2) .s` | `{{next_cards[1].sub}}` | string | "280+ distribuidores Nissan en México<br>Recoge y entrega a domicilio sin costo" |
| `.next-card:nth-child(2) .btn` | `{{next_cards[1].url}}` | string | autocava visit URL + "Reservar horario →" |
| `.next-card:nth-child(3) .h` | `{{next_cards[2].label}}` | string | "Descargar reporte completo" |
| `.next-card:nth-child(3) .s` | `{{next_cards[2].sub}}` | string | "PDF 32 páginas · comparativa técnica detallada<br>Envío a tu email en 30 segundos" |
| `.next-card:nth-child(3) .btn` | `{{next_cards[2].url}}` | string | autocava cavi URL + "Recibir PDF →" |

### 08.2 type 取值

| type | 用途 | 默认 url |
|------|------|------|
| `whatsapp` | 第 1 张：聊天 | `https://wa.me/525527419019` |
| `test_drive` | 第 2 张：预约试驾 | `https://www.autocava.com.mx/visit?seriesId=356` |
| `cavi_ai` | 第 3 张：PDF 下载 | `https://www.autocava.com.mx/cavi` |

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{next_cards}}` | list<3> | 3 张 next-card（type/label/sub/url） |
| 预期 3 个 type | enum | `whatsapp` / `test_drive` / `cavi_ai` |

---
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
