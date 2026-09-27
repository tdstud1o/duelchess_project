# 烽决(DuelChess) 项目总览

双人在线对抗棋（Stratego/军棋类）全栈项目。本仓（`duelchess_project`）为**聚合仓**，通过 git submodule 管理全部子仓。

> 规则权威来源：`duelchess/docs/PRD.md`（v0.1 定稿）

## 子仓结构

| 子仓 | 角色 | 部署 | 技术栈 | 进度 |
|---|---|---|---|---|
| [duelchess](https://github.com/t1mmoc/duelchess) | 前端 SPA | CF Pages | Preact + TS + Vite | M2 同屏双人 |
| [duelchess_server](https://github.com/t1mmoc/duelchess_server) | 后端 API / 实时对战 | Worker + DO + D1 + R2 + KV | Workers TS | M3 完成 |
| [duelchess_rules](https://github.com/t1mmoc/duelchess_rules) | 规则引擎（前后端共享） | npm 包 | TS + vitest | M1 完成 |
| [duelchess_mapbuilder](https://github.com/t1mmoc/duelchess_mapbuilder) | 地图编辑器 + 地图数据 | 静态页 | 单文件 HTML | 编辑器进行中 |

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

## 克隆全部子仓

```bash
git clone --recurse-submodules git@github.com:t1mmoc/duelchess_project.git
cd duelchess_project
```

## 里程碑

| 阶段 | 内容 | 状态 |
|---|---|---|
| M1 | rules 包：布局校验 / 走子 / 吃子 / 胜负 + 单测 | ✅ |
| M2 | 前端同屏双人 + 布局编辑器，Pages 上线 | ✅ |
| M3 | 在线房间：对弈码匹配、DO 实时对战 | ✅ |
| M4 | 战绩 / 棋谱回放、变体地图（铁道 / 传送门） | ⏳ |
| M5 | 注册体系、积分段位、观战 | ⏳ |

详见 [docs/v1/init.md](docs/v1/init.md)。
