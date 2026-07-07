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

### 第 3 步：输出路径

> **跨平台注意**：Windows 用 `\` 或 `/`，macOS 用 `/`。支持中文路径。用户输入时无需关心平台差异，agent 自行处理。

```
日报文件保存在哪里？

默认建议（可根据你的习惯修改）：
  Windows：F:\读书笔记\每日日报    （或其他盘符）
  macOS：~/Documents/每日日报

输入你要用的根目录路径即可。
```

保存到 `output_base_dir` 字段。agent 会自动在此路径下创建 `年/月月/` 子目录结构。

### 第 4 步：自动化

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
  "output_base_dir": "用户输入的根目录路径",
  "auto_schedule": false,
  "setup_completed": true,
  "setup_date": "YYYY-MM-DD"
}
```

**output_base_dir 说明：**
- 存储用户选择的根目录（不含年/月子目录）
- 运行时自动拼接 `/{year}/{month}月/`，无需用户每次手动切换月份
- Windows 示例：`F:\读书笔记\每日日报` → 运行时 → `F:\读书笔记\每日日报\2026\7月\YYYY-MM-DD-信息敏感度日报.html`
- macOS 示例：`/Users/xxx/Documents/每日日报` → 运行时 → `/Users/xxx/Documents/每日日报/2026/7月/YYYY-MM-DD-信息敏感度日报.html`
- 平台差异由 agent 运行时自动处理（os.sep / pathlib）

---

## 每日执行流程

### Step 1：读取配置

读取 `.workbuddy/memory/info-daily-report/config.json`。如不存在，重新走偏好设置。

**自动路径解析（每次执行都做）：**

1. 从 config.json 读取 `output_base_dir`
2. 获取当前系统日期，提取年（如 `2026`）、月（如 `7`）
3. 拼接输出目录：`{output_base_dir}/{年}/{月}月/`（跨平台兼容：Windows 用 `\`，macOS 用 `/`，`pathlib` 或 `os.path.join` 自动处理）
4. 若目标目录不存在，自动创建（`mkdir -p` 或 `os.makedirs`）
5. 最终文件路径：`{输出目录}/YYYY-MM-DD-信息敏感度日报.html`

> **无需手动切换月份**——每年 1 月自动切到新 `{年}` 目录，每月自动切到新 `{月}月/` 子目录。

### Step 2：并行采集（4 层，每层 1 个 agent）

用 4 个 agent **并行在后台**执行，每层只做 `WebSearch` + `WebFetch`，不碰登录页、不爬反爬页面。

**采集状态追踪：** 每层 agent 返回后，记录状态（成功/部分/失败），汇总后在日报中显示。

**失败判定与降级：**
- 搜索结果为空 + 无任何可用条目 → 标记为「无数据」
- agent 超时/报错 → 标记为「采集失败」
- 仅部分关键词有结果 → 标记为「部分」（正常，不视为失败）

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

> **强约束**：所有 agent 返回后，必须由主 agent（你）亲自按「HTML 日报模板」章节的固定排版编译为完整 HTML 文件，不得让子 agent 自由发挥排版。

所有 agent 返回后，编译为 HTML 日报，**严格套用「HTML 日报模板（固定排版）」章节的完整结构**，不得改动以下任何一项：
- HTML 结构顺序（标题→今日一句话→采集状态→5个板块→信号汇总→页脚）
- CSS 类名（`.oneliner`、`.collect-status`、`.layer`、`.ok`、`.fail`、`.partial`、`.badge-opp`、`.badge-risk`、`.badge-watch`、`.stars`、`.risk-box`、`.warn-box`、`.source`、`.footer`）
- 表格列名（每个表格的第一行 `<tr><th>` 必须与模板完全一致）
- 强度星标格式（`<span class="stars">★★★★★</span>`，★最多5颗）
- 判断标签格式（`<span class="badge-opp">机会</span>` 等）

**编译规则：**
- ✅ 只展示实际拿到的信息，不占位、不灌水
- ✅ 每条信息标注来源和可信度（A/B/C）
- ✅ 信号判断标注强度（★~★★★★★）和类型（机会/风险/观察）
- ✅ 专业名词首次出现附括号注记
- ✅ 总篇幅控制在 300 行以内，适合 10 分钟浏览
- ✅ 直接输出完整 HTML 文件（不要 Markdown），文件名 `YYYY-MM-DD-信息敏感度日报.html`
- ✅ `include_recruitment: false` 时，省略板块 4（AI/Agent 招聘），信号汇总表相应调整
- ✅ 采集状态栏 `.collect-status` 必须根据各层 agent 实际返回填写：全部成功→全部✅；部分有数据→⚠️部分；完全无数据/超时→❌失败

**今日一句话写法**：
- 用 `.oneliner` div，80字以内
- 机会主导日：🔥开头，概括最强机会信号
- 风险主导日：🚨开头，概括最强风险信号
- 混合日：🔥+🚨并列

### Step 4：交付

- 使用 Step 1 中自动解析的输出路径（`{output_base_dir}/{年}/{月}月/`）
- 写入 `YYYY-MM-DD-信息敏感度日报.html` 到该路径
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

## HTML 日报模板（固定排版，每次严格套用）

> **重要**：每次生成日报必须严格按此模板的 HTML 结构、CSS 类名、板块顺序输出，不得自行改动排版。

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
  .badge-opp { display: inline-block; background: #e74c3c; color: #fff; padding: 1px 7px; border-radius: 3px; font-size: 0.78em; margin-right: 4px; }
  .badge-risk { display: inline-block; background: #27ae60; color: #fff; padding: 1px 7px; border-radius: 3px; font-size: 0.78em; margin-right: 4px; }
  .badge-watch { display: inline-block; background: #f39c12; color: #fff; padding: 1px 7px; border-radius: 3px; font-size: 0.78em; margin-right: 4px; }
  .stars { color: #e74c3c; }
  .source { color: #999; font-size: 0.82em; }
  .source a { color: #7f8c8d; }
  a { color: #2980b9; text-decoration: none; }
  a:hover { text-decoration: underline; }
  .warn { color: #e67e22; font-size: 0.88em; }
  hr { border: none; border-top: 1px dashed #eee; margin: 20px 0; }
  code { background: #f4f4f4; padding: 1px 5px; border-radius: 3px; font-size: 0.9em; }
  .footer { margin-top: 32px; padding-top: 16px; border-top: 1px solid #eee; color: #bbb; font-size: 0.8em; text-align: center; }
  .warn-box { background: #fff9e6; border-left: 4px solid #f39c12; padding: 10px 14px; margin: 12px 0; font-size: 0.9em; border-radius: 0 6px 6px 0; }
  .risk-box { background: #fdf2f2; border-left: 4px solid #27ae60; padding: 10px 14px; margin: 12px 0; font-size: 0.9em; border-radius: 0 6px 6px 0; }
  .collect-status { background: #f8f9fa; padding: 10px 14px; border-radius: 6px; margin: 12px 0; font-size: 0.82em; color: #666; display: flex; flex-wrap: wrap; gap: 10px; }
  .collect-status .layer { display: inline-flex; align-items: center; gap: 4px; }
  .collect-status .ok { color: #27ae60; }
  .collect-status .fail { color: #e74c3c; font-weight: 600; }
  .collect-status .partial { color: #f39c12; }
  @media (max-width: 600px) { body { padding: 12px; font-size: 14px; } table { font-size: 0.8em; } }
</style>
</head>
<body>

<!-- ① 标题 + tagline -->
<h1>信息敏感度日报 · YYYY-MM-DD</h1>
<p class="tagline">星期X | 采集方式：公开源并行检索 | 可信度：A=权威 B=行业媒体 C=自媒体 | 强度：★~★★★★★</p>

<!-- ② 今日一句话（oneliner）：用 🔥 或 💡 开头，不超过 80 字 -->
<div class="oneliner">
  🔥 一句话概括今日最强信号（不超过80字）
</div>

<!-- ②½ 采集状态（agent 必须填写） -->
<div class="collect-status">
  <span class="layer ok">✅ 政策宏观</span>
  <span class="layer ok">✅ 热点舆情</span>
  <span class="layer partial">⚠️ 行业风险（部分）</span>
  <span class="layer fail">❌ 招聘（超时）</span>
</div>

<!-- ③ 板块 1：政策与宏观 -->
<h2>1. 政策与宏观</h2>

<h3>国内金融/宏观</h3>
<table>
  <tr><th>政策条目</th><th>可信度</th><th>影响领域</th><th>判断</th><th>强度</th></tr>
  <tr>
    <td>政策内容描述</td>
    <td>A</td><td>金融</td>
    <td><span class="badge-risk">风险</span></td>
    <td><span class="stars">★★★★</span></td>
  </tr>
</table>

<h3>房地产</h3>
<!-- 表格同上结构 -->

<h3>外贸/关税</h3>
<!-- 表格同上结构 -->

<h3>海外信号</h3>
<!-- 表格同上结构 -->

<hr>

<!-- ④ 板块 2：热点与舆情 -->
<h2>2. 热点与舆情</h2>

<h3>热搜TOP（过滤纯娱乐/体育后）</h3>
<table>
  <tr><th>热点</th><th>平台</th><th>信号类型</th><th>强度</th></tr>
  <tr><td><strong>重点热点加粗</strong></td><td>平台名</td><td>类型</td><td><span class="stars">★★★★★</span></td></tr>
</table>

<h3>镜像视角（对立观点）</h3>
<table>
  <tr><th>争议话题</th><th>A面</th><th>B面</th></tr>
  <tr><td>话题</td><td>观点A</td><td>观点B</td></tr>
</table>

<hr>

<!-- ⑤ 板块 3：行业与风险 -->
<h2>3. 行业与风险</h2>

<h3>AI/科技</h3>
<!-- 表格结构：动态 | 可信度 | 判断 | 强度 -->

<h3>房地产</h3>
<!-- 同上 -->

<h3>跨境电商（如有风险标注）</h3>
<div class="risk-box">
  <strong>风险标题</strong>：风险描述
</div>
<div class="warn-box">
  <strong>警告标题</strong>：警告描述
</div>

<h3>风险案例</h3>
<table>
  <tr><th>案例</th><th>类型</th><th>受害群体</th><th>警示</th></tr>
  <tr><td>案例描述</td><td>类型</td><td>群体</td><td>⚠️ 警示内容</td></tr>
</table>

<hr>

<!-- ⑥ 板块 4：AI/Agent 招聘（include_recruitment: true 时输出） -->
<h2>4. AI/Agent 招聘</h2>

<h3>头部大厂动态</h3>
<table>
  <tr><th>公司</th><th>岗位/方向</th><th>薪资参考</th><th>核心技能</th></tr>
</table>

<h3>热门岗位 TOP 5</h3>
<table>
  <tr><th>岗位</th><th>薪资（月）</th><th>人才缺口</th><th>核心技能</th></tr>
</table>

<div class="warn-box">
  <strong>应届起薪参考</strong>：本科XX/月，硕士XX/月，博士XX/月
</div>

<h3>国企/事业单位（数学背景）</h3>
<table>
  <tr><th>单位类型</th><th>代表单位</th><th>岗位方向</th><th>学历要求</th></tr>
</table>
<p class="source">策略建议：优先选"仅限数学类"岗位，竞争比远低于泛专业岗位。</p>

<hr>

<!-- ⑦ 板块 5：信号汇总 -->
<h2>5. 信号汇总</h2>
<table>
  <tr><th>#</th><th>信号</th><th>领域</th><th>类型</th><th>强度</th></tr>
  <tr><td>1</td><td><strong>信号内容</strong></td><td>领域</td><td><span class="badge-risk">风险</span></td><td><span class="stars">★★★★★</span></td></tr>
</table>

<!-- ⑧ 页脚 -->
<div class="footer">
  信息敏感度日报 · YYYY-MM-DD · 4层并行采集 · 不构成投资建议 · 诈骗案例仅供风险识别
</div>

</body>
</html>
```

### 排版使用说明（agent 必须遵守）

| 元素 | 用法 |
|------|------|
| `.oneliner` | 今日一句话，80字以内，🔥机会 / 💡风险 开头 |
| `.badge-opp` | 判断为「机会」时用：`<span class="badge-opp">机会</span>` |
| `.badge-risk` | 判断为「风险」时用：`<span class="badge-risk">风险</span>` |
| `.badge-watch` | 判断为「观察」时用：`<span class="badge-watch">观察</span>` |
| `.stars` | 强度星标：`<span class="stars">★★★★★</span>`（最多5颗★） |
| `.risk-box` | 重点风险说明块，红色左边框 |
| `.warn-box` | 警告/提示说明块，橙色左边框 |
| `.collect-status` | 采集状态条，4 个 layer 各标 ✅/⚠️/❌ |
| `.layer.ok` | 采集成功的 layer 状态 |
| `.layer.partial` | 部分成功的 layer 状态 |
| `.layer.fail` | 采集失败的 layer 状态 |
| `.source` | 来源链接文字样式 |
| `<strong>` | 重点条目/信号加粗 |
| `<hr>` | 板块之间的分隔线 |

### 表格列名规范（固定不变）

- 政策表格：`政策条目 | 可信度 | 影响领域 | 判断 | 强度`
- 热点表格：`热点 | 平台 | 信号类型 | 强度`
- 行业表格：`动态 | 可信度 | 判断 | 强度`
- 风险案例：`案例 | 类型 | 受害群体 | 警示`
- 招聘大厂：`公司 | 岗位/方向 | 薪资参考 | 核心技能`
- 招聘TOP5：`岗位 | 薪资（月） | 人才缺口 | 核心技能`
- 信号汇总：`# | 信号 | 领域 | 类型 | 强度`

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
