# MD 生成 · 展现样式规范

**文档版本**：v1.2
**创建日期**：2026-08-27
**修订日期**：2026-08-27（v1.2：明确 Versa 标杆 1 个车系 + 4 维度锁定）
**目的**：明确本项目**唯一**的报告展现样式 + 标杆车系的数据规范
**关联**：`docs/MD-数据-需求清单.md`（字段）· `docs/CAVI 报告式页面重构方案.md` v2.0

---

## 一、核心结论

> **本项目有 1 个报告展现样式（西语 PC），跑 1 个标杆车系（Versa 2026）。**

报告的"长什么样"由这 1 个样式决定；渲染到哪由设备载体（PC 或 H5）决定。

**反过来说**：
- ❌ 不存在"中文模板" / "西语模板"——只有 1 个西语模板
- ❌ 不存在"燃油车模板" / "电动车模板"——只有 1 个 Versa 燃油模板
- ❌ 不存在"MX 模板" / "CN 模板"——只有 1 个 MX 模板

**标杆 = Versa 2026（series_id=356）**。未来加新车型时，**先跑通 1 个标杆**，再讨论扩展。

---

## 二、唯一展现样式

| 项 | 值 |
|---|---|
| **样式 ID** | `cavi-report-v1` |
| **来源** | `reports/templates/autocava-pc-v1.html`（PC 版）|
| **章节数** | 8 段（首屏 + 02..08）|
| **block-eyebrow** | 西语字面锁定（见 §三）|
| **适用市场** | MX（其他市场未来扩展）|
| **适用语言** | 西语（es）|
| **适用能源** | 燃油（其他能源类型未来扩展）|

**章节骨架**：

```
段 0  首屏：Hero + Spec strip + Trim selector + Compare bar
段 02 Precio y financiamiento
段 03 Core selling points
段 04 CAVI reviews
段 05 Cost of ownership
段 06 Protection & service
段 07 Competitors
段 08 Next steps
```

---

## 三、西语 block-eyebrow 字面锁定

**前端按字面渲染**——AI 生成 MD 时必须严格匹配，**字面一个字符不能差**：

| 段 | block-eyebrow |
|----|---------------|
| 02 | `02 · PRECIO Y FINANCIAMIENTO` |
| 03 | `03 · CORE SELLING POINTS` |
| 04 | `04 · CAVI · RESEÑAS` |
| 05 | `05 · COST` |
| 06 | `06 · PROTECTION` |
| 07 | `07 · COMPETITORS` |
| 08 | `08 · SIGUIENTE PASO` |

**校验**：`docs/MD-生成-质量保证.md` §4.2 字面校验失败 = 阻塞。

---

## 四、2 种渲染载体（设备版式）

| 载体 | HTML 模板 | 用途 | 数据 |
|------|----------|------|------|
| **PC 桌面** | `reports/templates/autocava-pc-v1.html` | 桌面浏览器 | 与 H5 **同一份 MD** |
| **H5 移动** | （在 reports/ 根）| 手机浏览器 | 与 PC **同一份 MD** |

**关系**：
```
一份 MD 数据（reports/es/nissan-versa-2026.md）
        ↓
   ┌────┴────┐
   ↓         ↓
  PC 渲染   H5 渲染
（不同 CSS/版式，相同内容）
```

**含义**：
- 不需要为 PC 和 H5 各生成一份 MD
- 同一份 MD 喂给两个 HTML 模板，前端按设备加载
- H5 模板与 PC 模板在字段层面**完全兼容**（字段集相同）

---

## 五、Versa 标杆的数据规范

### 5.1 CAVI 4 维度（锁定）

> **未来加新车型时 4 维度需要重新评审，但当前不做**。

| slug | 西语 label | 中文 label | 实际值（Versa 2026）| 星级 | is_weakness |
|------|----------|----------|---------------------|------|-------------|
| `cajuela` | Cajuela | 后备厢 | 4.8 | 5 | false |
| `consumo` | Consumo | 油耗 | 4.6 | 5 | false |
| `seguridad` | Seguridad | 安全 | 4.5 | 4 | false |
| `ruido` | Ruido | 噪音 | 3.9 | 3 | **true** |

**为什么含 1 个短板维度**：报告强调"3 秒读懂"，含 1 个短板（如噪音 3.9）让评分更可信，避免"全是满分"的营销感。

### 5.2 版本（6 个锁定）

| code | name | msrp | official | monthly | is_recommended |
|------|------|------|----------|---------|----------------|
| sense-base-mt | SENSE BASE MT | 309,990 | 374,900 | 5,162 | false |
| sense-mt | SENSE MT | 316,990 | 382,900 | 5,278 | false |
| sense-cvt | SENSE CVT | 341,990 | 406,900 | 5,694 | false |
| advance-mt | ADVANCE MT | 363,990 | 428,900 | 6,061 | false |
| advance-cvt | ADVANCE CVT | 374,990 | 439,900 | 6,244 | **true** |
| exclusive-cvt | EXCLUSIVE CVT | 406,990 | 470,900 | 6,777 | false |

**恰好 1 个推荐**（ADVANCE CVT）。

### 5.3 金融方案（4 卡锁定）

- **主价卡（深色）**：ADVANCE CVT 经销价 374,990 MXN，交换补贴 -15,000，实际价 359,990
- **BBVA 36 个月**：月供 6,244，利率 12.9%，贷款 70%
- **Nissan 厂家 48 个月**：月供 5,899，0% 首年贴息（Credissan）
- **交换补贴**：-15,000，Vigencia 31 oct 2026

### 5.4 3 个直接竞品（锁定）

| 标签 | 车名 | 起售价 | pros | cons |
|------|------|--------|------|------|
| SELF | Nissan Versa | 309,990 | Mejor precio · cajuela 482L · 6 airbags std | Ruido carretera · garantía 3 años |
| ALTERNATIVA | VW Virtus | 322,490 | Calidad de manejo alemana · 126 HP | Consumo ligeramente mayor · ensamble India |
| ALTERNATIVA | Kia K3 Sedán | 304,900 | Garantía 5 años · 8 versiones · ensamble MX | Resale inferior al Versa |
| PREMIUM | Mazda 2 Sedán | 301,900 | Manejo Skyactiv · ensamble MX · diseño KODO | Cajuela 440L · garantía 3 años |

### 5.5 3 个行动入口（锁定）

| type | label | sub | url |
|------|-------|-----|-----|
| whatsapp | Hablar con un asesor | WhatsApp directo · < 5 min respuesta | wa.me/525527419019 |
| test_drive | Agendar prueba de manejo | 280+ distribuidores en México | autocava.com.mx/test-drive |
| cavi_ai | Preguntar a Cavi (AI) | Asistente inteligente 24/7 | autocava.com.mx/cavi |

---

## 六、未来扩展（如需要）

### 6.1 新增车型

- 当前：仅 Versa 2026
- 未来：先跑通 Versa 1 个标杆，**再讨论加新车系**
- 实施：
  1. PM 评审必要性
  2. 评审新车型 4 维度定义
  3. 接入新数据源（如果新车型需要）
  4. 灰度上线

### 6.2 新增市场

- 当前：MX
- 未来：CN / CO / AR
- 实施：
  1. 新增市场级配置（货币、电话、WhatsApp 号、银行产品等）
  2. 决定每个市场的 4 维度标签
  3. 决定每个市场的 eyebrow 字面

### 6.3 新增语言

- 当前：es（西语）
- 未来：zh（中文）/ en（英文）
- 实施：
  1. 新增一套 eyebrow 字面（`02 · 价格与金融方案` 等）
  2. AI 翻译系统原文（**AI 可改字面，不许改数字**）
  3. 字段表新增 `label_zh` / `label_en` 列

### 6.4 新增能源类型 / 车型

- 不需要新增模板
- 同一展现样式下，spec_strip / key_specs / cavi_dimensions 的具体值随车型变化
- 4 维度由 PM 按车型类别评审

### 6.5 新增"其他展现样式"

> **决策点**：如果未来真的需要"简版报告"（如邮件摘要版），那是**第二个展现样式**。

**当前不需要**。YAGNI。

---

## 七、与其他文档的关系

```
展现样式（PC/H5）= 数据怎么渲染
MD 骨架（v3.0）   = 数据怎么组织
字段表            = 数据是什么
数据源            = 数据从哪来
质量保证          = 数据怎么验证
```

**本文件只管"渲染"**——前端、HTML 模板、CSS、设备版式。其他维度归其他文档管。

---

## 八、变更日志

- v1.0（2026-08-27）：初版
- v1.1（2026-08-27）：明确"单一展现样式 + 2 种设备载体"
- v1.2（2026-08-27）：聚焦 1 个标杆车型（Versa），4 维度 / 6 版本 / 4 金融卡 / 3 竞品 / 3 行动入口 全部锁定
