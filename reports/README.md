# reports/

CAVI 购车报告产出。本目录即 `qigang2026/cavi-reports` GitHub Pages 仓库。

## 目录结构

```
reports/
├── es/                       # 西语产出
│   ├── nissan-versa-2026.html         ← 西语主版（最新）
│   ├── nissan-versa-2026-interpretacion.html  ← 西语"解读"深度评测版
│   └── nissan-versa-2026.md           ← 西语 Markdown 源
├── zh/                       # 中文产出
│   ├── nissan-versa-2026.html         ← 中文主版（最新）
│   ├── nissan-versa-2026-v2.html      ← 历史版本
│   ├── nissan-versa-2026-v3.html
│   └── nissan-versa-2026.md           ← 中文 Markdown 源
└── templates/                # 通用模板（不参与发布）
    ├── autocava-pc-v1.html
    ├── autocava-pc-mx-v1.html
    └── autocava-pc-v1-cn-demo.html
```

## 当前最新版本

| 语言 | 文件 | 线上 URL |
|---|---|---|
| ES | `es/nissan-versa-2026.html` | https://qigang2026.github.io/cavi-reports/es/nissan-versa-2026.html |
| ZH | `zh/nissan-versa-2026.html` | https://qigang2026.github.io/cavi-reports/zh/nissan-versa-2026.html |

## 旧 URL 兼容

仓库根保留了 5 个 meta-refresh 重定向文件以兼容历史链接：

- `356-nissan-versa-2026.html` → `es/nissan-versa-2026.html`
- `356-nissan-versa-2026-zh.html` → `zh/nissan-versa-2026.html`
- `356-nissan-versa-2026-zh-v2.html` → `zh/nissan-versa-2026-v2.html`
- `356-nissan-versa-2026-zh-v3.html` → `zh/nissan-versa-2026-v3.html`
- `356-nissan-versa-2026-interpretacion.html` → `es/nissan-versa-2026-interpretacion.html`

## 命名规范

- **目录** = 语言（`es/` / `zh/`）
- **文件名** = `{车型}-{年款}.{ext}`，不再带 `356-` 前缀和 `-es`/`-zh` 后缀
- **历史版本** 用 `-v2`/`-v3` 后缀保留在同语言目录下
