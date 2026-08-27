# CAVI 购车报告示例（v3.0）

基于 Nissan Versa 2026 生成的完整示例，演示如何按 v3.0 模板填充数据（8 段结构）。

> v3.0 关键变化：9 Tab → 8 段（首屏含参数 + 02..08 七段内容），章节命名对齐西语 `block-eyebrow`。

---

## 示例：生成 Nissan Versa 2026 购车报告

### 输入

```yaml
series_id: 356
force_update: false
```

> 当前仅支持 Versa 2026 一个标杆车系。

### 输出文件

```
reports/es/nissan-versa-2026.md
```

---

## 完整输出示例

```markdown
---
title: Nissan Versa 2026 购车指南
series_id: 356
model: nissan-versa
year: 2026
market: MX
lang: es
energy_type: 燃油
body_type: Sedán
device: pc
generated_at: 2026-08-27
source: CAVI (AutoCava AI)
cavi_score: 4.6
cavi_recommend: 89
cavi_verdict: Balance precio/espacio/seguridad inmejorable
spec_strip:
  - { label: "Mejor consumo", value: "18.81", unit: "km/L",  sub: "SENSE MT",   icon: "fuel" }
  - { label: "Cajuela",       value: "482",   unit: "L",     sub: "2 maletas",  icon: "trunk" }
  - { label: "Ventas julio",  value: "7,486", unit: "unid.", sub: "sedán #1 MX", icon: "trending" }
key_specs:
  - { label: "马 力",     value: "118 HP" }
  - { label: "油 耗",     value: "5.3 L/100km" }
  - { label: "后备厢",    value: "480 L" }
  - { label: "中控屏",    value: "7 寸" }
annual_cost: 17300
finance_cards:
  main:     { version_name: "ADVANCE CVT", price: 374990, currency: "MXN", source: "distribuidor" }
  bank:     { monthly: 6244, term: 36, rate: "12.9%", loan_ratio: "70%" }
  factory:  { monthly: 5899, term: 48, rate: "0%",  tag: "0% primer año" }
  trade_in: { amount: 15000, condition: "Si entregas tu auto actual" }
competitors:
  - { tag: "SELF",        name: "Nissan Versa",  price: 309990, pros: "Mejor precio · cajuela 482L · 6 airbags", cons: "Ruido carretera" }
  - { tag: "ALTERNATIVA", name: "VW Virtus",     price: 322490, pros: "Calidad alemana · 126 HP",                cons: "Consumo mayor" }
  - { tag: "ALTERNATIVA", name: "Kia K3 Sedán",  price: 304900, pros: "Garantía 5 años",                          cons: "Resale inferior" }
  - { tag: "PREMIUM",     name: "Mazda 2 Sedán", price: 301900, pros: "Skyactiv KODO",                            cons: "Cajuela 440L" }
next_cards:
  - { type: "whatsapp",   label: "Hablar con un asesor",   sub: "WhatsApp · < 5 min" }
  - { type: "test_drive", label: "Agendar prueba de manejo", sub: "280+ distribuidores" }
  - { type: "cavi_ai",    label: "Preguntar a Cavi (AI)",    sub: "Asistente 24/7" }
---

# Nissan Versa 2026 购车指南

> CAVI 智能解读 · 真实数据来源
> "Balance precio/espacio/seguridad inmejorable" — CAVI

---

## 身份识别

| 维度 | 数据 |
|------|------|
| 品牌 | Nissan |
| 车系 | Versa |
| 年款 | 2026 |
| 能源类型 | 燃油 (Gasolina) |
| 车身形式 | Sedán |
| 座位数 | 5 人 |
| 后备箱空间 | 482 L |
| 发动机 | 1.6L 4-cyl |
| 最大功率 | 118 HP |

### 用户价值定位

!!! success "适合人群"
- 城市通勤为主的家用买家
- 看重月供 + 油费总成本
- 需要 5 座 + 大后备厢的家庭
- 对二手保值率敏感的实用派

---

## Spec strip

| 核心数据 | 数值 |
|----------|------|
| **18.81** km/L | SENSE MT · Mejor consumo |
| **482** L | 2 maletas grandes + equipaje de mano |
| **7,486** unidades | Ventas julio · sedán #1 en México |

---

## Versiones

| Versión | MSRP | Recomendado |
|---------|------|-------------|
| SENSE BASE MT | $309,990 | - |
| SENSE MT | $316,990 | - |
| SENSE CVT | $341,990 | - |
| ADVANCE MT | $363,990 | - |
| **ADVANCE CVT** | **$374,990** | ⭐ RECOMENDADO |
| EXCLUSIVE CVT | $406,990 | - |

---

## 02 · PRECIO Y FINANCIAMIENTO

> Precio y planes
> Distribuidor + bancos + financiación de marca + bono de intercambio

### 主价卡（distribuidor）

> **Precio sugerido · ADVANCE CVT** · $374,990 MXN

| 项目 | 金额 |
|------|------|
| Precio base | $374,990 |
| Bono intercambio | −$15,000 |
| Precio efectivo | $359,990 |

### 银行方案（BBVA 36 个月）

> **$6,244 /mes** · Enganche 30% · Tasa 12.9% APR

| 项目 | 数值 |
|------|------|
| Enganche | $112,497 |
| Monto a financiar | $262,493 |
| Interés total | $39,768 |

### 厂家金融（Nissan Credissan 48 个月）

> **$5,899 /mes** · 0% intereses primer año

| 项目 | 数值 |
|------|------|
| Enganche | $93,748 |
| Plan | Credissan 48 cuotas |
| CAT | 14.8% sin IVA |

### 交换补贴

> **−$15,000** · Si entregas tu auto actual

---

## 03 · CORE SELLING POINTS

> Lo que destaca
> 三段洞察：动力够用 + 油耗省 + 安全配置厚道

### 核心数据

- **118 HP** · 城市通勤动力足
- **5.3 L/100km** · 油耗低，月油费约 $2,000 MXN
- **480 L** · 后备厢可装 2 大行李箱 + 手提
- **6 airbags** · 全系标配

---

## 04 · CAVI · RESEÑAS

> Reseñas de usuarios
> 3,240 propietarios · corte julio 2026

### CAVI 综合评分

> **CAVI 4.6 / 5.0** · 89% recomienda
>
> "Balance precio/espacio/seguridad inmejorable" — CAVI
>
> Sobresale: **cajuela 4.8**, **consumo 4.6**, **seguridad 4.5**. Áreas: ruido (3.9).

### 元信息

| 指标 | 数值 |
|------|------|
| 评论人数 | 3,240 propietarios |
| 月销量 | 7,486 ventas |
| 排名 | #1 sedán en México |

### 4 维度细分

| 维度 | 评分 | 星级 |
|------|------|------|
| Cajuela | 4.8 | ★★★★★ |
| Consumo | 4.6 | ★★★★★ |
| Seguridad | 4.5 | ★★★★☆ |
| Ruido | 3.9 | ★★★☆☆ (短板) |

### 精选评论

> "Llevo casi un año y consumo 18 km/L. Mantenimiento muy económico, servicio menor $650 MXN."
> — 2024 ADVANCE CVT · 15,000 km · CDMX

> "Diseño atractivo, CVT muy suave en ciudad. Solo el ruido de carretera es notable, pero aceptable por el precio."
> — 2025 SENSE CVT · 8,200 km · Guadalajara

---

## 05 · COST

> Costo anual
> 按年行驶 15,000 km 计算

### 年度用车成本

> **$17,300 /年** （约 $1,442 /月）

### 5 项明细

| 项目 | 金额 | 占比 |
|------|------|------|
| Combustible | $8,000 | 46% |
| Seguro | $4,000 | 23% |
| Mantenimiento | $2,500 | 14% |
| Depreciación | $2,000 | 12% |
| Tenencia | $800 | 5% |

### 省钱建议

- Mantén presión de neumáticos en 32 PSI — ahorra 3% combustible
- Compara seguros en línea cada año — diferencia típica 15%

---

## 06 · PROTECTION

> Cobertura post-compra
> 全国 1,500+ 网点

### 整车质保

> **3 年 / 100,000 km**

### 保养包含

> **12 次 / 24,000 km** (Nissan México plan)

### 支持项目

- [全车保修]
- [电池保修]
- [软件OTA]
- [24h 道路救援]

---

## 07 · COMPETITORS

> ¿Qué comparar?
> 4 sedanes subcompactos · mismo rango de precio

### 4 张并排卡

#### TU ELECCIÓN · Nissan Versa
> Desde **$309,990** MXN
- **+** Mejor precio · cajuela 482L · 6 airbags std
- **−** Ruido carretera · garantía 3 años

#### ALTERNATIVA · VW Virtus
> Desde **$322,490** MXN
- **+** Calidad de manejo alemana · 126 HP
- **−** Consumo ligeramente mayor · ensamble India

#### ALTERNATIVA · Kia K3 Sedán
> Desde **$304,900** MXN
- **+** Garantía 5 años · 8 versiones · ensamble MX
- **−** Resale inferior al Versa

#### PREMIUM · Mazda 2 Sedán
> Desde **$301,900** MXN
- **+** Manejo Skyactiv · ensamble MX · diseño KODO
- **−** Cajuela 440L · garantía 3 años

---

## 08 · SIGUIENTE PASO

> ¿Listo para decidir?
> Elige cómo continuar

### 3 张 next-card

#### Hablar con un asesor
> WhatsApp directo · < 5 min respuesta
> [Abrir chat →](https://wa.me/525527419019)

#### Agendar prueba de manejo
> 280+ distribuidores en México
> [Reservar horario →](https://www.autocava.com.mx/test-drive)

#### Preguntar a Cavi (AI)
> Asistente inteligente 24/7
> [Abrir Cavi →](https://www.autocava.com.mx/cavi)

---

*📊 数据来源：[AutoCava CAVI](https://www.autocava.com.mx/cavi) · Versa Series ID: 356 · 生成时间: 2026-08-27*
```

---

## 关键说明

### AI 字段 vs 系统字段

| 类型 | 在示例中的体现 |
|------|--------------|
| **系统字段**（原样填入）| 价格、销量、CAVI 分、版本 msrp |
| **AI 润色**（系统值 + 自然语言）| "Balance precio/espacio/seguridad inmejorable" 这类洞察句 |
| **AI 自由生成** | "省钱建议"、"城市通勤为主的家用买家"（不涉数字）|
| **AI 不可触碰** | 价格 / 评分 / 销量 / 评论原文 — AI 不许改 |

### 西语 block-eyebrow 字面（必须字面一致）

- `02 · PRECIO Y FINANCIAMIENTO`
- `03 · CORE SELLING POINTS`
- `04 · CAVI · RESEÑAS`
- `05 · COST`
- `06 · PROTECTION`
- `07 · COMPETITORS`
- `08 · SIGUIENTE PASO`

---

*参考文档：*
- *字段表：`docs/MD-数据-需求清单.md`*
- *AI 骨架：`assets/standard-template-v3.md`*
- *SKILL：`SKILL.md`（v3.1）*
- *质量保证：`docs/MD-生成-质量保证.md`*
