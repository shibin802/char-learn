# 认字小助手 - 项目交接文档

> 最后更新：2026-03-28 by 小宝

---

## 一、项目概览

| 项目 | 内容 |
|------|------|
| 项目名称 | char-learn |
| GitHub 仓库 | https://github.com/shibin802/char-learn |
| 线上地址 | https://char.shibin.space |
| Cloudflare Pages 项目 | `char-learn` |
| 自定义域名 | `char.shibin.space` → CNAME → `charlearn.pages.dev` |
| 目标用户 | 一年级小学生 + 家长 |

---

## 二、技术栈

- **纯前端**：单文件 HTML，无框架、无构建
- **字库模块**：`chars.js`（ES Module import）
- **部署方式**：Cloudflare Pages（wrangler CLI）
- **数据存储**：浏览器 localStorage（无后端、无需注册）
- **本地存储 key**：`cl_v5`

---

## 三、文件结构

```
char-learn/
├── index.html    ← 主程序（所有逻辑）
└── chars.js      ← 字库数据
```

---

## 四、核心功能

### 4.1 两种练习模式

| 模式 | 说明 |
|------|------|
| `pin` 字→拼音 | 显示汉字，手写或选择写出拼音 |
| `chr` 拼音→字 | 显示拼音+词组提示，书写汉字 |

> `chr` 模式展示逻辑（2026-03-27 改版）：
> - 只显示蓝色拼音框 + 同组词组中另一个字
> - 目标字可能在词组左侧或右侧
> - 不直接展示目标字本身

### 4.2 两种答题方式

| 方式 | 说明 |
|------|------|
| 手写输入 | 调用系统自带输入法（iOS/Android 原生手写） |
| 选择答题 | 4 选 1，从同组字中随机生成干扰项 |

### 4.3 SM-2 间隔重复算法

每个字独立追踪以下字段（存在 localStorage）：

```json
{
  "0-0": {
    "ef": 2.5,   // ease factor（难度因子）
    "iv": 0,     // interval（下次复习间隔天数）
    "rp": 0,     // reps（连续答对次数）
    "nr": 0,     // next review（下次复习时间戳，毫秒）
    "lr": null   // last result（'ok' | 'fail'）
  }
}
```

**掌握标准**：`rp >= 3` 且 `iv >= 7`

**升级规则（答对）**：
- 第 1 次答对：iv = 1天
- 第 2 次答对：iv = 3天
- 第 3 次+：iv = iv * ef（可进一步 * 1.3）

**降级规则（答错）**：
- iv 重置为 1天
- ef 下调 0.2（最低 1.3）

### 4.4 家长面板

- 各组进度条（掌握数/总字数）
- 薄弱字列表（黄色标签）
- 总体掌握率
- 进度重置（清除所有 localStorage）

---

## 五、字库结构（chars.js）

### 分组概览

| 组号 | 名称 | 约字数 |
|------|------|--------|
| 0 | 自然·姓氏·动作 | ~54 |
| 1 | 爱国·家乡 | ~50 |
| 2 | 校园·游戏·朋友 | ~50 |
| 3 | 日常·节日·生活 | ~50 |

### phraseMap（词组提示）

- 316 个常用词组，彬哥亲自整理
- 格式：`{ "字": "词组" }`，如 `{ "伙": "伙伴", "玩": "玩耍" }`
- 全部为 2 字词，常见、地道、无姓氏/专有名词

### GROUPS 数组结构

```js
export const GROUPS = [
  {
    name: "自然·姓氏·动作",
    chars: [
      { char: "霜", pinyin: "shuāng", tone: 1 },
      { char: "吹", pinyin: "chuī", tone: 1 },
      // ...
    ]
  },
  // ...
];
```

---

## 六、最近开发记录

### 2026-03-28（最新）

**commit**: `33d14b8` — feat: add choice mode selection on home screen

**新增功能**：
- 首页点击模式卡片 → 弹出「选择答题方式」页面（☑️选择答题 / ✍️手写输入）
- 按组练习同样先进入答题方式选择页
- 选择题逻辑：4 选 1，从同组字中随机抽 3 个干扰项，正确答案位置随机
- 答对：绿色 ✓，600ms 后自动下一题；答错：红色 ✗ + 正确答案高亮，900ms 后自动下一题

**状态**：代码已推送到 GitHub，等待部署。

### 2026-03-27

- chr 模式彻底改版：只展示拼音蓝色框 + 词组另一个字，不直接展示答案
- phraseMap 替换为彬哥提供的 316 个词组
- 手写输入功能实现（方案：调用系统自带输入法）

### 更早

- v1.0 正式发布：双向练习 + SM-2 + 家长面板

---

## 七、部署指南

### 方式一：wrangler CLI（推荐）

```bash
cd /root/.openclaw/workspace/projects/char-learn

# 登录 Cloudflare（如未登录）
wrangler login

# 部署
wrangler pages deploy . --project-name=char-learn
```

### 方式二：GitHub Actions CI（待接入）

推送到 main 分支后自动触发部署，需配置：

```bash
# 环境变量
CLOUDFLARE_API_TOKEN=xxxx   # Cloudflare API Token
```

> ⚠️ Cloudflare Token 目前未配置，请联系彬哥或小马配置。

---

## 八、Cloudflare 配置

| 项目 | 值 |
|------|-----|
| Pages 项目名 | char-learn |
| 部署地址 | charlearn.pages.dev |
| 自定义域名 | char.shibin.space |
| DNS CNAME 目标 | charlearn.pages.dev |
| DNS 记录 ID（上次） | `4f12a06ab9a4f9df2897096396a4c932` |

---

## 九、给接手同学的提示

1. **代码风格**：index.html 是单文件，HTML/CSS/JS 全在一起，约 300 行，方便小改动
2. **调试**：直接在浏览器打开 index.html（ES Module 需要起本地服务器或用 Safari/部分浏览器直接打开）
3. **localStorage 清理**：浏览器开发者工具 → Application → Local Storage → 清除
4. **拼音匹配**：选择题用 `st()` 函数（去除声调后匹配），支持有声调/无声调混合输入
5. **词组逻辑**：phraseMap 查找目标字在词组中位置，决定左边还是右边显示

---

## 十、Git 提交记录（最近）

| commit | 描述 |
|--------|------|
| `33d14b8` | feat: add choice mode selection on home screen |
| `c74affd` | feat: 家长面板问号提示，文字布局优化 |
| `35a13d0` | feat: 家长面板加问号说明 |
| `c8a0fc2` | feat: 全面改版 - 纯手写输入模式，家长面板显示掌握明细 |
| `0fb4526` | feat: 认字小助手 v1 - 双向练习+间隔重复+家长面板 |
