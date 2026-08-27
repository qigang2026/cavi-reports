# MD 生成 · 数据源与可靠性

**文档版本**：v1.0
**创建日期**：2026-08-27
**目的**：明确 MD 文档每个数据点的来源、缓存策略、失败处理，**确保生成结果可靠可追溯**
**关联**：`docs/MD-数据-需求清单.md`（字段权威表）· `docs/MD-生成-质量保证.md`（验收）

---

## 一、数据源分层

```
┌────────────────────────────────────────────────────┐
│  ① 权威数据源（强一致）                              │
│     CAVI 系统 · 销售系统 · 银行/厂家 API            │
│     必须 100% 准确，缺失则生成失败                    │
└────────────────────────────────────────────────────┘
                         ↓
┌────────────────────────────────────────────────────┐
│  ② 半权威数据源（弱一致）                            │
│     评论库 · 营销中台 · 资产 CDN                     │
│     缺失可降级，标"未知"                             │
└────────────────────────────────────────────────────┘
                         ↓
┌────────────────────────────────────────────────────┐
│  ③ 派生数据（计算得出）                              │
│     月供 / 占比 / 优惠后价格 / 星级                 │
│     由 ① ② 计算，纯函数                             │
└────────────────────────────────────────────────────┘
                         ↓
┌────────────────────────────────────────────────────┐
│  ④ AI 文案（自然语言生成）                           │
│     副标题 / 洞察 / 推荐语                          │
│     受风格规范约束，不许编数字                        │
└────────────────────────────────────────────────────┘
```

**原则**：层级越靠下，AI 自由度越大；层级越靠上，AI 不能动。

---

## 二、数据源清单

### 2.1 CAVI 内部 API（车系 + CAVI 评分）

| 字段组 | API Endpoint | 缓存 TTL | 失败行为 |
|--------|-------------|----------|---------|
| `series_id` / `model` / `year` / `market` / `body_type` | `GET /api/v1/series/{series_id}` | 24h | **失败 = 不生成** |
| `energy_type` | `GET /api/v1/series/{id}/specs` | 24h | **失败 = 不生成** |
| `versions[]` | `GET /api/v1/series/{id}/versions` | 6h | **失败 = 不生成** |
| `spec_strip` | `GET /api/v1/series/{id}/key-specs?top=3` | 24h | 失败 → 兜底通用 3 数据 |
| `key_specs` | `GET /api/v1/series/{id}/key-specs?top=4` | 24h | 失败 → 兜底通用 4 数据 |
| `cavi_score` | `GET /api/v1/cavi/score/{series_id}` | 24h | 失败 → 跳过 CAVI 段，标"暂无评分" |
| `cavi_recommend` | 同上 | 24h | 失败 → `null` |
| `cavi_dimensions` | `GET /api/v1/cavi/dimensions/{series_id}` | 24h | 失败 → 兜底 4 通用维度（外观/动力/舒适/价格）|
| `cavi_meta` | `GET /api/v1/cavi/meta/{series_id}` | 24h | 失败 → 跳过 |
| `featured_reviews` | `GET /api/v1/cavi/reviews/{series_id}?top=3&sort=helpful` | 1h | 失败 → 跳过 |
| `competitors` | `GET /api/v1/cavi/competitors/{series_id}?top=3&range=10pct` | 24h | 失败 → 跳过段 07 |

### 2.2 销售系统（月销量 + 排名）

| 字段组 | API Endpoint | 缓存 TTL | 失败行为 |
|--------|-------------|----------|---------|
| `monthly_sales` | `GET /api/v1/sales/monthly/{series_id}` | 24h | 失败 → 兜底"无销量数据"|
| `rank_text` | `GET /api/v1/sales/rank/{series_id}` | 24h | 失败 → AI 自动生成 |

### 2.3 银行 / 厂家 API（金融方案）

| 字段组 | API Endpoint | 缓存 TTL | 失败行为 |
|--------|-------------|----------|---------|
| `finance_cards.bank` | `GET /api/v1/finance/banks?series_id={id}&term=36` | 6h | 失败 → 跳过银行卡，AI 提示"暂无方案" |
| `finance_cards.factory` | `GET /api/v1/finance/factory?series_id={id}` | 6h | 失败 → 跳过厂家卡 |
| `finance_cards.trade_in` | `GET /api/v1/marketing/trade-in?series_id={id}` | 1h | 失败 → 跳过交换卡 |
| `finance_cards.main` | 派生（取 `versions[is_recommended].msrp`）| 计算 | 缺失推荐版本 = 不生成 |

### 2.4 评论库（精选评论）

| 字段组 | API Endpoint | 缓存 TTL | 失败行为 |
|--------|-------------|----------|---------|
| `featured_reviews[]` | `GET /api/v1/cavi/reviews/{series_id}?top=3` | 1h | 失败 → 段 04 仍渲染但无评论卡片 |
| 评论筛选 | `sort=helpful&min_stars=4&verified=true` | - | - |

### 2.5 资产 CDN（车图）

| 字段 | URL Pattern | 缓存 TTL | 失败行为 |
|------|-------------|----------|---------|
| Hero 车图 | `https://cdn.autocava.com.mx/car/series/{hash}.png` | 7d | 失败 → 隐藏车图（仍渲染其他元素）|
| 竞品车图 | `https://cdn.autocava.com.mx/series/white/{id}.png` 或 `/car/series/{hash}.png` | 7d | 失败 → 显示占位灰卡 |
| Logo | `https://cdn.autocava.com.mx/_nuxt/asset/logo.v3.{hash}.svg` | 30d | 失败 → 文字 logo |

### 2.6 营销中台（优惠 + 政策）

| 字段组 | API Endpoint | 缓存 TTL | 失败行为 |
|--------|-------------|----------|---------|
| `bonus_trade_in` | `GET /api/v1/marketing/bonuses?series_id={id}` | 1h | 失败 → 0 |
| `vigencia` / `applies_to` | 同上 | 1h | 失败 → 隐藏 |
| `hero_cta.url` | 模板级常量 | 静态 | 缺失 = 不生成 |

### 2.7 厂家数据（质保 + 服务网络）

| 字段 | API Endpoint | 缓存 TTL | 失败行为 |
|------|-------------|----------|---------|
| `warranty.*` | `GET /api/v1/manufacturer/warranty?brand={b}&model={m}&year={y}` | 7d | 失败 → 兜底默认值（见 §三）|
| `maintenance.*` | 同上 | 7d | 失败 → 兜底 0 |
| `service_network.*` | `GET /api/v1/manufacturer/network?brand={b}` | 30d | 失败 → 0 |

---

## 三、字段兜底默认值表

| 字段 | 缺失时的兜底 |
|------|------------|
| `cavi_score` | `null` + 段 04 标题改为"用户口碑（暂无评分）" |
| `cavi_recommend` | `null` + 不显示推荐率 |
| `cavi_verdict` | `"Sin datos suficientes"`（西语）/ `"暂无数据"`（中文）|
| `cavi_dimensions` | 兜底 4 维度（外观/动力/舒适/价格），每项 3.5 |
| `featured_reviews` | `[]` + 段 04 隐藏评论卡片 |
| `competitors` | `[]` + 段 07 显示"暂无竞品数据"|
| `versions[].monthly` | `0` + UI 显示"咨询"|
| `finance_cards.bank` | `null` + 银行卡位置显示"联系经销商"|
| `finance_cards.factory` | `null` + 厂家卡位置显示"联系经销商"|
| `finance_cards.trade_in` | `null` + 交换卡位置显示"暂无"|
| `bonus_trade_in` | `0` |
| `monthly_sales` | `null` + Hero 描述不引用销量 |
| `rank_text` | `null` + Hero 描述不引用排名 |
| `warranty.years` | `3`（行业默认）|
| `warranty.mileage_km` | `100000`（行业默认）|
| `service_network.count` | `0` + 不显示 |
| `support_features` | `[]` + 段 06 显示"详见厂家页面"|
| `annual_cost.total` | `null` + 段 05 显示"暂无数据"|
| `saving_tips` | `[]` + 段 05 不显示省钱建议 |
| `hero.image_url` | `null` + 隐藏车图（Hero 仍渲染）|
| `competitor.image_url` | `null` + 占位灰卡 |
| `next_cards[]` | `[]` + 段 08 显示"联系经销商"通用按钮 |

**兜底原则**：
- **结构数据**（价格、版本、CAVI 分）缺失 → 跳过该段，不生成兜底文
- **非结构数据**（描述、洞察、推荐语）缺失 → 显示"暂无"占位
- **影响购买决策的字段**（价格、版本、CAVI 总分）缺失 → **不生成报告**，触发告警

---

## 四、缓存策略

### 4.1 缓存层级

```
┌──────────────┐
│  L1: 内存     │  进程内（同一请求）  TTL: 请求内
└──────┬───────┘
       ↓ miss
┌──────────────┐
│  L2: Redis   │  跨进程（同一服务）  TTL: 见 §二
└──────┬───────┘
       ↓ miss
┌──────────────┐
│  L3: 源 API  │  外部数据源          无缓存
└──────────────┘
```

### 4.2 缓存 Key 规范

```
cavi:series:{series_id}:v{schema_version}              # 基础车系
cavi:specs:{series_id}:top{3|4}                       # 关键数据
cavi:score:{series_id}                                 # CAVI 评分
cavi:dimensions:{series_id}                            # CAVI 4 维度
cavi:reviews:{series_id}:top3                          # 精选评论
sales:monthly:{series_id}                              # 月销量
sales:rank:{series_id}                                 # 排名
finance:bank:{series_id}:{term}                        # 银行方案
finance:factory:{series_id}                            # 厂家方案
marketing:trade-in:{series_id}                         # 交换补贴
manufacturer:warranty:{brand}:{model}:{year}           # 质保
manufacturer:network:{brand}                           # 服务网络
asset:car-image:{series_id}                            # 车图
```

### 4.3 缓存失效

| 触发条件 | 失效范围 |
|---------|---------|
| CAVI 评分重算完成 | `cavi:score:*` + `cavi:dimensions:*` + `cavi:reviews:*` |
| 价格更新（厂家/经销）| `finance:*` + `versions:*` + `cavi:series:*` |
| 厂家政策变化 | `manufacturer:*` |
| 评论人工干预 | `cavi:reviews:*` |
| 手动强制刷新 | 全部 `cavi:*` |

---

## 五、失败重试策略

### 5.1 单字段级重试

```
读取字段 X →
  try 1 (立即)         → 失败
  try 2 (200ms 后)     → 失败
  try 3 (1000ms 后)    → 失败
  → 兜底默认值（按 §三）
```

### 5.2 整体重试

**触发**：3+ 个核心字段同时失败（车系、版本、CAVI 任一失败）

**行为**：
- 等待 30s
- 重试 1 次
- 仍失败 → **不生成报告**，写入 `error_log`，触发告警
- 告警通知：PM + 后端 oncall

### 5.3 部分失败的处理

| 失败字段数 | 处理 |
|-----------|------|
| 0 | 正常生成 |
| 1-2 个非核心 | 正常生成 + 字段标"暂无"|
| 1-2 个核心 | 正常生成 + 告警（PM 关注）|
| 3+ 个核心 | 阻塞生成 + 立即告警 |
| 价格/版本缺失 | 阻塞 + 告警（**不允许无价格报告上线**）|

---

## 六、数据新鲜度

### 6.1 新鲜度等级

| 等级 | 字段 | 更新频率 | 缓存 TTL |
|------|------|---------|---------|
| **实时** | 价格、库存、利率 | 每次生成时拉取 | ≤ 1h |
| **准实时** | 评论、月销量、CAVI 评分 | 每日刷新 | ≤ 24h |
| **周级** | 质保、服务网络、厂家政策 | 每周刷新 | ≤ 7d |
| **月级** | 销量排名、年度成本基准 | 每月刷新 | ≤ 30d |
| **静态** | 模板常量、章节文案、Logo | 永久 | 永不过期（除非手动改）|

### 6.2 新鲜度策略

- **价格 / 销量 / CAVI**：每次生成时检查缓存，过期则重新拉取
- **质保 / 网络**：默认使用缓存，每月一次 cron 刷新
- **过期数据警示**：若关键字段缓存超过 2× TTL，报告顶部加 "数据更新于 X 天前" 提示

### 6.3 历史快照

每次生成 MD 时，保存"数据快照"用于回放：

```yaml
# data_snapshot.yaml
generated_at: 2026-08-27T14:30:00Z
series_id: 356
data_version: v20260827-1430
sources:
  cavi_score: { value: 4.6, source: "cavi:score:356", fetched_at: "...", cache_ttl: 86400 }
  versions: { source: "cavi:series:356", fetched_at: "...", count: 6 }
  finance_bank: { value: {...}, source: "finance:bank:356:36", fetched_at: "..." }
# ...
```

**用途**：
- 调试："为什么这份报告价格不对？" → 看快照
- 回放：同一快照 + 同一模板 → 同一 MD 哈希
- 审计：3 个月前用户看的报告当时数据是什么？

---

## 七、跨字段一致性

### 7.1 一致性规则

| 规则 | 涉及字段 | 校验 |
|------|---------|------|
| **CAVI 总分唯一** | `cavi_score` 在 frontmatter / Hero / 段 04 出现 3 处 | 三处必须相等 |
| **推荐版本唯一** | `versions[is_recommended=true]` 恰好 1 个 | 多于 1 个 = 警告 |
| **主价卡 = 推荐版本 MSRP** | `finance_cards.main.price` = `versions[recommended].msrp` | 必须相等 |
| **价格非负** | 所有 `price` / `amount` / `total` | `≥ 0` |
| **百分比合理** | `pct` / `loan_ratio` / `recommend` | 0-100 |
| **CAVI 维度评分** | `cavi_dimensions[].value` | 0-5 |
| **4 维度数量** | `cavi_dimensions` | 恰好 4 条 |
| **竞品数量** | `competitors` | 4 条（1 本车 + 3 竞品）|
| **行动卡数量** | `next_cards` | 3 条，各 type 不重复 |
| **spec_strip 数量** | `spec_strip` | 恰好 3 条 |
| **版本数量** | `versions` | 6 条，1 个推荐 |

### 7.2 校验时机

| 时机 | 校验内容 |
|------|---------|
| **数据获取后**（生成前）| 必填字段、字段类型、字段范围 |
| **MD 渲染后**（生成后）| 西语 eyebrow 字面、跨字段一致性、变量替换完整 |
| **人工审核时** | 抽样校验（5% 报告）|

---

## 八、可观测性

### 8.1 每次生成产出

- **MD 文件**：`reports/{lang}/{model}-{year}.md`
- **数据快照**：`reports/_snapshots/{model}-{year}.{timestamp}.yaml`
- **生成日志**：`logs/md-gen/{date}/{model}-{year}.log`

### 8.2 关键指标

| 指标 | 公式 | 目标 |
|------|------|------|
| 生成成功率 | 成功数 / 调用数 | > 99% |
| 字段完整率 | 必填字段非空数 / 必填字段总数 | > 99% |
| 兜底率 | 触发兜底字段数 / 总字段数 | < 5% |
| 跨字段一致率 | 一致字段数 / 总校验字段数 | > 99.5% |
| 单次生成耗时 | end - start | < 60s |
| 数据新鲜度 | 1 - (超 TTL 字段数 / 总字段数) | > 95% |

### 8.3 告警

| 告警 | 触发条件 | 通知 |
|------|---------|------|
| 价格缺失 | `versions[].msrp` 任一缺失 | 立即 · PM + 后端 oncall |
| CAVI 评分缺失 | `cavi_score` 缺失 | 立即 · PM |
| 兜底率超阈值 | 单次生成兜底 > 20% | 1h · PM |
| 跨字段不一致 | 任一一致性规则违反 | 立即 · PM |
| 数据陈旧 | 关键字段超 2× TTL | 6h · PM |
| 生成失败 | 3+ 核心字段失败 | 立即 · PM + oncall |

---

## 九、扩展：新增数据源

新增数据源时，按以下流程：

1. **PM 评审**：评估必要性、影响范围、是否可由现有源派生
2. **API 契约**：填入 §二 的清单（endpoint / TTL / 失败行为）
3. **字段表更新**：同步 `MD-数据-需求清单.md`
4. **缓存 key 注册**：同步 §4.2
5. **告警规则**：同步 §8.3
6. **回归测试**：用 1 个真实车型跑端到端
7. **上线**：灰度 10% → 50% → 100%

---

## 十、相关文档

- *字段权威定义：`docs/MD-数据-需求清单.md`*
- *质量验收：`docs/MD-生成-质量保证.md`*
- *端到端流程：`docs/MD-生成-总览.md`*
- *AI 骨架：`skills/cavi-guide-gen/assets/standard-template-v3.md`*
