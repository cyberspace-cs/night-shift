# 🕵️ ShiftX（夜班侦探 Night Shift）学习指南

> **你睡着以后，他才开始工作。**
> *When You Sleep, He Works.*

ShiftX 是 **AdventureX 2026** 黑客松获奖项目，本仓库（`night-shift`）即其完整开源实现。
本文档从**产品创意 → 玩法设计 → 代码架构 → 技术栈**四个层面拆解这个项目，方便快速上手与学习。

---

## 🏆 项目名片

| 项目 | 说明 |
| --- | --- |
| 项目名 | **夜班侦探 Night Shift（ShiftX）** |
| 获奖情况 | 🥇 AdventureX 2026 · Amazon Quick × First Prize；🏆 Injective Blockchain × AI Innovation Award |
| 在线预览 | 🌐 **https://www.shiftx.top/** |
| 一句话定位 | 与你轮班生活的**异步侦探游戏**：白天整理线索、晚上睡觉，侦探林渡替你去城市调查 |
| 核心情绪 | **我想早点睡，因为我想知道侦探今晚会发现什么** |
| 开源协议 | Open Source |

---

## 💡 核心产品创意

### 解决的问题

人已经疲惫，却仍不愿结束屏幕活动。传统睡眠应用用评分、连续签到、完成率提醒用户"做得还不够好"，把休息变成另一项绩效任务。

### 产品回答：用悬念代替说教

- 😴 不说"你应该睡觉"，而是让玩家**想知道侦探今晚会发现什么**
- 🌙 不评价睡眠好坏，任何时长都能推进核心案件
- 🔒 **Local-first**：无账号、无数据库、无 API Key、无可穿戴设备也能完整游玩
- 🤖 **受约束 AI**：AI 只负责玩家主动授权的晨间短笺语气，绝不改变案件真相
- 🎁 收藏是时间留下的叙事痕迹，不是分数、货币或排行榜

### 核心玩法循环

```text
阅读晨报 → 整理线索与推论 → 选择调查方向 + 一件随身物
    ↑                                            ↓
获得晨报/线索/明信片/夜印/植物/回声 ← 城市夜间行动 ← 玩家离开屏幕休息
```

| 阶段 | 玩家做什么 | 系统做什么 |
| --- | --- | --- |
| 白天 | 读证词、整理证物、合成推论 | 案件板可检索，推论须玩家亲自确认归档 |
| 睡前交接 | 选调查方向 + 随身物 | 方向决定去处与遭遇，随身物改变细节观察 |
| 夜间 | 离开屏幕去休息 | 城市继续行动（真实时间，Demo 可压缩到 12 秒） |
| 清晨 | 拆晨报，收获新线索 | 寄回证物、明信片、夜印、植物、睡隙回声 |

### 差异化卖点

1. **休息不是空白，而是游戏输入** —— 离开屏幕的时间就是叙事发生的时间
2. **睡得短不会失败** —— 时长只影响观察丰富度，不锁线索、不枯植物、不锁结局
3. **确定性悬疑 + 受约束 AI** —— 案件真相全部确定，AI 只润色晨间短笺语气
4. **一套运行时承载多套案件** —— 通过 `CampaignManifest` 接入，五案共享同一生命周期

---

## 🎮 游戏内容

当前案件书架收录 **5 套完整五夜案件**：

| 案件 | 简介 |
| --- | --- |
| 《零点四十三分的末班车》 | 一张来自昨天的七年旧车票，与一条官方否认存在的 43 号线 |
| 《只在雨中播出的电台》 | 只有下雨时才出现的公共频率，与四十七户被城市遗忘的居民 |
| 《面包奇谈》 | 十二人合作社、一个开放访客份额与一场被改写责任的火灾 |
| 《千早诺亚的现身》 | 十三次抵达、十二段被现实排除的人生，以及谁有权裁定真相 |
| 《雾中无狼》 | 失舌银铃、成对蜡筒、缺失的一分钟与一场无人承认的夜晚 |

每案包含：三幕序章 · 五夜循环 · 12 条核心线索 · 8 件收藏品 · 确定性推论链 · 三种结局 · 独立本地存档。

其他能力：真实夜班恢复、多案件存档隔离、中英文首案、好友线索二维码、虚拟睡眠硬件样机、Home Assistant 空间外设桥、Injective EVM Testnet 收藏回执。

---

## 🏗️ 代码架构

### 运行形态（多端复用同一套产品）

```text
┌──────────────────────────────────────────────────────────────┐
│                        产品层（共享）                          │
│  案件书架 → 序章 → 游戏页 → 推理 → 收藏（同一 UI + 本地存档）   │
└──────────────────────────┬───────────────────────────────────┘
                           │
      ┌────────────────────┼─────────────────────┐
      ▼                    ▼                     ▼
┌──────────┐     ┌─────────────────┐     ┌──────────────────┐
│  Vercel  │     │  Cloudflare     │     │  Capacitor 8     │
│ next build│     │  Sites/Worker   │     │  Android / iOS   │
│  (.next/)│     │  (dist/ + wrangler)   │  原生壳（WebView）  │
└──────────┘     └─────────────────┘     └──────────────────┘
```

- **Web 双目标**：默认 `next build` 生成 Vercel 部署；`build:sites` 通过 Vinext/Vite 生成 Cloudflare Sites/Worker 产物
- **移动端**：Capacitor 8 提供 Android/iOS 原生壳，安全 WebView 加载生产地址
- **桌宠**：Electron（`apps/desk-pet`）接收环境读数
- **硬件**：`apps/rdk-sentry`（地瓜 RDK X5 睡眠哨兵）+ `apps/connector`（Home Assistant 空间桥）
- **区块链**：Hardhat + Solidity 合约，Injective EVM Testnet 收藏回执（`contracts/`、`contract-tests/`）

### 前端状态与内容架构

| 模块 | 位置 | 职责 |
| --- | --- | --- |
| 状态机 | `src/stores/game-store.ts` | `day → ready → night → morning → ending` 全流程独占状态 |
| 案件包契约 | `src/content/campaigns/types.ts` | `CampaignManifest`、引用完整性校验与案件查询 |
| 案件注册表 | `src/content/campaigns/registry.ts` | 五案白名单、默认案件与合法 `campaignId` |
| 案件内容包 | `src/content/campaigns/*.ts` | 每案五夜章节、线索、收藏与结局（独立文件） |
| 视觉资源包 | `src/content/*-assets.ts` | 各案专属横幅、夜印、明信片、植物、人物与城区资产 |
| 本地化核心 | `src/i18n/*` | Cookie/请求语言协商、案件能力回退、递归内容投影 |
| 推理合成台 | `src/components/game/` | 证物档案库、推论合成、密文台、案件板 |

### 目录结构速览

```text
night-shift/
├── src/              # 产品前端（Next.js App Router + React 19 + TS）
│   ├── components/   # UI 组件（游戏/收藏/海报/音乐/钱包）
│   ├── content/      # 案件内容包（CampaignManifest + 资产）
│   ├── i18n/         # 中英文本地化
│   ├── stores/       # Zustand 状态机（游戏/收藏/硬件/密文）
│   └── lib/          # 工具库
├── app/              # Next 路由（night-shift 游戏 / api / keepsake / posters）
├── apps/             # 桌面/硬件应用：desk-pet 桌宠、rdk-sentry、connector
├── contracts/        # Solidity 收藏合约（Hardhat + Injective EVM）
├── docs/             # 产品/美术/架构/剧本圣经等完整文档
├── plans/            # 开发计划与决策记录
├── android/ ios/     # Capacitor 原生壳
├── worker/           # Cloudflare Worker
└── db/ drizzle/      # 数据库（未启用 D1 示例）
```

---

## 🛠️ 技术栈

| 技术 | 用途 |
| --- | --- |
| Next.js 16 + React 19 + TypeScript 5.9 | 产品核心 |
| Tailwind CSS 4 + Zustand 5 | 样式与状态 |
| Drizzle ORM | 数据访问（可选） |
| Solidity + Hardhat 3 + Injective EVM | 区块链收藏回执 |
| Cloudflare Workers / OpenNext | Serverless 部署 |
| Electron | 桌宠（硬件联动） |
| Capacitor 8 | Android / iOS 原生壳 |
| Playwright + Vitest | E2E 与单元测试 |

---

## 📚 学习路线建议

按以下顺序阅读，理解速度最快：

1. **先玩**：[在线预览](https://www.shiftx.top/) 跑一遍首案，感受"休息即游戏输入"的循环
2. **读产品**：[docs/product-overview.md](docs/product-overview.md) → [docs/north-star.md](docs/north-star.md)
3. **读架构**：[docs/architecture.md](docs/architecture.md) → [docs/index.md](docs/index.md)
4. **读代码**：`src/stores/game-store.ts`（状态机）→ `src/content/campaigns/types.ts`（案件契约）→ 任一案件内容包
5. **看故事**：各案 `story-bible` 了解确定性叙事设计
6. **读决策**：`docs/decision-log.md` + `plans/` 了解工程演进过程

---

## 💬 一句话总结

> ShiftX 不是另一张睡眠报表，而是一个**让你真心期待放下手机的理由**——
> local-first、确定性悬疑、受约束 AI、文学性城市收藏，把"休息"从绩效重新变成故事。
