# MD 生成 · 展现样式规范

**文档版本**：v1.0
**创建日期**：2026-08-27
**目的**：明确本项目**只有一个报告展现样式**，但有 **2 种渲染载体**（PC 桌面 / H5 移动）
**关联**：`docs/MD-数据-需求清单.md`（字段）· `docs/CAVI 报告式页面重构方案.md` v2.0

---

## 一、核心结论

> **本项目只有 1 个报告展现样式。**
>
> 报告的"长什么样"由这 1 个样式决定；
> "渲染到哪"由设备载体（PC 或 H5）决定。
>
> 不存在"多模板族"。

**反过来说**：
- ❌ 不存在"中文模板" vs "西语模板"——同一展现样式下，只是字段值不同
- ❌ 不存在"燃油车模板" vs "电动车模板"——同上
- ❌ 不存在"MX 模板" vs "CN 模板"——同上

---

## 二、唯一展现样式

| 项 | 值 |
|---|---|
| **样式 ID** | `cavi-report-v1` |
| **来源** | `reports/templates/autocava-pc-v1.html`（PC 版）|
| **章节数** | 8 段（首屏 + 02..08）|
| **block-eyebrow** | 西语字面锁定（见 §三）|
| **适用市场** | MX（其他市场未来扩展）|
| **适用语言** | 西语（es）— 中文版未来扩展 |

**章节骨架**（与 `standard-template-v3.md` 一一对应）：

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

**校验**：`MD-生成-质量保证.md` §4.2 字面校验失败 = 阻塞。

> 当前**只有西语版本**的 eyebrow。中文/英文版本需新增第二套字面，并先经前端 + PM 评审。

---

## 四、2 种渲染载体（设备版式）

| 载体 | HTML 模板 | 用途 | 数据 |
|------|----------|------|------|
| **PC 桌面** | `reports/templates/autocava-pc-v1.html` | 桌面浏览器 | 与 H5 **同一份 MD** |
| **H5 移动** | `reports/autocava-h5-mx-v1.html` | 手机浏览器 | 与 PC **同一份 MD** |

**关系**：
```
一份 MD 数据（reports/{lang}/{model}-{year}.md）
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

## 五、关于 templates/ 目录的现状

| 文件 | 性质 | 处理 |
|------|------|------|
| `autocava-pc-v1.html` | **PC 模板（唯一）** | 保留 |
| `autocava-h5-mx-v1.html` | **H5 模板（移动版）** | 保留（在 reports/ 根，不在 templates/）|
| `autocava-pc-v1-cn-demo.html` | **中文 demo 数据示例**（不是模板）| 保留作参考（demo 用同一展现样式填充）|
| `autocava-pc-mx-v1.html` | **与 pc-v1.md5 完全相同**（命名重复）| **删除** |

> **重要**：cn-demo 虽然叫"模板"，但它是**同一展现样式下用另一份数据填的 demo**，不是另一个模板。

---

## 六、未来扩展（如需要）

### 6.1 新增语言

- 当前：es（西语）
- 未来：zh（中文）/ en（英文）
- 实施：
  1. 新增一套 eyebrow 字面（`02 · 价格与金融方案` 等）
  2. 决定中文 4 维度标签（外观/内饰/智驾/售后？vs 复用西语）
  3. 中文 AI 文案风格规范
  4. 新增/修改字段表中的"label_zh"列

### 6.2 新增市场

- 当前：MX
- 未来：CN / CO / AR
- 实施：
  1. 新增市场级配置（货币、电话、WhatsApp 号、银行产品等）
  2. 决定每个市场的 4 维度标签
  3. 决定每个市场的 eyebrow 字面（市场可能跟语言耦合：CN→zh，CO→es）

### 6.3 新增能源类型 / 车型

- 不需要新增模板
- 同一展现样式下，spec_strip / key_specs / cavi_dimensions 的具体值随车型变化
- 4 维度由 PM 按车型类别预先定义在 `cavi_dimensions.yaml`

### 6.4 新增"其他展现样式"（如需）

> **决策点**：如果未来真的需要"简版报告"（如邮件摘要版），那是**第二个展现样式**，需要：
> 1. PM 评审：必要性、字段集
> 2. 新建 `reports/templates/cavi-report-summary-v1.html`
> 3. 新建 SKILL 入口（如 `cavi-guide-summary`）
> 4. 字段表新增"摘要版"差异列

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

- v1.0（2026-08-27）：初版，明确"单一展现样式 + 2 种设备载体"
