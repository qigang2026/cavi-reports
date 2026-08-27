# CAVI 报告式页面重构方案

**文档版本**：v2.0
**创建日期**：2026-08-27
**修订日期**：2026-08-27（升级到 v2.0，章节数与最终模板对齐）
**目的**：把"车系详情页"改造成"购车决策报告"
**依据**：`购车手册/20260827-142004.jpeg`（Autocava CAVI 报告范式） + `reports/autocava-h5-mx-v1.html` & `reports/templates/autocava-pc-v1.html`（最终落地的 H5/PC 模板）

---

## 一、核心结论

> **报告 ≠ 详情页。**
> 详情页是"陈列信息让用户自己读"；报告是"先给结论，再用数据支撑，每个章节有判定"。
> 用户来到 CAVI 报告，30 秒内要回答 3 个问题：
> 1. **值不值得买？**（CAVI 指数 + 一句话）
> 2. **要花多少钱、怎么买？**（现价 + 金融方案对比）
> 3. **跟同价位对手比怎么样？**（3 个直接竞品并排）

模板实际呈现：**8 段内容（首屏 + 02..08）**——首屏把"参数"融入 Spec strip，章节命名全部用西语 block-eyebrow 标签，v1.0 的 9 Tab 设计已被实际落地版本替代。

---

## 二、报告 vs 详情页 · 5 个本质差异

| # | 维度 | 详情页（v1.0 旧版） | 报告（v2.0 落地版） |
|---|------|-------------------|-------------------|
| 1 | **首屏** | 概览 Tab + 参数 Tab 分两段 | Hero + Spec strip（3 核心数据）+ Trim selector + Compare bar |
| 2 | **价格章** | 列表 + 月供 | **主价卡（深色 distribuidor）** + **3 补充卡**（BBVA / Nissan financing / 交换补贴）+ **cta-bar** "¿Calcular tu cuota?" |
| 3 | **卖点章** | Tag 列表 | **3-4 个核心数据指标**（大数字+短标签）+ 1 段洞察（配大图） |
| 4 | **口碑章** | 3-5 条评论卡 | **CAVI 4.6** + **89% 推荐** + **4 维度细分**（cajuela 4.8 / consumo 4.6 / seguridad 4.5 / ruido 3.9）+ 精选评论 |
| 5 | **竞品章** | 卡片堆叠 + 对比表 | **3 张并排卡**（VW Virtus / Kia K3 / Mazda 2）+ **ALTERNATIVA / PREMIUM 标签** |
| 6 | **CTA** | 底部固定 WhatsApp+电话 | **3 张 next-card**（WhatsApp 顾问 / 预约试驾 / Cavi AI）+ **Hero CTA** "Consultar planes" + 底部 Fixed bottom（窄 WhatsApp + 橙色 Llamar） |

**根因**：详情页是"导购工具"，报告是"决策辅助"。用户场景完全不同——

- 详情页用户：还在逛、比价、看车
- 报告用户：已经锁定车型，**正在下决定要不要出手**

报告的关键体验：**每章都有"判定"**，没有一段是纯陈述。

---

## 三、CAVI 8 段骨架对照表（v2.0）

| # | block-eyebrow（西语） | block-title | 现有落地版（h5-mx / pc-v1）| 改造状态 |
|---|---------------------|------------|------------------------|---------|
| 0 | (Hero 段) | 车名 + Spec strip + Trim selector + Compare bar | Hero 区 | ✅ 已落地 |
| 02 | `02 · PRECIO Y FINANCIAMIENTO` | Precio y planes | 主价卡（深色）+ 3 补充卡 + cta-bar | ✅ 已落地 |
| 03 | `03 · CORE SELLING POINTS` | Lo que destaca | 3-4 核心数据 + 洞察 | ✅ 已落地 |
| 04 | `04 · CAVI · RESEÑAS` | Reseñas de usuarios | CAVI 4.6 + 4 维度 + 评论 | ✅ 已落地 |
| 05 | `05 · COST` | Costo anual | 年度总额 + 5 项明细 + 省钱建议 | ✅ 已落地 |
| 06 | `06 · PROTECTION` | Cobertura post-compra | 质保 + 保养 + 服务网络 | ✅ 已落地 |
| 07 | `07 · COMPETITORS` | ¿Qué comparar? | 3 张并排 + ALTERNATIVA/PREMIUM 标签 | ✅ 已落地 |
| 08 | `08 · SIGUIENTE PASO` | ¿Listo para decidir? | 3 张 next-card（WhatsApp/试驾/Cavi AI）| ✅ 已落地 |

> **v1.0 → v2.0 关键变化**：
> - "概览 Tab + 参数 Tab" 合并为 Hero + Spec strip（首屏含 3 核心数据）
> - CAVI 评分 4.05 → **4.6**（基于 Versa 2026 实际数据）
> - CAVI 维度 3 个 → **4 个**（cajuela / consumo / seguridad / ruido，含 1 个短板）
> - 价格从 4 列并排 → **主价卡 + 3 补充卡**（每张突出不同信息维度）
> - 竞品从 3 卡 + 全文对比表 → 3 张并排卡 + 标签分级（ALTERNATIVA / PREMIUM）
> - 行动入口从"方案/试驾/官网" → **WhatsApp 顾问 / 预约试驾 / Cavi AI**（与底部 Fixed bottom 互补）

---

## 四、每章的内容维度 + 呈现方式（基于 v2.0 落地版）

### 4.0 首屏（Hero + Spec strip + Trim selector + Compare bar）

**目的**：30 秒给出"值/不值"的判定 + 让用户锁定自己关心的版本

**内容维度**：
1. **车名 + 副标题**（如 `Nissan Versa 2026 · Sedán`）
2. **Spec strip**：3 个核心数据条（如 `118 HP · 5.3L/100km · 7,486 ventas/mes`）
3. **Trim selector**：所有版本横向选择（点击切换主价卡数据）
4. **Compare bar**：与已收藏/同价位车型的快速对比入口

**Hero CTA**：黄底"Consultar planes"按钮（直链 `autocava.com.mx/finance/apply?sourceType=1&seriesId={id}&brandId={id}`）

**呈现方式**：
```
┌─────────────────────────────────────┐
│  Nissan Versa 2026                  │
│  Sedán · 118 HP · 5.3L/100km        │
│  ┌──────┬──────┬──────┐             │
│  │ 118HP│ 5.3L │7,486 │  (Spec)     │
│  └──────┴──────┴──────┘             │
│  [Trim 1] [Trim 2] [Trim 3] [Trim 4]│  (Trim selector)
│  [Comparar con...]                  │  (Compare bar)
│                                     │
│  [ Consultar planes ]  (Hero CTA)   │
└─────────────────────────────────────┘
```

---

### 4.2 价格与金融方案

**目的**：回答"要花多少钱、怎么买"

**内容维度**（基于 Versa 2026 实际）：
1. **主价卡**（深色）：distribuidor 价 + 详细拆解（precio base / bono intercambio / precio efectivo）
2. **BBVA 银行方案**：36 个月 + 月供 + 年利率
3. **Nissan 厂家金融**：48 个月 + 0% 利率（首年贴息）
4. **Bono intercambio**：交换补贴
5. **cta-bar**："¿Calcular tu cuota? · Compara hasta 3 bancos · 1 min · Calcular →"

**呈现方式**：
```
┌─────────────────────────────────────┐
│ 02 · PRECIO Y FINANCIAMIENTO       │
│ Precio y planes                     │
│ Distribuidor + bancos + marca + ... │
├─────────────────────────────────────┤
│ ╔═══════════════════════════════╗   │
│ ║ Precio sugerido · ADVANCE CVT ║   │  (主价卡·深色)
│ ║         $374,990               ║   │
│ ║ MXN · precio de lista Nissan  ║   │
│ ║ ─────────────────────────────  ║   │
│ ║ Precio base      $374,990     ║   │
│ ║ Bono intercambio −$15,000     ║   │
│ ║ Precio efectivo  $359,990     ║   │
│ ╚═══════════════════════════════╝   │
│ ┌──────────┬──────────┬──────────┐  │
│ │BBVA 36m  │Nissan Fin│Bono interc│ │  (3 补充卡)
│ │$6,244/m  │$5,899/m  │−$15,000   │  │
│ │12.9% APR │0% 1er año│Si entregas│  │
│ └──────────┴──────────┴──────────┘  │
│ ┌─────────────────────────────────┐│
│ │ ¿Calcular tu cuota? → Calcular  ││  (cta-bar)
│ │ Compara hasta 3 bancos · 1 min  ││
│ └─────────────────────────────────┘│
└─────────────────────────────────────┘
```

**关键数据点**：
- 4 个数字维度：distribuidor / BBVA 月供 / Nissan 月供 / 交换补贴
- 厂家金融的"0% 利率"或"贴息"标签
- cta-bar 给出"60s 申请"可量化预期

---

### 4.3 核心卖点

**目的**：用 3-4 个关键数据讲清"为什么是它"

**内容维度**：
1. **3-4 个核心数据指标**（大数字+短标签）
2. **1 段洞察描述**（配大图，约 50-80 字）
3. **可选**：1 句"试驾印象"或"用户高频提到"的话

**呈现方式**：
```
┌─────────────────────────────────────┐
│ 03 · CORE SELLING POINTS            │
│ Lo que destaca                      │
├─────────────────────────────────────┤
│ ┌──────┬──────┬──────┬──────┐       │
│ │ 118HP│5.3L  │ 480L │ 7"   │       │
│ │ 马力  │油耗  │后备厢│中控屏│       │  (核心数据)
│ └──────┴──────┴──────┴──────┘       │
│ ┌─────────────────────────────────┐│
│ │ [大图]                            ││
│ │ 空调也不孤单，甚至出行时少吸点烟  ││  (洞察)
│ │ 一段 50-80 字的洞察描述...        ││
│ └─────────────────────────────────┘│
└─────────────────────────────────────┘
```

**关键数据点**（按车型类型选 3-4 个）：
- **电动车**：续航/动力/电池/快充电压
- **燃油车**：马力/扭矩/油耗/后备厢容积
- **混动车**：综合油耗/纯电续航/系统综合功率/电池容量

---

### 4.4 用户口碑评价（CAVI 指数）

**目的**：用第三方视角验证"用户买完觉得怎么样"

**内容维度**（基于 Versa 2026 实际 CAVI 4.6）：
1. **CAVI 综合评分**：4.6 / 5.0 + **89% 推荐率**
2. **一句洞察**：如 "Balance precio/espacio/seguridad inmejorable"
3. **4 维度细分**（cajuela / consumo / seguridad / ruido）：
   - cajuela 4.8 ★★★★★
   - consumo 4.6 ★★★★★
   - seguridad 4.5 ★★★★☆
   - ruido 3.9 ★★★☆☆（**短板**）
4. **元信息**：Ventas / 排名 / 评论数
5. **精选评论**：2-3 条

**呈现方式**：
```
┌─────────────────────────────────────┐
│ 04 · CAVI · RESEÑAS                 │
│ Reseñas de usuarios                 │
│ 3,240 propietarios · corte jul 2026 │
├─────────────────────────────────────┤
│ ┌────────┐                          │
│ │ CAVI   │ Balance precio/espacio/  │  (大数 + 洞察)
│ │  4.6   │ seguridad inmejorable     │
│ │ /5.0   │ 4.6 = 89% recomienda     │
│ │89% rec │ Sobresale: cajuela 4.8   │
│ └────────┘                          │
│ 7,486 ventas · #1 Sedán MX · 3.2K  │  (元信息)
├─────────────────────────────────────┤
│ Cajuela  ★★★★★  4.8/5               │
│ Consumo  ★★★★★  4.6/5               │  (4 维度细分)
│ Seguridad ★★★★☆ 4.5/5               │
│ Ruido    ★★★☆☆ 3.9/5  (短板)       │
├─────────────────────────────────────┤
│ "Llevo casi un año y consumo 18 km/L"│  (精选评论)
│ 2024 ADVANCE CVT · 15,000 km · CDMX │
└─────────────────────────────────────┘
```

**关键数据点**：
- CAVI 综合分 = 4.6（基于实际车主评论，**与"营销评分"区分**）
- 4 维度：cajuela / consumo / seguridad / ruido（**含 1 个短板**才显真实）
- 维度评分 + 星级可视化 + 评论人数

---

### 4.5 用车成本

**目的**：让用户知道"买完之后每年要花多少"

**内容维度**：
1. **年度总成本**（大数字）
2. **5 项并列明细**：燃料 / 保险 / 维护 / 折旧 / 购置税
3. **每项的金额 + 占比 + 简短说明**
4. **省钱建议**（2-3 条）

**呈现方式**：
```
┌─────────────────────────────────────┐
│ 05 · COST                           │
│ Costo anual                        │
│ 以下数据基于15,000km/年计算          │
├─────────────────────────────────────┤
│ 用车成本                    $7,280/年│  (总额)
├──────┬──────┬──────┬──────┬─────────┤
│ 燃料 │ 保险 │ 维护 │ 折旧 │ 购置税   │  (5 项明细)
│$2,766│$1,749│$1,512│$1,266│ $?     │
└──────┴──────┴──────┴──────┴─────────┘
[省钱建议：xxx · xxx · xxx]
```

**关键数据点**：
- 5 项：Fuel / Insurance / Maintenance / Depreciation / Tax（可按市场调整）
- 每项金额 = 月度 × 12
- 总额 = 5 项之和

---

### 4.6 购车保障与服务覆盖

**目的**：让用户知道"买了之后出问题怎么办"

**内容维度**：
1. **质保里程**（大字，如 7 年 / 120,000 km）
2. **保养含**（如 12 次 / 24,000 km）
3. **支持项目清单**（3-4 项 Tag）
4. **服务网络覆盖**（如 1,500+ 网点）

**呈现方式**：
```
┌─────────────────────────────────────┐
│ 06 · PROTECTION                     │
│ Cobertura post-compra              │
├──────────────┬──────────────┤
│  质保         │  保养含       │  (双列)
│7年/120,000km│12次/24,000km│
├──────────────┴──────────────┤
│ [全车保修] [电池保修]        │  (Tag 列表)
│ [软件OTA] [24h 道路救援]     │
└─────────────────────────────┘
```

**关键数据点**：
- 整车质保年限 + 里程
- 电池质保（电动车专属）
- 保养包次数 + 里程
- 服务网络规模
- 24h 道路救援

---

### 4.7 主要竞品怎么选

**目的**：让用户一眼看到"同价位还有什么选"

**内容维度**（基于 Versa 2026 实际 3 个直接竞品）：
1. **3 个直接竞品**（价格段 ±10%）：VW Virtus / Kia K3 Sedán / Mazda 2 Sedán
2. **每竞品一张卡**：车图 + 名称 + 起售价 + **pros/cons** + 标签
3. **本车不显示在卡片中**（避免自我对比），但用"ALTERNATIVA"或"PREMIUM"分级标签

**呈现方式**：
```
┌─────────────────────────────────────┐
│ 07 · COMPETITORS                    │
│ ¿Qué comparar?                     │
├──────────────┬──────────────┬──────────────┐
│ ALTERNATIVA  │ ALTERNATIVA  │ PREMIUM      │  (3 张并排)
│ [VW Virtus]  │ [Kia K3]     │ [Mazda 2]    │
│ Desde $322K  │ Desde $304K  │ Desde $301K  │
│ + Manejo     │ + Garantía   │ + Skyactiv   │
│   alemán     │   5 años     │   KODO       │
│ − Consumo    │ − Resale     │ − Cajuela    │
│   mayor      │   inferior   │   440L       │
└──────────────┴──────────────┴──────────────┘
```

**关键数据点**：
- 3 个直接对标（价格段 ±10%）
- 每竞品：起售价 + pros/cons + 标签分级
- 标签语义：**ALTERNATIVA**（同级替代） / **PREMIUM**（越级比较）
- **不展示全文对比表**——CAVI 报告强调"3 秒读懂"，对比表信息密度过高

---

### 4.8 想进一步了解？（CTA 行动入口）

**目的**：让用户从"了解"变成"动作"

**内容维度**（基于 Versa 2026 实际 3 张 next-card）：
1. **Hablar con un asesor**（WhatsApp 顾问）：`< 5 min respuesta` 预期
2. **Agendar prueba de manejo**（预约试驾）：`280+ distribuidores` 网络
3. **Preguntar a Cavi (AI)**（Cavi AI 助手）：`24/7 智能问答`

**呈现方式**：
```
┌─────────────────────────────────────┐
│ 08 · SIGUIENTE PASO                 │
│ ¿Listo para decidir?                │
│ Elige cómo continuar                │
├─────────────────────────────────────┤
│ 💬 Hablar con un asesor              │  (next-card 1)
│    WhatsApp directo · < 5 min resp   │
│    Abrir chat →                      │
├─────────────────────────────────────┤
│ 🚗 Agendar prueba de manejo          │  (next-card 2)
│    280+ distribuidores en México     │
│    Reservar horario →                │
├─────────────────────────────────────┤
│ 🤖 Preguntar a Cavi (AI)             │  (next-card 3)
│    Asistente inteligente 24/7        │
│    Abrir Cavi →                      │
└─────────────────────────────────────┘
```

**配套全局 CTA**：
- **Hero CTA**："Consultar planes"（直链 finance apply）
- **Fixed bottom**：窄 WhatsApp 图标 + 橙色 "Llamar"

**关键数据点**：
- 3 个 next-card 与底部 Fixed bottom 互补（next-card 是结构化选项，Fixed bottom 是快捷动作）
- 每个 next-card 有可量化预期（时长 / 网络规模 / 服务时间）

---

## 五、MD 模板字段映射（呼应 PRD v3.0）

参考 PRD v3.0 的章节结构，把 CAVI v2.0 范式映射为 MD frontmatter 字段：

```yaml
---
title: Nissan Versa 2026 购车指南
series_id: 356
model: nissan-versa
year: 2026
market: MX
lang: zh
energy_type: 燃油
body_type: Sedán
generated_at: 2026-08-27

# CAVI v2.0 新增字段
cavi_score: 4.6           # CAVI 综合指数
cavi_recommend: 89        # 推荐率（%）
cavi_verdict: Balance precio/espacio/seguridad inmejorable   # 一句结论
cavi_dimensions:          # 4 维度细分
  cajuela: 4.8
  consumo: 4.6
  seguridad: 4.5
  ruido: 3.9               # 短板维度
spec_strip:               # 首屏 3 核心数据
  - { label: 马力, value: "118 HP" }
  - { label: 油耗, value: "5.3 L/100km" }
  - { label: 月销量, value: "7,486" }
key_specs:                # 卖点章 3-4 个核心数据
  - { label: 马力, value: "118 HP" }
  - { label: 油耗, value: "5.3 L/100km" }
  - { label: 后备厢, value: "480 L" }
  - { label: 中控屏, value: "7 寸" }
annual_cost: 17300         # 年度用车成本
finance_cards:            # 金融方案 4 卡
  main: { price: 374990, currency: "MXN", source: "distribuidor" }
  bank: { monthly: 6244, term: 36, rate: "12.9%", loan_ratio: "70%" }
  factory: { monthly: 5899, term: 48, rate: "0%", tag: "0% primer año" }
  trade_in: 15000
competitors:              # 3 张并排卡
  - { name: "VW Virtus", price: 322490, tag: "ALTERNATIVA", pro: "Manejo alemán", con: "Consumo mayor" }
  - { name: "Kia K3 Sedán", price: 304900, tag: "ALTERNATIVA", pro: "Garantía 5 años", con: "Resale inferior" }
  - { name: "Mazda 2 Sedán", price: 301900, tag: "PREMIUM", pro: "Skyactiv KODO", con: "Cajuela 440L" }
next_cards:               # 3 张 next-card
  - { type: whatsapp, label: "Hablar con un asesor", sub: "< 5 min respuesta" }
  - { type: test_drive, label: "Agendar prueba de manejo", sub: "280+ distribuidores" }
  - { type: cavi_ai, label: "Preguntar a Cavi (AI)", sub: "Asistente 24/7" }
---
```

---

## 六、落地动作清单

### ✅ 已完成（v2.0 落地）

- [x] H5 模板：`reports/autocava-h5-mx-v1.html`（54KB，13 sections，8 段核心内容）
- [x] PC 模板：`reports/templates/autocava-pc-v1.html`（62KB，13 sections，8 段核心内容）
- [x] Hero CTA：顶部黄底"Consultar planes"按钮
- [x] Fixed bottom：窄 WhatsApp 图标（56px）+ 橙色 Llamar
- [x] 西语 block-eyebrow 标签全段对齐
- [x] CAVI 4.6 + 4 维度（cajuela/consumo/seguridad/ruido）
- [x] 价格 4 卡（主价卡深色 + 3 补充卡 + cta-bar）
- [x] 竞品 3 张并排（ALTERNATIVA / PREMIUM 标签）
- [x] 行动入口 3 张 next-card（WhatsApp / 试驾 / Cavi AI）
- [x] PRD v3.0 章节结构表（docs/AUTOCAVA 购车报告需求.md）
- [x] SKILL v3.0 Tab 映射（skills/cavi-guide-gen/SKILL.md）
- [x] standard-template v3.0（skills/cavi-guide-gen/assets/standard-template-v3.md）

### 待办（v2.0 之后）

- [ ] 用 v2.0 模板批量重做 1 个完整车型作为标杆（建议：Versa 2026 西语版）
- [ ] 埋点：CAVI 指数展示、CTA 点击、章节停留时长
- [ ] A/B 测试：报告式 vs 详情式 转化率对比
- [ ] 跨市场适配：模板目前以 MX 市场为主，需确认其他市场（CN/CO/AR）的字段兼容性

---

## 七、参考资源

- **参考图**：`购车手册/20260827-142004.jpeg`
- **H5 模板**：`reports/autocava-h5-mx-v1.html`（最终落地的移动版）
- **PC 模板**：`reports/templates/autocava-pc-v1.html`（最终落地的桌面版）
- **PRD**：`docs/AUTOCAVA 购车报告需求.md`（v3.0）
- **SKILL**：`skills/cavi-guide-gen/SKILL.md`（v3.0）
- **解读版**（参考，非 CAVI 范式）：`reports/es/nissan-versa-2026-interpretacion.html`
