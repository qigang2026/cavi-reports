# AUTOCAVA 购车报告需求

**需求编号**：PRD-AUTO-2026-0827

**文档版本**：v1.0

**创建日期**：2026-08-27

**产品经理**：叶启刚

**状态**：待评审

## 一、需求概述

### 1.1 背景

AUTOCAVA 目前用户在车系页面的线索提交转化较低，通过购车报告作为钩子来引导用户提交线索获取报告，从而达到车系页用户线索提交转化的目标。
购车报告以 H5 形式呈现给用户，确保后续：
1. 购车报告的内容更新及时
2. 报告的浏览记录可查询
3. 报告中植入的 cavi 能承接通过分享进入报告页面的用户咨询

### 1.2 目标

| 目标 | 描述 | 验收标准 |
|---|---|---|
| 文档标准化 | 制定 MD 文档规范 | AI 生成 10+ 车型 MD 文档，格式一致率 > 95% |
| model 体系建立 | 车型维度唯一标识 | 每个车型有唯一 model，支持埋点追溯 |
| Skill 规范化 | 定义 AI 生成流程 | cavi-guide-gen Skill 生成文档成功率 > 99% |

### 1.3 范围

**本期范围**：
- 制定 MD 文档规范
- 更新 cavi-guide-gen Skill
- 覆盖 cava 平台所有车型
- 明确购车报告 H5 页面样式
- 明确基础埋点内容

---

## 二、功能需求

### 2.1 MD 文档规范

#### 2.1.1 文件命名

**命名规则**：`{series_id}-{车型名（小写连字符）}-{年款}.md`

```
{series_id}-{model}-{year}.md
```

**示例**：
- `zh/nissan-versa-2026.md`

- 前端根据 MD 文档结构读取内容并进行页面展示渲染

- 每个报告的 URL：`https://www.autocava.com.mx/auto/series/{series_id}/report`

#### 2.1.2 Frontmatter 必填字段

```yaml
---
title: Nissan Versa 2026 购车指南
series_id: 356              # CAVI 系统车系 ID
model: nissan-versa         # 车型名（小写、连字符）
year: 2026
market: MX                  # 市场：MX=墨西哥
lang: es                    # 语言：es=西班牙语 / zh=中文
energy_type: 燃油           # 燃油 / 混动 / 纯电
body_type: Sedán            # 车身形式
generated_at: 2026-08-27
---
```

#### 2.1.3 文档章节结构（v3.0）

> 8 段结构 = 首屏（Hero + Spec strip + Trim selector + Compare bar）+ 02..08 七个内容段。
> "概览"和"参数"不再分两段；参数融入首屏 Spec strip，身份/定位作为 Hero 元信息。

| 段 | block-eyebrow（西语） | block-title | 必须包含 |
|----|---------------------|------------|----------|
| 首屏 | (无 eyebrow，Hero 段) | 车名 + Spec strip + Trim selector + Compare bar | 车名、3 核心数据（Spec strip）、版本选择、对比条 |
| 02 | `02 · PRECIO Y FINANCIAMIENTO` | Precio y planes | 主价卡（distribuidor 深色）+ 3 补充卡（BBVA / Nissan financing / 交换补贴）+ cta-bar "¿Calcular tu cuota?" |
| 03 | `03 · CORE SELLING POINTS` | Lo que destaca | 3-4 个核心数据指标（大数字+短标签）+ 1 段洞察（配大图） |
| 04 | `04 · CAVI · RESEÑAS` | Reseñas de usuarios | CAVI 综合评分 + 推荐率 + 4 维度细分（cajuela / consumo / seguridad / ruido）+ 精选评论 2-3 条 |
| 05 | `05 · COST` | Costo anual | 年度总成本 + 5 项并列明细（fuel / insurance / maintenance / depreciation / tax）+ 省钱建议 |
| 06 | `06 · PROTECTION` | Cobertura post-compra | 质保年限+里程、保养包次数+里程、服务网络规模、支持项目 Tag |
| 07 | `07 · COMPETITORS` | ¿Qué comparar? | 3 张并排卡（VW Virtus / Kia K3 / Mazda 2 类直接对标）+ ALTERNATIVA / PREMIUM 标签 |
| 08 | `08 · SIGUIENTE PASO` | ¿Listo para decidir? | 3 张 next-card：WhatsApp 顾问 / 预约试驾 / Cavi AI |

**配套全局元素**（不属于 8 段之一，但所有报告必备）：
- Hero CTA：黄底"Consultar planes"按钮（直链 finance apply）
- Fixed bottom：窄 WhatsApp 图标 + 橙色"Llamar"按钮
- 章节内 cta-bar：每个 block 末尾的可量化小 CTA（"¿Calcular tu cuota? · Compara hasta 3 bancos · 1 min"）

### 2.2 Skill 规范

#### 2.2.1 输入参数

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| series_id | string | ✅ | - | 车系 ID |
| market | string | ✅ | MX | 市场代码 |
| lang | string | ❌ | es | 语言代码 |
| force_update | boolean | ❌ | false | 强制更新 |

#### 2.2.2 输出规范

```json
{
  "success": true,
  "file_path": "/reports/zh/nissan-versa-2026.md",
  "model": "nissan-versa",
  "series_id": 356,
  "cava_score": 78,
  "version_count": 3,
  "generated_at": "2026-08-27T12:00:00Z"
}
```

#### 2.2.3 错误处理

| 错误码 | 说明 | 处理方式 |
|---|---|---|
| E001 | series_id 无效 | 返回错误，提示有效 ID 列表 |
| E002 | 网络请求失败 | 重试 3 次，失败返回错误 |
| E003 | 数据不完整 | 返回警告，标注缺失字段 |
| E004 | 文件写入失败 | 返回错误，提示磁盘空间 |

### 2.3 车型标识规范

#### 2.3.1 model 字段生成规则

model = {品牌（小写）}-{车系（小写）}

示例：nissan-versa、volkswagen-golf

#### 2.3.2 文件路径规则

/reports/{lang}/{series_id}-{model}-{year}.md

示例：/reports/zh/nissan-versa-2026.md

---

## 三、非功能需求

### 3.1 性能

- 单个 MD 文档生成时间 < 30 秒
- 批量生成 10 个车型 < 5 分钟

### 3.2 质量

- MD 文档格式一致率 > 95%
- 关键字段（价格、评分）准确率 > 99%
- 文档完整性（无缺失章节）> 99%

### 3.3 可维护性

- Skill 版本管理，支持回滚
- 文档模板版本管理
- 错误日志完整记录

---

## 四、依赖关系

### 4.1 前置需求

| 需求编号 | 需求名称 | 状态 |
|---|---|---|
| - | autocava.com.mx 数据接口 | ✅ 已完成 |
| - | cavi-guide-gen Skill 基础能力 | ✅ 已完成 |

### 4.2 后续需求

| 需求编号 | 需求名称 | 计划日期 |
|---|---|---|
| PRD-AUTO-2026-0901 | 前端 MD 解析组件开发 | 2026-09-15 |
| PRD-AUTO-2026-0902 | 埋点系统接入 | 2026-10-15 |

### 4.3 外部依赖

| 依赖方 | 依赖内容 | 交付日期 |
|---|---|---|
| 前端团队 | MD 解析组件接口定义确认 | 2026-09-01 |
| 数据团队 | 埋点方案确认 | 2026-09-15 |
| 设计团队 | 购车报告 UI 规范 | 2026-09-10 |

---

## 五、验收标准

### 5.1 交付物

| 交付物 | 格式 | 说明 |
|---|---|---|
| Skill 规范 | MD 文件 | cavi-guide-gen/SKILL.md |
| 标准模板 | MD 文件 | cavi-guide-gen/assets/standard-template-v2.md |
| 示例文档 | MD 文件 | cavi-guide-gen/examples.md |

### 5.2 验收检查点

- [ ] MD 文档规范评审通过
- [ ] model 生成规则评审通过
- [ ] Skill 按规范生成 10 个车型 MD 文档
- [ ] 文档格式一致率 > 95%
- [ ] model 生成正确率 100%
- [ ] 关键字段无缺失

---

## 六、里程碑

| 阶段 | 内容 | 目标日期 | 负责人 |
|---|---|---|---|
| M1 | 需求评审通过 | 2026-08-30 | PM |
| M2 | 规范文档定稿 | 2026-09-05 | PM |
| M3 | Skill 更新完成 | 2026-09-10 | AI |
| M4 | 10 个车型 MD 生成 | 2026-09-15 | AI |
| M5 | 交付验收 | 2026-09-20 | PM |

---

## 七、风险与对策

| 风险 | 影响 | 概率 | 对策 |
|---|---|---|---|
| AI 生成格式不一致 | 高 | 中 | 加强模板约束，增加检查点，并且进行人工检查 |
| 数据源不稳定（运营后台更新了数据） | 高 | 低 | 监控后台车系数据维度表若存在数据更新 AI 进行内容识别确认是否同步更新购车报告页面内容 |

## 八、附录

### 8.1 相关文档

| 文档 | 链接 |
|---|---|
| cavi-guide-gen Skill | skills/cavi-guide-gen/SKILL.md |
| 标准模板 | skills/cavi-guide-gen/assets/standard-template-v2.md |
| 示例文档 | skills/cavi-guide-gen/examples.md |
| 原型地址 | https://qigang2026.github.io/cavi-reports/zh/nissan-versa-2026.html |

---

**评审记录**：

| 日期 | 评审人 | 意见 | 状态 |
|---|---|---|---|
| - | - | - | 待评审 |
