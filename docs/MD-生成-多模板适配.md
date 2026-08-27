# MD 生成 · 多模板适配规范

**文档版本**：v1.0
**创建日期**：2026-08-27
**目的**：定义"模板族（template family）"机制，确保新模板按统一规范加入，不破坏数据流
**关联**：`docs/MD-数据-需求清单.md`（字段）· `docs/CAVI 报告式页面重构方案.md` v2.0

---

## 一、为什么需要"模板族"

现状 templates/ 目录有 2 个**实质不同**的模板：

| 模板 | 内容 | CAVI 4 维度 | 金融产品 | 适用场景 |
|------|------|------------|---------|---------|
| `autocava-pc-v1.html` | Versa 2026 · 西语 MX · 燃油车 | cajuela/consumo/seguridad/ruido | BBVA 36 + Nissan 厂家 | MX 燃油 Sedan |
| `autocava-pc-v1-cn-demo.html` | 电动车 SUV · 中文 | 外观/内饰/智驾/售后 | 48 期金融方案 | CN 电动 SUV |

→ **同一份 MD 骨架，不同的数据填充 = 不同的报告**。但它们不是"翻译关系"，是**两套独立模板**。

引入"模板族"概念，让这种多模板情况有清晰的扩展规则。

---

## 二、模板族定义

### 2.1 什么是模板族

> **模板族** = 一组共享 8 段结构、但在不同维度（语言/市场/能源类型/车型大类）上有差异的 MD 生成配置。

每份 MD 报告绑定一个 `template_id`，模板族决定：
- 西语/中文 eyebrow 是否字面渲染
- CAVI 4 维度的具体标签
- 金融产品的卡类型
- 章节标题语言
- 特定可选字段

### 2.2 模板族 ID 规范

格式：`{device}-{market}-{energy}-{body}-{version}`

| 段 | 含义 | 取值 |
|----|------|------|
| `{device}` | 设备类型 | `pc` / `h5` |
| `{market}` | 市场 | `mx` / `cn` / `co` / `ar` |
| `{energy}` | 能源 | `fuel` / `ev` / `hybrid` / `phev` |
| `{body}` | 车型大类 | `sedan` / `suv` / `hatchback` / `pickup` |
| `{version}` | 版本号 | `v1` / `v2` / ... |

**示例**：
- `pc-mx-fuel-sedan-v1` — PC · MX · 燃油 · Sedan · v1
- `pc-cn-ev-suv-v1` — PC · CN · 电动 · SUV · v1
- `h5-mx-fuel-sedan-v1` — H5 移动版 · MX · 燃油 · Sedan · v1

### 2.3 当前已识别模板族

| template_id | 对应 HTML | 数据状态 |
|-------------|----------|---------|
| `pc-mx-fuel-sedan-v1` | `autocava-pc-v1.html` | 完整（Versa 数据） |
| `h5-mx-fuel-sedan-v1` | `autocava-h5-mx-v1.html` | 完整（Versa 数据） |
| `pc-cn-ev-suv-v1` | `autocava-pc-v1-cn-demo.html` | demo 级（电动车占位数据） |

---

## 三、模板族配置（Template Family Config）

每新增一个模板族，必须在 `template_families.yaml` 登记：

```yaml
# template_families.yaml
families:
  pc-mx-fuel-sedan-v1:
    name: "PC · MX · 燃油 Sedan · v1"
    html_template: "templates/autocava-pc-v1.html"
    language: es
    market: MX
    energy_type: 燃油
    body_type: Sedán
    
    # 段 02..08 的 block-eyebrow（西语/中文字面）
    eyebrows:
      "02": "02 · PRECIO Y FINANCIAMIENTO"
      "03": "03 · CORE SELLING POINTS"
      "04": "04 · CAVI · RESEÑAS"
      "05": "05 · COST"
      "06": "06 · PROTECTION"
      "07": "07 · COMPETITORS"
      "08": "08 · SIGUIENTE PASO"
    
    # 段 02..08 的 block-title（章节标题）
    titles:
      "02": "Precio y planes de financiamiento"
      "03": "Lo que realmente destaca del {model}"
      "04": "Reseñas de usuarios reales"
      "05": "Costo anual de uso"
      "06": "Cobertura post-compra"
      "07": "Si te interesa el {model}, también estás viendo"
      "08": "¿Listo para decidir?"
    
    # CAVI 4 维度定义
    cavi_dimensions:
      - { slug: cajuela,    label_es: "Cajuela",   label_zh: "后备厢" }
      - { slug: consumo,    label_es: "Consumo",   label_zh: "油耗" }
      - { slug: seguridad,  label_es: "Seguridad", label_zh: "安全" }
      - { slug: ruido,      label_es: "Ruido",     label_zh: "噪音" }
    
    # 金融产品配置
    finance_products:
      main_card_label: "Precio sugerido · {version_name}"
      bank_card_label: "Banco · BBVA {term} meses"
      bank_default_term: 36
      factory_card_label: "Financiamiento {brand}"
      trade_in_card_label: "Bono intercambio"
    
    # 行动入口配置
    next_card_titles:
      whatsapp: "Hablar con un asesor"
      test_drive: "Agendar prueba de manejo"
      cavi_ai: "Preguntar a Cavi (AI)"
    
    # 必备全局元素
    required_globals:
      - hero_cta          # 黄底 Consultar planes
      - fixed_bottom      # WhatsApp + Llamar
      - footer

  pc-cn-ev-suv-v1:
    name: "PC · CN · 电动 SUV · v1"
    html_template: "templates/autocava-pc-v1-cn-demo.html"
    language: zh
    market: CN
    energy_type: 纯电
    body_type: SUV
    
    eyebrows:
      "02": "02 · 价格与金融方案"
      "03": "03 · 核心卖点"
      "04": "04 · CAVI · 用户口碑"
      "05": "05 · 用车成本"
      "06": "06 · 购车保障与覆盖"
      "07": "07 · 主要竞品怎么选"
      "08": "08 · 想进一步了解"
    
    titles:
      "02": "价格与金融方案"
      "03": "核心卖点"
      # ... 其他
    
    cavi_dimensions:
      - { slug: diseno,        label_es: "Diseño",     label_zh: "外观设计" }
      - { slug: interior,      label_es: "Interior",   label_zh: "内饰质感" }
      - { slug: ad_asistente,  label_es: "ADAS",       label_zh: "智能驾驶" }
      - { slug: postventa,     label_es: "Postventa",  label_zh: "售后保养" }
    
    finance_products:
      main_card_label: "建议零售价 · {version_name}"
      bank_card_label: "{term} 期金融方案"
      bank_default_term: 48
      factory_card_label: "厂家贴息方案"
      trade_in_card_label: "置换补贴"
    
    next_card_titles:
      whatsapp: "联系销售顾问"
      test_drive: "预约试驾"
      cavi_ai: "咨询 Cavi 智能助手"
    
    required_globals:
      - hero_cta
      - fixed_bottom
      - footer
```

---

## 四、模板族差异点

### 4.1 差异维度

| 维度 | 例子 | 谁决定 |
|------|------|--------|
| **语言** | `es` vs `zh` | `language` |
| **市场** | `MX` vs `CN` | `market` |
| **能源类型** | `燃油` vs `纯电` | `energy_type` |
| **车型大类** | `Sedan` vs `SUV` | `body_type` |
| **金融产品** | BBVA 36 期 vs 48 期方案 | `finance_products` |
| **CAVI 4 维度** | cajuela/consumo vs 外观/内饰 | `cavi_dimensions` |
| **eyebrow 字面** | 西语 `02 · PRECIO...` vs 中文 `02 · 价格...` | `eyebrows` |
| **CTA 文案** | `Calcular →` vs `立即测算 →` | `next_card_titles` |
| **货币 + 千分位** | MXN vs CNY | `market` |

### 4.2 共性维度

不论哪个模板族，以下内容**结构固定**：
- 8 段结构（首屏 + 02..08）
- 8 段的存在与编号（02..08 不能变成 03..09）
- 4 张价格卡（主价 + 银行 + 厂家 + 交换）
- 3 张并排竞品卡（含本车）
- 3 张行动卡（whatsapp / test_drive / cavi_ai）
- spec_strip 3 条 / key_specs 3-4 条 / versions 6 条

---

## 五、新增模板族流程

新增一个模板族（如 `pc-cn-fuel-sedan-v1`）必须经过以下流程：

### 5.1 PM 评审（必须）

- [ ] 必要性：为什么需要新模板族？（不是用现有模板族扩展？）
- [ ] 影响范围：哪些字段会变？哪些会新增？
- [ ] 风险评估：跟现有模板族的数据兼容性
- [ ] 验收标准：何时算"上线"？

### 5.2 模板族配置登记

- [ ] 在 `template_families.yaml` 补全 §三的所有字段
- [ ] 决定 eyebrows / titles 的字面（PM 评审）
- [ ] 决定 cavi_dimensions（评审 CAVI 团队）
- [ ] 决定金融产品配置（评审 营销/产品）

### 5.3 字段表更新

- [ ] 如有新增字段：更新 `MD-数据-需求清单.md`
- [ ] 如有字段语义变化：更新 `MD-数据-需求清单.md` 标注

### 5.4 HTML 模板交付

- [ ] 前端提供 `templates/{template_id}.html`
- [ ] 8 段 eyebrow + title 字面与配置一致
- [ ] 通过前端自测（跨浏览器、响应式）

### 5.5 端到端测试

- [ ] 选 1 个真实车型（最好用过的车）
- [ ] 跑 SKILL 完整流程
- [ ] 自动校验通过
- [ ] 人工审阅通过
- [ ] HTML 渲染对比（如果有旧版本）

### 5.6 灰度上线

- [ ] 灰度 10% 流量 → 监控 24h
- [ ] 灰度 50% 流量 → 监控 24h
- [ ] 全量 100%

---

## 六、模板族 vs 设备/版式

**注意区分**：
- **模板族** = 数据配置（语言/市场/能源/车型）
- **设备/版式** = 前端呈现（PC/H5、桌面/移动）

理论上 PC 和 H5 可以共享同一模板族（因为它们的 MD 数据相同），但 HTML 模板不同。

```
模板族 pc-mx-fuel-sedan-v1
  ├── HTML 模板：autocava-pc-v1.html        (PC 桌面)
  └── HTML 模板：autocava-h5-mx-v1.html    (H5 移动)
```

**当前映射**：
- `pc-mx-fuel-sedan-v1` 映射到 `autocava-pc-v1.html`（PC）
- `h5-mx-fuel-sedan-v1` 映射到 `autocava-h5-mx-v1.html`（H5）
- 两者 MD 数据相同（同一模板族），但 HTML 渲染不同

**未来扩展**：
- `pc-cn-ev-suv-v1` 映射到 `autocava-pc-v1-cn-demo.html`（PC 中文）
- 未来若有 H5 中文版 → 新增 `h5-cn-ev-suv-v1` 模板族，共享 `pc-cn-ev-suv-v1` 数据

---

## 七、模板族废弃

当一个模板族不再使用时：

1. 在 `template_families.yaml` 标记 `deprecated: true`
2. 现有报告保留 30 天（提供切换期）
3. 30 天后**禁止**用该模板族生成新报告
4. 6 个月后可考虑物理删除相关 HTML 模板

---

## 八、特殊场景

### 8.1 同一车型多模板族渲染

同一车型（如 Versa 2026）可以同时在 PC 和 H5 显示：
- 共享同一份 MD 数据
- 渲染时按设备加载不同 HTML 模板
- → 只需生成**一份 MD**，渲染阶段分流

### 8.2 跨语言复用

未来如需要"同一车型，西语 + 中文"双版本：
- 方案 A：生成两份独立 MD（西语一份 / 中文一份）→ 数据双倍维护
- 方案 B：生成一份 MD（中文），渲染时翻译 HTML（不推荐，破坏数据可靠）
- **方案 C**：生成两份 MD，**结构相同但语言字段不同**（推荐）

方案 C 实现：
- 同一份 `series_id` + `market` 触发
- `lang` 参数：`es` / `zh`
- 数据源基本相同，AI 文案按语言生成
- 输出：`reports/es/{model}-{year}.md` + `reports/zh/{model}-{year}.md`

### 8.3 新能源 + 燃油车混市场

未来市场可能有混车型：
- CN 市场既有燃油也有电动
- 同一 `market=CN` 但 `energy_type` 不同
- → 用 `template_id` 区分（如 `pc-cn-fuel-sedan-v1` vs `pc-cn-ev-suv-v1`）

---

## 九、检查清单

新增模板族时，确保：

- [ ] `template_families.yaml` 完整登记
- [ ] 字段差异在 `MD-数据-需求清单.md` 标注
- [ ] HTML 模板交付且与配置一致
- [ ] 端到端跑通 1 个真实车型
- [ ] 自动校验全部 pass
- [ ] 人工审阅通过
- [ ] 灰度上线 10% → 50% → 100%
- [ ] 同步更新 `skills/cavi-guide-gen/SKILL.md` 的输入参数说明
- [ ] 同步更新 `examples.md` 提供新模板族的参考示例

---

## 十、相关文档

- *字段表：`docs/MD-数据-需求清单.md`*
- *总览：`docs/MD-生成-总览.md`*
- *质量保证：`docs/MD-生成-质量保证.md`*
- *前端重构方案（已升级到 v2.0）：`docs/CAVI 报告式页面重构方案.md`*
