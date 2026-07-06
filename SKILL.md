---
name: info-sensitivity-daily-report
description: 轻量级每日信息情报采集。4 层并行采集（政策宏观、热点舆情、行业风险、招聘），10 分钟快速浏览。覆盖金融/房产/外贸/新兴行业/AI科技，输出 HTML 日报。
agent_created: true
---

# 信息敏感度日报 Skill（精简版）

## 概述

将信息敏感度训练框架转化为**轻量级每日情报采集**，4 层并行采集，约 5-8 分钟完成，产出可供 10 分钟快速浏览的 HTML 日报。

**核心价值观**：早（抢在信息传导链末端前）、准（标注来源与可信度）、快（精简到核心，不贪多）、判（给出信号判断）。

---

## 首次使用：偏好设置流程

检测到用户首次使用（无 `.workbuddy/memory/info-daily-report/config.json`）时，先完成偏好设置。每次 1-2 个问题，不要一次问太多。

### 第 1 步：关注领域

```
请选择你最关注的领域（可多选）：

□ 金融财经投资（股市/基金/证券/宏观政策）
□ 房产地产（限购/信贷/土拍/政策）
□ 跨境电商/外贸（关税/海外政策/出海品牌）
□ 新兴小众行业（3D打印/汉服/宠物殡葬/二次元等）
□ 人工智能/科技（大模型/AI应用/融资/监管）
```

### 第 2 步：招聘信息（可选）

```
是否需要每日采集招聘信息？

○ 不需要（默认，日报更精简）
○ 需要（增加 AI/Agent 岗位 + 国企/事业单位招聘）
```

### 第 3 步：自动化

```
要不要设成每日定时自动化？

○ 先跑一次看效果（推荐）
○ 现在就设成每日定时（需告知具体时间）
○ 只要 skill，手动触发
```

### 保存配置

写入 `.workbuddy/memory/info-daily-report/config.json`：

```json
{
  "version": "2.0",
  "domains": ["finance", "real_estate", "cross_border", "emerging", "ai_tech"],
  "include_recruitment": false,
  "recruitment_extra": {},
  "auto_schedule": false,
  "setup_completed": true,
  "setup_date": "YYYY-MM-DD"
}
```

---

## 每日执行流程

### Step 1：读取配置

读取 `.workbuddy/memory/info-daily-report/config.json`。如不存在，重新走偏好设置。

### Step 2：并行采集（4 层，每层 1 个 agent）

用 4 个 agent **并行在后台**执行，每层只做 `WebSearch` + `WebFetch`，不碰登录页、不爬反爬页面。

**① 政策宏观层**
搜索关键词（选 3-4 个组合搜）：
- "2026年7月 政策 经济 央行 财政部 发改委 公告"
- "2026年7月 外贸政策 关税 跨境电商 新规"
- "2026年7月 中美贸易 中欧贸易 地缘"
- 叠加用户关注领域的政策关键词

输出要点：政策条目 + 来源 + 影响判断（机会/风险/观察）

**② 热点舆情层**
搜索关键词：
- "2026年7月 微博热搜 知乎热榜 百度热搜 今日热点"
- "2026年7月 雪球 股市 基金 热议"
- 知乎镜像问题（对立视角）

⚠️ 过滤纯娱乐/体育/明星八卦，保留消费趋势、社会议题、争议话题。

输出要点：TOP 10-15 热点 + 3-5 个镜像视角

**③ 行业风险层**
搜索关键词：
- "2026年7月 AI融资 科技行业 动态"
- "2026年7月 房地产 楼市 政策 房价"
- 用户关注的新兴行业动态
- "2026年7月 反诈 新型诈骗 AI诈骗 金融诈骗"
- "2026年7月 金融直播 割韭菜 风险"

输出要点：行业动态 + 风险案例 + 判断

**④ 招聘信息层**（仅 `include_recruitment: true` 时执行）
搜索关键词：
- "2026年7月 AI岗位 招聘 大厂 校招 薪资"
- "2026年 国企 事业单位 招聘 数学 统计"
- "AI Agent 岗位 技能要求 薪资"

输出要点：大厂动态 + TOP 岗位 + 技能趋势 + 国企/事业单位

### Step 3：编译日报

所有 agent 返回后，编译为 HTML 日报，结构如下（精简为 5 个核心板块）：

```
0. 今日一句话
1. 政策与宏观（含海外信号）
2. 热点与舆情（TOP 10-15 + 镜像视角）
3. 行业与风险（行业动态 + 风险案例合并）
4. AI/Agent 招聘（如开启）
5. 信号汇总表（一表收尾，最多 12 条）
```

**编译规则：**
- ✅ 只展示实际拿到的信息，不占位、不灌水
- ✅ 每条信息标注来源和可信度（A/B/C）
- ✅ 信号判断标注强度（★~★★★★★）和类型（机会/风险/观察）
- ✅ 专业名词首次出现附括号注记
- ✅ 总篇幅控制在 300 行以内，适合 10 分钟浏览
- ✅ 直接输出 HTML（不要 Markdown），文件名 `YYYY-MM-DD-信息敏感度日报.html`

### Step 4：交付

- 从 `config.json` 读取 `output_dir` 字段获取输出目录
- 若配置中无此字段，默认输出到当前工作目录
- 写入 `{output_dir}/YYYY-MM-DD-信息敏感度日报.html`
- 调用 `present_files` 展示
- 不要额外生成 .md 文件

---

## 招聘信息采集细则

### 数据源
1. 人社部招聘月活动 / 新华网招聘报道
2. CSDN/知乎/脉脉 JD 分析文章
3. 国资央企招聘平台 / 国家大学生就业服务平台
4. BOSS直聘/猎聘公开岗位（不登录浏览）

### 核心输出
- 头部大厂动态表（公司/岗位数/地点/薪资/核心技能）
- 热门岗位 TOP 5（岗位/公司类型/薪资/技能）
- 高频技能词统计
- 技能树与学习路径（准入→进阶→推荐顺序）
- 城市薪资参考（样本不足不输出）
- 国企/事业单位专项（如用户需要）

### 周环比趋势
- 每周一将上周技能词频存档至 `.workbuddy/memory/info-daily-report/skill-trend-YYYY-WW.json`
- 本周对比上周，输出趋势表
- 无上周数据时标注「暂无对比数据」

---

## HTML 日报模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>信息敏感度日报 · YYYY-MM-DD</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; max-width: 800px; margin: 0 auto; padding: 24px 20px; line-height: 1.75; color: #2c3e50; background: #fff; font-size: 15px; }
  h1 { font-size: 1.4em; margin-bottom: 4px; }
  h2 { font-size: 1.15em; margin-top: 28px; padding-bottom: 6px; border-bottom: 2px solid #e74c3c; color: #c0392b; }
  h3 { font-size: 1.05em; margin-top: 18px; color: #555; }
  .tagline { color: #999; font-size: 0.85em; margin-bottom: 20px; }
  .oneliner { background: linear-gradient(135deg, #fff5f5, #fef9e7); padding: 14px 18px; border-radius: 8px; margin: 16px 0; font-weight: 600; font-size: 1.05em; border-left: 4px solid #e74c3c; }
  table { border-collapse: collapse; width: 100%; margin: 10px 0 18px; font-size: 0.9em; }
  th, td { border: 1px solid #e8e8e8; padding: 7px 10px; text-align: left; vertical-align: top; }
  th { background: #f5f5f5; font-weight: 600; white-space: nowrap; }
  tr:nth-child(even) td { background: #fafafa; }
  .badge-opp { display: inline-block; background: #e74c3c; color: #fff; padding: 1px 7px; border-radius: 3px; font-size: 0.78em; }
  .badge-risk { display: inline-block; background: #27ae60; color: #fff; padding: 1px 7px; border-radius: 3px; font-size: 0.78em; }
  .badge-watch { display: inline-block; background: #f39c12; color: #fff; padding: 1px 7px; border-radius: 3px; font-size: 0.78em; }
  .stars { color: #e74c3c; }
  .source { color: #999; font-size: 0.82em; }
  .source a { color: #7f8c8d; }
  a { color: #2980b9; text-decoration: none; }
  a:hover { text-decoration: underline; }
  .warn { color: #e67e22; font-size: 0.88em; }
  hr { border: none; border-top: 1px dashed #eee; margin: 20px 0; }
  code { background: #f4f4f4; padding: 1px 5px; border-radius: 3px; font-size: 0.9em; }
  pre { background: #2d3436; color: #dfe6e9; padding: 14px; border-radius: 6px; overflow-x: auto; font-size: 0.85em; line-height: 1.5; }
  .footer { margin-top: 32px; padding-top: 16px; border-top: 1px solid #eee; color: #bbb; font-size: 0.8em; text-align: center; }
  .skill-tree { background: #f8f9fa; padding: 14px 18px; border-radius: 8px; margin: 12px 0; }
  .skill-tree h4 { margin-bottom: 8px; color: #2c3e50; }
  .skill-tree ul { padding-left: 20px; }
  .skill-tree li { margin: 3px 0; }
  @media (max-width: 600px) { body { padding: 12px; font-size: 14px; } table { font-size: 0.8em; } }
</style>
</head>
<body>
<!-- 模板占位，agent 实际填充 -->
<h1>信息敏感度日报 · YYYY-MM-DD</h1>
<p class="tagline">采集方式：公开源并行检索 | 可信度：A=权威 B=行业媒体 C=自媒体 | 强度：★~★★★★★</p>

<div class="oneliner"><!-- 今日一句话 --></div>

<h2>1. 政策与宏观</h2>
<!-- 政策表格：条目 | 可信度 | 影响领域 | 判断 -->
<!-- 海外信号合并在此板块 -->

<h2>2. 热点与舆情</h2>
<!-- TOP 10-15 热点 + 3-5 镜像视角 -->

<h2>3. 行业与风险</h2>
<!-- 行业动态 + 诈骗/风险案例 -->

<h2>4. AI/Agent 招聘（如开启）</h2>
<!-- 大厂动态 + 热门岗位 + 技能趋势 + 学习路径 + 国企招聘 -->

<h2>5. 信号汇总</h2>
<!-- 一表收尾：信号 | 领域 | 类型 | 强度 -->

<div class="footer">信息敏感度日报 · 不构成投资建议 · 诈骗案例仅供风险识别</div>
</body>
</html>
```

---

## 红线与原则

1. **效率优先**：4 层并行，5-8 分钟出结果；拿不到的数据源直接跳过，不空转
2. **不写占位**：只展示实际获取的信息，未取到的板块不留痕迹
3. **注明来源**：每条信息附链接；无源不收录
4. **区分事实与判断**：事实标来源，判断写「agent 推测」
5. **警惕割韭菜**：金融直播/付费内容默认加 ⚠️ 标签
6. **控制篇幅**：300 行以内，10 分钟可读完
7. **直接出 HTML**：不生成 Markdown 中间文件

---

## 调用方式

| 指令 | 说明 |
|------|------|
| `跑今日信息敏感度日报` | 执行当日采集 |
| `跑今日日报，特别关注：{主题}` | 加临时主题 |
| `汇总本周日报，提炼 3 个最强信号` | 周复盘 |
| `开启招聘信息` / `关闭招聘信息` | 切换招聘板块 |
| `哪些城市 AI 岗最多？薪资怎么样？` | 招聘统计 |
