# templates/

报告展现样式的 HTML 模板。

## 当前文件

| 文件 | 用途 | 设备 | 大小 |
|------|------|------|------|
| `autocava-pc-v1.html` | **PC 桌面版**（唯一正式模板）| pc | 62KB |

**H5 移动版**在 `reports/autocava-h5-mx-v1.html`（不在本目录）。

## 模板规范

- **展现样式 ID**：`cavi-report-v1`
- **章节数**：8 段（首屏 + 02..08）
- **block-eyebrow**：西语字面锁定（见 `docs/MD-生成-质量保证.md` §4.2）
- **数据来源**：`reports/{lang}/{model}-{year}.md`（同一份 MD 喂给 PC 和 H5）

## 添加新模板

> 当前项目**只有 1 个报告展现样式**。如确需新增（如简版摘要），需：
> 1. PM 评审必要性
> 2. 新建 `autocava-pc-v{2}.html`（v2 编号递增）
> 3. 更新 `docs/MD-生成-展现样式规范.md`
> 4. 在 `SKILL.md` 加新 device 选项（如果需要新的渲染载体）

不要在本目录放：
- ❌ 与已有模板相同的复制品
- ❌ 数据的 demo 样例（放到 `reports/_demo/`）
- ❌ 历史归档版本（放到 `reports/_archive/` 或外部存储）
