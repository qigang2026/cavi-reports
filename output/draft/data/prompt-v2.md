# CAVI 购车解读报告生成 Prompt 模板

## 概述

本 Prompt 用于指导 AI 生成 AutoCava CAVI 购车解读报告（HTML 单文件，PC + H5 自适应）。

---

## 输入数据格式

AI 应接收以下 JSON 格式的结构化数据：

```json
{
  "_schema_version": "1.0.0",
  "_description": "CAVI 购车解读报告数据结构 v1.0",

  "report": {
    "publishDate": "YYYY-MM-DD",
    "expireDays": 7,
    "reportType": "标准报告",
    "hero": {
      "conclusion": "一句话结论",
      "tags": ["标签1", "标签2", "标签3"],
      "description": "车型描述段落",
      "coreStats": {
        "range": { "value": 510, "unit": "km", "standard": "CLTC" },
        "voltage": { "value": 800, "unit": "V" },
        "trunk": { "value": 571, "unit": "L" }
      }
    }
  },

  "vehicle": {
    "brand": "品牌",
    "series": "车系",
    "model": "年款 版本",
    "powertrainType": "EV|ICE|PHEV",
    "bodyType": "车身类型",
    "segment": "细分市场",
    "market": "墨西哥",
    "officialImages": {
      "main": "车型主图URL",
      "hero": "Hero大图URL",
      "scenario": "场景图URL(可选)"
    },
    "links": {
      "series": "车系页URL",
      "lead": "留资页URL",
      "testDrive": "试驾页URL",
      "whatsapp": "WhatsApp链接",
      "service": "服务网点URL"
    }
  },

  "cava": {
    "score": 4.05,
    "maxScore": 5,
    "reviewCount": 301,
    "summary": "CAVA结论",
    "subScores": {
      "safety": 4.6,
      "maintenanceSatisfaction": 4.0
    },
    "strengths": ["优势1", "优势2", "优势3"],
    "concerns": ["关注点1", "关注点2"],
    "dimensions": [
      { "tag": "标签", "title": "标题", "text": "描述文字" }
    ],
    "comparisonCount": 487
  },

  "price": {
    "currency": "MXN",
    "officialAmount": 869900,
    "promotionalAmount": 869900,
    "discountAmount": 60000,
    "priceLevel": "近6个月较低位",
    "historicalHigh": 929900,
    "comparison": {
      "vsHistoricalHigh": { "amount": 60000, "note": "说明" },
      "vsMonthlyAvg": { "amount": 17820, "monthlyAvg": 887820, "note": "说明" }
    }
  },

  "financePlans": [
    {
      "institution": "机构名称",
      "institutionCode": "M",
      "institutionColor": "green|red",
      "productName": "方案名称",
      "productCode": "方案代码",
      "audience": "适合人群",
      "monthlyPayment": 12873,
      "downPaymentRatio": 30,
      "downPaymentAmount": 260970,
      "financingAmount": 608930,
      "termMonths": 72,
      "annualInterestRate": 14.99,
      "description": "方案说明",
      "ctaLink": "WhatsApp链接"
    }
  ],

  "priceTrend": {
    "months": ["2026-03", "2026-04", ...],
    "officialPrices": [899900, 899900, ...],
    "promotionalPrices": [879900, 879900, ...],
    "currentPrice": 869900,
    "source": "数据来源说明"
  },

  "specs": {
    "powertrain": {
      "powerHp": 487,
      "driveType": "AWD",
      "fuelType": "Electric"
    },
    "energy": {
      "rangeKm": 510,
      "rangeStandard": "CLTC",
      "batteryKwh": 80.2,
      "voltagePlatform": "800 V",
      "electricConsumptionKwh100Km": 15.7,
      "chargeTimeNote": "家充约8小时充满"
    },
    "dimensions": {
      "trunkLiters": 571,
      "trunkLitersExpanded": 1374,
      "seats": 5,
      "doors": 5,
      "isoFixCount": 2
    }
  },

  "stats": [
    { "code": "01", "key": "参数名", "value": 510, "unit": "km", "note": "说明" }
  ],

  "ownershipCost": {
    "annualDistanceKm": 12000,
    "energyType": "electricity|gasoline",
    "electricity": {
      "consumptionKwh100Km": 15.7,
      "electricityPriceMxnKwh": 2,
      "annualElectricityCostMxn": 3768,
      "formulaNote": "计算说明"
    },
    "fuel": {
      "referenceVehicle": "同级燃油车型",
      "consumptionL100Km": 8,
      "gasolinePriceMxnLiter": 24,
      "gasolineType": "Regular",
      "annualFuelCostMxn": 22733
    },
    "maintenance": {
      "annualAverageMaintenanceMxn": 3512,
      "monthlyAverage": 293,
      "first5YearTotal": 15080,
      "maintenanceSchedule": [
        { "year": 1, "cost": 2465 }
      ]
    },
    "annualOperatingCostMxn": 7280,
    "monthlyOperatingCostMxn": 607,
    "annualEnergySavingMxn": 18965,
    "soWhat": "结论文案"
  },

  "warranty": [
    { "code": "A", "type": "质保类型", "value": "期限", "note": "备注" }
  ],

  "benefits": [
    { "code": "01", "title": "权益标题", "description": "权益说明", "condition": "条件" }
  ],

  "service": {
    "dealerCount": 6,
    "coveredCities": ["城市1", "城市2"],
    "cityCount": 5,
    "serviceNote": "说明",
    "serviceLink": "URL"
  },

  "scenarios": [
    { "type": "strong|conditional|weak", "label": "标签", "title": "标题", "description": "描述" }
  ],

  "competitors": [
    {
      "position": "本车系|同级竞品",
      "isTarget": true|false,
      "brand": "品牌",
      "model": "车型",
      "priceRange": "价格区间",
      "monthlyPaymentFrom": 13303,
      "image": "图片URL",
      "suitFor": "适合人群",
      "strengths": ["优势"],
      "cautions": ["注意事项"],
      "link": "车系URL"
    }
  ],

  "pitfalls": [
    {
      "icon": "icon-name",
      "title": "坑标题",
      "description": "坑描述",
      "tip": "避坑建议"
    }
  ],

  "pitfallSummary": {
    "title": "总结标题",
    "content": "总结内容"
  },

  "footer": {
    "brand": "品牌名",
    "disclaimer": "免责声明",
    "links": [
      { "text": "链接文字", "url": "URL" }
    ]
  }
}
```

---

## 报告结构规范

### 页面结构（8 大板块）

```
1. Header          - 导航栏（Logo / 分享按钮）
2. Hero            - 车型总览（标签 / 结论 / 核心参数 / 评分 / 车型图）
3. Block 1         - 帮我省钱（车价 / 金融方案 / 用车成本 / 价格走势）
4. Block 2         - 让我安心（CAVA 口碑 / 质保 / 权益 / 服务网点）
5. Block 3         - 助我省时（适用场景 / 竞品对比 / 核心参数）
6. Block 4         - 避坑指南（6 个提醒 + 一句话总结）
7. Footer CTA      - 底部行动点（3 个 CTA 按钮）
8. Floating        - 悬浮组件（Cavi 助手 / 分享弹层 / WhatsApp 按钮）
```

### 设计规范

- **技术栈**: Vanilla HTML + CSS + 极少 JS（Iconify CDN / ECharts）
- **品牌色**: CSS 变量 `--navy` / `--amber`（不要写死 hex）
- **响应式**: `@media (max-width: 768px)` 为 H5 布局
- **字体**: Inter / Noto Sans SC（Google Fonts CDN）
- **图标**: Simple Icons via Iconify CDN

### 数据真实性约束

- ❌ 不得编造价格、销量、评分等数据
- ✅ 可信来源：autocava.com.mx 平台自有数据
- ✅ 数据缺失时标注 `[数据来源：需人工确认]`

---

## 输出要求

1. **单文件 HTML**: PC + H5 自适应，一份 HTML + `@media` 覆盖双尺寸
2. **文件名**: `{品牌}-{车系}-{年款}.html`（如 `xpeng-g6-2026.html`）
3. **发布**: GitHub Pages 静态托管
4. **无后端**: 纯静态产物，不涉及 API 调用

---

## 典型输出示例

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{品牌} {车系} {年款} — 购车解读报告 | AUTOCAVA</title>
  <!-- CSS 变量定义 -->
  <style>
    :root {
      --navy: #0a1929;
      --amber: #d4a41a;
      /* ... */
    }
  </style>
</head>
<body>
  <!-- Header -->
  <header class="site-header">...</header>
  
  <!-- Hero -->
  <section class="hero">...</section>
  
  <!-- Block 1: 帮我省钱 -->
  <section class="section section-light" id="block1">...</section>
  
  <!-- Block 2: 让我安心 -->
  <section class="section" id="block2">...</section>
  
  <!-- Block 3: 助我省时 -->
  <section class="section section-light" id="block3">...</section>
  
  <!-- Block 4: 避坑指南 -->
  <section class="section" id="block4">...</section>
  
  <!-- Footer CTA -->
  <section class="section section-light" id="contacto">...</section>
  
  <!-- Footer -->
  <footer class="site-footer">...</footer>
  
  <!-- Floating -->
  <a class="fab-wa" href="...">...</a>
  <div class="cavi-fab">...</div>
  
  <script>
    // 图表初始化 / 交互逻辑
  </script>
</body>
</html>
```
