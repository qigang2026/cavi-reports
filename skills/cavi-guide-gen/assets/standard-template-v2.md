# CAVI 购车手册标准模板 v2.0

**版本**：2.0
**更新日期**：2026-08-27
**用途**：AI 生成 MD 文档的标准骨架，映射前端 HTML Tab 结构

---

## 设计原则

1. **章节固定**：前端按 Tab 顺序渲染，必须有对应 MD 章节
2. **变量占位符**：`{{变量名}}` 格式，AI 填充实际数据
3. **可选区块**：用 `{{optional_xxx}}` 标记无数据时可删除
4. **禁止修改**：章节顺序、层级、变量名不可变更

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
---
```

---

## Tab 1：概览 (Overview)

```
# {{title}}

> **CAVI 智能解读 · 真实数据来源**
> "{{one_liner}}" — CAVI

---

## 身份识别

| 维度 | 数据 |
|------|------|
| 品牌 | {{brand}} |
| 车系 | {{series}} |
| 年款 | {{year}} |
| 能源类型 | {{energy_type}} ({{energy_type_en}}) |
| 车身形式 | {{body_type}} |
| 座位数 | {{seats}} 人 |
| 行李箱空间 | {{boot_space}} |
| 驱动方式 | {{drivetrain}} |
| 发动机 | {{engine}} |
| 最大功率 | {{horsepower}} |
{{identity_extra}}

### 用户价值定位

!!! success "适合人群"
{{target_users}}

> "{{ai_recommendation_quote}}"
> *({{ai_recommendation_translation}})*

---

{{sales_data_section}}
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{one_liner}}` | string | 一句话定位 |
| `{{brand}}` | string | 品牌名 |
| `{{series}}` | string | 车系名 |
| `{{year}}` | number | 年款 |
| `{{energy_type}}` | string | 能源类型中文 |
| `{{energy_type_en}}` | string | 能源类型西班牙语 |
| `{{body_type}}` | string | 车身形式 |
| `{{seats}}` | number | 座位数 |
| `{{boot_space}}` | string | 行李箱空间 |
| `{{drivetrain}}` | string | 驱动方式 |
| `{{engine}}` | string | 发动机描述 |
| `{{horsepower}}` | string | 马力 |
| `{{target_users}}` | list | 适合人群列表 |
| `{{ai_recommendation_quote}}` | string | CAVI 推荐原话 |
| `{{ai_recommendation_translation}}` | string | 中文翻译 |
| `{{sales_data_section}}` | block | 可选：销量数据 |

---

## Tab 2：参数 (Specs)

```
## 详细参数

### 基本信息

| 参数 | 数值 |
|------|------|
| 车型 | {{model_type}} |
| 长×宽×高(mm) | {{dimensions}} |
| 轴距(mm) | {{wheelbase}} |
| 整备质量(kg) | {{kerb_weight}} |
| 油箱容积(L) | {{fuel_tank}} |
| 后备箱容积(L) | {{boot_volume}} |

### 动力系统

| 参数 | 数值 |
|------|------|
| 发动机 | {{engine_detail}} |
| 最大马力 | {{horsepower_detail}} |
| 最大扭矩 | {{torque_detail}} |
| 变速箱 | {{transmission}} |
| 综合油耗 | {{fuel_economy}} |
| 排放标准 | {{emission_standard}} |

### 安全配置

| 安全功能 | 状态 | 说明 |
|----------|------|------|
{{safety_features}}

> "{{safety_quote}}" — CAVI

### 科技配置

{{smart_features}}
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{model_type}}` | string | 车型分类 |
| `{{dimensions}}` | string | 长×宽×高 |
| `{{wheelbase}}` | string | 轴距 |
| `{{kerb_weight}}` | string | 整备质量 |
| `{{fuel_tank}}` | string | 油箱容积 |
| `{{boot_volume}}` | string | 后备箱容积 |
| `{{engine_detail}}` | string | 发动机详细 |
| `{{horsepower_detail}}` | string | 马力详细 |
| `{{torque_detail}}` | string | 扭矩详细 |
| `{{transmission}}` | string | 变速箱 |
| `{{fuel_economy}}` | string | 综合油耗 |
| `{{emission_standard}}` | string | 排放标准 |
| `{{safety_features}}` | table | 安全配置表格 |
| `{{safety_quote}}` | string | 安全评价 |
| `{{smart_features}}` | list | 科技配置列表 |

---

## Tab 3：金融 (Finance)

```
## 价格与金融方案

{{financing_default_plan}}

### 首付{{down_payment_percent}} · {{loan_term}}期 推荐方案

| 项目 | 金额 |
|------|------|
| 裸车价 | {{guide_price}} |
| 首付 | {{down_payment}} |
| 贷款额 | {{loan_amount}} |
| 利率 | {{interest_rate}} |
| 月供 | {{monthly_payment}}/月 |

{{alternative_plans}}

### 贷款材料

{{loan_requirements}}

### 购车礼包

{{purchase_bonus}}

{{financing_cta}}
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `{{financing_default_plan}}` | string | 推荐方案标题 |
| `{{down_payment_percent}}` | string | 首付比例 |
| `{{loan_term}}` | string | 贷款期数 |
| `{{guide_price}}` | string | 裸车价 |
| `{{down_payment}}` | string | 首付金额 |
| `{{loan_amount}}` | string | 贷款额 |
| `{{interest_rate}}` | string | 利率 |
| `{{monthly_payment}}` | string | 月供 |
| `{{alternative_plans}}` | block | 可选：其他方案 |
| `{{loan_requirements}}` | list | 贷款材料 |
| `{{purchase_bonus}}` | list | 购车礼包 |
| `{{financing_cta}}` | block | 行动按钮 |

---

## Tab 4：卖点 (Highlights)

```
## 核心卖点

{{#each highlights}}
### {{this.title}}

{{this.description}}

{{#if this.stats}}
> {{this.stats}}
{{/if}}
{{/each}}

---

## 版本配置差异

### {{base_version}} 版本（入门）

```
{{base_features}}
```

### {{mid_version}} 版本（中配 +{{mid_price_diff}} 起）

```
{{mid_features}}
```

### {{top_version}}（顶配）

```
{{top_features}}
```
```

---

## Tab 5：口碑 (Reviews)

```
## 用户口碑

{{#each reviews}}
### {{this.user_name}} {{this.rating_stars}}

**{{this.version}}** · 行驶 {{this.mileage}}km

**{{this.rating_stars}}**

 "{{this.content}}"

{{#each this.tags}}
- {{this}}
{{/each}}
{{/each}}

**{{overall_score}}** 综合评分

**{{recommendation_rate}}** 推荐度
```

---

## Tab 6：成本 (Cost)

```
## 用车成本

### 综合油耗

按年行驶 {{annual_km}}km 计算

**{{annual_fuel_cost}}/年**

油耗成本低于 {{fuel_cost_percent}}% 的同级车型

### 年度用车费用明细

| 项目 | 金额 |
|------|------|
| 燃油费 | {{annual_fuel}} |
| 保险费用 | {{annual_insurance}} |
| 保养费用 | {{annual_maintenance}} |
| 停车费用 | {{annual_parking}} |
| 洗车费用 | {{annual_car_wash}} |

### 保养周期

{{maintenance_schedule}}

### 省钱建议

{{cost_saving_tips}}
```

---

## Tab 7：保障 (Warranty)

```
## 购车保障与服务覆盖

### 整车质保

{{warranty_info}}

{{warranty_details}}

### 售后服务网络

{{service_network}}

### 车主权益

{{owner_benefits}}

### 智能互联

{{smart_connect}}
```

---

## Tab 8：竞品 (Competitors)

```
## 竞品车型

> "{{competitor_intro_quote}}"

{{#each competitors}}
### {{this.name}}

{{this.name_local}}

{{this.specs}}

{{this.price}}

{{this.monthly_payment}}

**适合：{{this.suitable_for}}**

{{#if this.highlight}}
✦ 亮点
{{this.highlight}}
{{/if}}

{{#if this.improvement}}
○ 待提升
{{this.improvement}}
{{/if}}
{{/each}}

### 对比项

| 对比项 | {{this.car}} | {{competitor1}} | {{competitor2}} |
|--------|--------------|------------------|------------------|
{{comparison_rows}}
```

---

## Tab 9：选购 (Purchase)

```
## 选购指南

> "{{purchase_guide_quote}}"

### 按需求推荐

{{recommendations_by_need}}

### 按预算推荐

{{recommendations_by_budget}}

### 选车建议

{{purchase_tips}}

---

## 行动入口

| 入口 | 链接 | 用途 |
|------|------|------|
{{action_buttons}}

---

## 快速问答

你可以继续向 CAVI 询问：

{{quick_questions}}

---

*📊 数据来源：[AutoCava CAVI](https://www.autocava.com.mx/cavi) · {{series}} Series ID: {{series_id}} · 生成时间: {{generated_at}}*
```

---

## 可选区块说明

| 区块 | 说明 | 使用条件 |
|------|------|----------|
| `{{sales_data_section}}` | 销量数据 | 有销量数据时保留 |
| `{{alternative_plans}}` | 备选金融方案 | 有多个方案时保留 |
| `{{smart_features}}` | 科技配置 | 有配置列表时保留 |
| `{{cost_saving_tips}}` | 省钱建议 | 有建议时保留 |

---

## 生成检查清单

- [ ] 所有 `{{变量}}` 已替换为实际值
- [ ] Frontmatter 完整
- [ ] 9 个 Tab 章节齐全
- [ ] 表格格式正确
- [ ] 无未填充的可选区块
- [ ] 行动入口链接有效
