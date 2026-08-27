# MD 生成 · SKILL 调用契约

**文档版本**：v1.2
**创建日期**：2026-08-27
**修订日期**：2026-08-27（v1.2：聚焦 1 个标杆车型 + 4 个真实数据源 + AI 润色不编数）
**目的**：定义 AI Agent（cavi-guide-gen）调用方与被调用方之间的**接口契约**——输入、输出、错误、并发、幂等
**关联**：`skills/cavi-guide-gen/SKILL.md`（AI 内部流程）· `docs/MD-数据-需求清单.md`（字段表）

> 本文档是**外部契约**（调用方关心）；`SKILL.md` 是**内部实现**（AI Agent 怎么调）。两者必须保持一致。

> **当前业务范围**：1 个标杆车型 Versa 2026（series_id=356），MX 西语燃油。详细见 `docs/MD-生成-总览.md` §二。

---

## 一、调用方是谁

```
┌─────────────────┐
│  报告平台/调度系统 │  ← 调用方
│  (AUTOCAVA 后端)  │
└────────┬────────┘
         │ SKILL 调用
         ↓
┌─────────────────┐
│  AI Agent        │  ← 被调用方
│  (cavi-guide-gen) │
└────────┬────────┘
         │ 数据获取（后端做）
         ↓
┌─────────────────┐
│  4 个真实数据源    │  ← CAVI API / CAVI 评分 / 销售 / 金融
└─────────────────┘
```

**调用方典型场景**：
- 用户在车系页点击"获取购车指南"
- 后台调度（每日定时刷新 Versa 报告）
- 数据更新触发（如价格变更）
- 人工强制重生成

---

## 二、输入参数

### 2.1 必填参数

```yaml
series_id: int           # 车系 ID，当前固定 356
```

> **v1.2 简化**：当前业务只支持 1 个车型（Versa 2026），`series_id` 必填但实际只接受 356。

**校验**：
- `series_id` = 356
- 未来扩展其他车型时，加 `market` / `device` 等参数

### 2.2 可选参数

```yaml
force_update: bool       # 强制重生成（忽略缓存），默认 false
priority: enum           # 任务优先级：high / normal / low，默认 normal
callback_url: url        # 异步任务完成回调 URL
metadata: object         # 调用方附加元数据（用于追踪）
```

### 2.3 输入示例

```json
{
  "series_id": 356,
  "force_update": false,
  "priority": "normal",
  "callback_url": "https://autocava.com.mx/api/v1/callbacks/md-gen",
  "metadata": {
    "user_id": "u_12345",
    "trigger": "user_request",
    "request_id": "req_abc123"
  }
}
```

---

## 三、输出

### 3.1 同步模式（priority=normal & 实时返回）

```json
{
  "status": "success",
  "md_file_path": "reports/es/nissan-versa-2026.md",
  "data_version": "v20260827-1430",
  "generated_at": "2026-08-27T14:30:00Z",
  "duration_ms": 4200,
  "validation": {
    "auto_pass": true,
    "fields_complete": "99.5%",
    "consistency_check": "pass",
    "ai_compliance_check": "pass",
    "warnings": []
  },
  "snapshot_path": "reports/_snapshots/nissan-versa-2026.20260827-1430.yaml"
}
```

### 3.2 异步模式（priority=low & 回调）

```json
// 立即返回
{
  "status": "queued",
  "task_id": "task_xyz789",
  "estimated_seconds": 60
}

// 回调时
{
  "task_id": "task_xyz789",
  "status": "success" | "failed",
  "result": { ... 同 §3.1 },
  "error": { ... 见 §四 }
}
```

### 3.3 输出字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `status` | enum | `success` / `failed` / `queued` / `partial` |
| `md_file_path` | string | 生成的 MD 文件相对路径 |
| `data_version` | string | 数据快照版本号（用于回放）|
| `generated_at` | ISO 8601 | 生成时间 |
| `duration_ms` | int | 耗时（毫秒）|
| `validation` | object | 校验结果（含 v1.2 新增 `ai_compliance_check`）|
| `snapshot_path` | string | 数据快照文件路径 |

### 3.4 validation 子结构

```json
{
  "auto_pass": true,
  "fields_complete": "99.5%",
  "consistency_check": "pass",
  "ai_compliance_check": "pass",   // v1.2 新增：AI 未篡改数字
  "warnings": []
}
```

### 3.5 partial 状态

当某些非关键字段缺失时，MD 已生成但带兜底：

```json
{
  "status": "partial",
  "missing_fields": ["annual_cost.saving_tips", "competitors[2].image_url"],
  "fallback_applied": ["saving_tips: 跳过该字段", "competitor image: 占位灰卡"],
  "md_file_path": "..."
}
```

---

## 四、错误码

### 4.1 错误码清单

| Code | HTTP 状态 | 含义 | 调用方处理 |
|------|----------|------|----------|
| `E001_INVALID_INPUT` | 400 | 输入参数缺失或非法 | 修正参数重试 |
| `E002_INVALID_SERIES` | 400 | `series_id` 不在白名单（当前仅 356）| 暂不支持其他车型 |
| `E101_DATA_FETCH_FAILED` | 502 | 数据源获取失败（3 次重试后）| 重试或人工介入 |
| `E102_DATA_INCOMPLETE_CRITICAL` | 422 | 关键字段缺失（价格/版本/CAVI）| 不生成，PM 介入 |
| `E103_DATA_STALE` | 422 | 数据快照超期（关键字段 2× TTL）| 强制刷新 |
| `E201_VALIDATION_FAILED` | 422 | 自动校验不通过 | 查看具体规则失败 |
| `E202_CROSS_FIELD_INCONSISTENT` | 422 | 跨字段不一致 | 修复数据 |
| `E203_AI_TAMPERED_DIGITS` | 422 | **AI 润色时篡改了系统数字**（v1.2 新增）| 重生成 + 报警 |
| `E301_AI_GENERATION_FAILED` | 500 | AI 调用失败 | 重试 |
| `E302_AI_RESPONSE_INVALID` | 500 | AI 返回非法格式 | 重试并记录 |
| `E401_TIMEOUT` | 504 | 单次生成超 60s | 重试或降级 |
| `E500_INTERNAL_ERROR` | 500 | 系统错误 | 告警 + 重试 |

### 4.2 错误响应格式

```json
{
  "status": "failed",
  "error": {
    "code": "E101_DATA_FETCH_FAILED",
    "message": "Failed to fetch cavi_score:356 after 3 retries",
    "details": {
      "source": "cavi-rating-system",
      "last_error": "503 Service Unavailable",
      "retry_count": 3
    },
    "retryable": true,
    "suggested_action": "Wait 60s and retry, or fallback to manual generation"
  }
}
```

### 4.3 重试策略（调用方建议）

| 错误码 | 重试？ | 间隔 |
|--------|-------|------|
| `E101` | 是 | 60s |
| `E201` / `E202` | 否（需修数据）| - |
| `E203` | 否（AI 改数是 P0）| 立即报警 |
| `E301` / `E302` | 是 | 30s |
| `E401` / `E500` | 是 | 60s |
| `E001-E002` | 否（参数错）| - |

---

## 五、并发与幂等

### 5.1 幂等性

**同一输入** + **同一数据快照** → **同一 MD 哈希**。

实现：
- `data_version` 记录数据快照版本
- 同一 `data_version` + `series_id` 不重生成
- `force_update=true` 强制无视幂等

### 5.2 并发控制

**同一 series_id** 同一时刻**只允许 1 个生成任务**：
- 任务进入 → 加锁（key = `lock:md-gen:{series_id}`）
- 锁 TTL：300s
- 重复请求 → 返回当前任务 ID（不入队）
- 任务完成 / 失败 → 释放锁

### 5.3 排队策略

| 任务优先级 | 排队行为 |
|----------|---------|
| `high` | 立即执行，跳过队列 |
| `normal` | FIFO 队列，最多等 60s |
| `low` | FIFO 队列，无超时限制 |

---

## 六、可观测性

### 6.1 调用日志

每次调用记录：

```json
{
  "timestamp": "2026-08-27T14:30:00Z",
  "task_id": "task_xyz789",
  "input": { "series_id": 356 },
  "duration_ms": 4200,
  "status": "success",
  "data_version": "v20260827-1430",
  "validation": { ... },
  "ai_call_count": 12,
  "ai_tokens_used": 8400,
  "cache_hit_rate": "73%"
}
```

### 6.2 关键指标

| 指标 | 目标 |
|------|------|
| P50 延迟 | < 30s |
| P99 延迟 | < 60s |
| 成功率 | > 99% |
| AI 调用次数 | < 15 |
| Token 消耗 | < 10k |
| 缓存命中率 | > 70% |
| **AI 字段合规率** | **100%** |

### 6.3 告警

| 告警 | 触发 | 通知 |
|------|------|------|
| P99 延迟超 60s | 持续 5min | 后端 oncall |
| 成功率 < 95% | 持续 10min | PM + oncall |
| E102 出现 | 单次 | PM（关键数据问题）|
| **E203 出现** | **单次** | **PM + oncall**（AI 篡改数字是 P0）|
| E101 连续 3 次 | 5min 内 | 后端 oncall（数据源故障）|
| 缓存命中率 < 50% | 持续 1h | 后端 oncall |

---

## 七、生命周期

### 7.1 任务状态机

```
       submit
         ↓
   ┌──────────┐
   │  queued  │ ← 排队
   └────┬─────┘
        ↓ pickup
   ┌──────────┐
   │ running  │ ← 执行中
   └────┬─────┘
        ↓
   ┌──────────┐    ┌──────────┐
   │ success  │    │  failed  │
   └──────────┘    └──────────┘
```

### 7.2 状态字段

```json
{
  "task_id": "task_xyz789",
  "state": "running" | "success" | "failed" | "queued",
  "created_at": "2026-08-27T14:30:00Z",
  "started_at": "2026-08-27T14:30:05Z",
  "finished_at": null,
  "progress": {
    "current_step": "fetch_data",       // fetch_data | ai_generation | validation | write
    "total_steps": 4,
    "completed_steps": 1,
    "pct": 25
  }
}
```

### 7.3 状态查询

```
GET /api/v1/md-gen/tasks/{task_id}
```

---

## 八、版本与兼容

### 8.1 API 版本

URL 路径带版本号：

```
/api/v1/md-gen/generate
/api/v1/md-gen/tasks/{task_id}
```

### 8.2 向后兼容策略

- **新增字段**：不影响调用方（旧字段保留）
- **删除字段**：先标记 `deprecated`，保留 3 个月后删除
- **修改语义**：发布 v2，旧 v1 保留 6 个月过渡

### 8.3 变更日志

每次契约变更需更新本文档并标注版本：
- v1.0（2026-08-27）：初版
- v1.1（2026-08-27）：删除 template_id，改为 device
- v1.2（2026-08-27）：聚焦 1 个标杆车型，删除 market/device 等；新增 E203 AI 篡改数字错误码

---

## 九、安全与权限

### 9.1 鉴权

- 调用方需在请求头携带 `Authorization: Bearer {token}`
- token 由 AUTOCAVA 平台颁发
- 校验失败返回 401

### 9.2 速率限制

- 单 token：60 次 / 分钟
- 单 IP：300 次 / 分钟
- 超限返回 429 + `Retry-After`

### 9.3 数据隔离

- 调用方只能访问自己权限内的 series_id
- 跨租户数据严格隔离（多租户模式下）

---

## 十、相关文档

- *总览：`docs/MD-生成-总览.md`*
- *字段表：`docs/MD-数据-需求清单.md`*
- *数据源：`docs/MD-生成-数据源与可靠性.md`*
- *质量保证：`docs/MD-生成-质量保证.md`*
- *展现样式规范：`docs/MD-生成-展现样式规范.md`*
- *AI 内部实现：`skills/cavi-guide-gen/SKILL.md`*
- *AI 骨架：`skills/cavi-guide-gen/assets/standard-template-v3.md`*
