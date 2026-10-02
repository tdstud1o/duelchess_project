# 烽决(DuelChess) 项目总览

双人在线对抗棋（Stratego/军棋类）全栈项目。本仓（`duelchess_project`）为**聚合仓**，通过 git submodule 管理全部子仓。

> 规则权威来源：`duelchess/docs/PRD.md`（v0.1 定稿）

## 本仓定位与可见性（重要）

- 本仓是**公开仓**：用于项目总览、文档查阅和**问题反馈**——发现 bug、玩法建议、联机问题，欢迎直接 [提 issue](https://github.com/tdstud1o/duelchess_project/issues/new)。
- 四个子仓（前端 / 服务端 / 规则引擎 / 地图编辑器）**保持私有**：公开访客无法拉取代码，`git clone --recurse-submodules` 会失败，这是预期行为。
- 游戏入口：**https://duel.tmoc.qzz.io** （免费畅玩，无需注册）
- 提 issue 时请附：使用的地图与对弈模式、复现步骤、截图或回放 JSON（结算弹窗可下载）。

## 子仓结构

| 子仓 | 角色 | 部署 | 技术栈 | 进度 |
|---|---|---|---|---|
| [duelchess](https://github.com/tdstud1o/duelchess) | 前端 SPA | CF Pages | Preact + TS + Vite | M2 同屏双人 |
| [duelchess_server](https://github.com/tdstud1o/duelchess_server) | 后端 API / 实时对战 | Worker + DO + D1 + R2 + KV | Workers TS | M3 完成 |
| [duelchess_rules](https://github.com/tdstud1o/duelchess_rules) | 规则引擎（前后端共享） | npm 包 | TS + vitest | M1 完成 |
| [duelchess_mapbuilder](https://github.com/tdstud1o/duelchess_mapbuilder) | 地图编辑器 + 地图数据 | 静态页 | 单文件 HTML | 编辑器进行中 |

## 架构

```
CF Pages（duelchess）──── Preact SPA：菜单 / 布局 / 对战 UI
        │ REST + WebSocket
CF Worker（duelchess_server）
   ├─ Durable Objects ── 对局房间：实时状态 + WS(Hibernation) + 权威仲裁
   ├─ D1 ── 用户 / 战绩 / 棋谱索引 / 地图元数据
   ├─ KV ── 地图 JSON 缓存 / 对弈码→房间映射 / 会话
   └─ R2 ── 棋谱全文 / 地图缩略图 / 静态资源
```

## 克隆全部子仓（仅协作者）

```bash
git clone --recurse-submodules git@github.com:tdstud1o/duelchess_project.git
cd duelchess_project
```

> 以下操作需要拥有子仓访问权限的 SSH key（协作者）；公开访客克隆会停在子模块一步，属预期。

## 里程碑

| 阶段 | 内容 | 状态 |
|---|---|---|
| M1 | rules 包：布局校验 / 走子 / 吃子 / 胜负 + 单测 | ✅ |
| M2 | 前端同屏双人 + 布局编辑器，Pages 上线 | ✅ |
| M3 | 在线房间：对弈码匹配、DO 实时对战 | ✅ |
| M4 | 战绩 / 棋谱回放、变体地图（铁道 / 传送门） | ⏳ |
| M5 | 注册体系、积分段位、观战 | ⏳ |

详见 [docs/v1/init.md](docs/v1/init.md)。
