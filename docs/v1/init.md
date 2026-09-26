# 决斗王！(DuelChess) 项目总览 v1

> 本仓（duelchess_project）为聚合仓：以 git submodule 管理全部子仓 + 项目级文档。
> 规则权威来源：`duelchess/docs/PRD.md` v0.1（2026-09-26 定稿）。

## 子仓结构

| 子仓 | 角色 | 部署 | 技术栈 |
|---|---|---|---|
| `duelchess` | 前端 SPA | CF Pages | Preact + TS + Vite |
| `duelchess_server` | 后端 API / 实时对战 | CF Worker + DO + D1 + R2 + KV | Workers TS |
| `duelchess_rules` | 规则引擎（前后端共享） | npm 包（两仓引用） | TS + vitest |
| `duelchess_mapbuilder` | 地图编辑器 + 地图数据 | 静态页 | 单文件 HTML |

## 架构

```
CF Pages（duelchess）──── Preact SPA：棋盘渲染 / 布局编辑 / 房间对战 UI
        │ REST + WebSocket
CF Worker（duelchess_server）
   ├─ Durable Objects ── 对局房间：实时状态 + WS(Hibernation) + 权威仲裁
   ├─ D1 ── 用户 / 战绩 / 棋谱索引 / 地图元数据
   ├─ KV ── 地图 JSON 缓存 / 对弈码→房间映射 / 会话
   └─ R2 ── 棋谱全文 / 地图缩略图 / 静态资源
```

## 对局模型

- 房间内双方各自完成 18 位布局（布局校验由 rules 包执行）→ 进入对局 → 服务端逐手仲裁并广播。
- 胜负：吃掉对方基地即胜。
- 计时：**无限时间**（暂不设计时器/超时）。

## 克隆本仓

```bash
git clone --recurse-submodules git@github.com:t1mmoc/duelchess_project.git
```

## 里程碑（摘要，详见各仓）

| 阶段 | 内容 |
|---|---|
| M1 | rules 包：布局校验 / 走子 / 吃子 / 胜负 / 草丛·炸弹·5级暴露 + 单测 |
| M2 | 前端同屏双人 + 布局编辑器，Pages 上线 |
| M3 | 在线房间：对弈码匹配、DO 实时对战 |
| M4 | 战绩 / 棋谱回放、变体地图（数值由 mapbuilder 编辑，含假基地占位） |
| M5 | 注册体系、积分段位、观战 |
