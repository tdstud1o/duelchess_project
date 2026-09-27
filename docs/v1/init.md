# 烽决(DuelChess) 项目文档 v1

> 本文件为 `duelchess_project` 聚合仓的项目级文档。规则细节以 `duelchess/docs/PRD.md` 为准。

## 1. 产品

双人在线对抗棋（Stratego/军棋类）。双方在布局区布置 18 个棋子 / 设施，通过吃子、吃陷阱、吃基地取胜。特色地形：河道 / 草丛 / 陷阱 / 大本营；变体扩展铁道与传送门。

## 2. 仓库结构（submodule 聚合）

| 子仓 | 角色 | 部署 | 技术栈 | 状态 |
|---|---|---|---|---|
| `duelchess` | 前端 SPA | CF Pages | Preact + TS + Vite | M2 同屏双人 ✅ |
| `duelchess_server` | 后端 API / 实时对战 | Worker + DO + D1 + R2 + KV | Workers TS | M3 完成 ✅ |
| `duelchess_rules` | 规则引擎（前后端共享） | npm 包 | TS + vitest | M1 完成 ✅ |
| `duelchess_mapbuilder` | 地图编辑器 + 地图数据 | 静态页 | 单文件 HTML | 编辑器进行中 |

## 3. 架构

```
CF Pages（duelchess）──── Preact SPA：主菜单 / 布局编辑 / 对局界面
        │ REST + WebSocket
CF Worker（duelchess_server）
   ├─ Durable Objects ── 对局房间：实时状态 + WS(Hibernation) + 权威仲裁
   ├─ D1 ── 用户 / 战绩 / 棋谱索引 / 地图元数据
   ├─ KV ── 地图 JSON 缓存 / 对弈码→房间映射 / 会话 token
   └─ R2 ── 棋谱全文 / 地图缩略图 / 皮肤等静态资源
```

- **规则引擎**（`@duelchess/rules`）被前端与服务端同时引用：前端做即时反馈 / 走子预览 / 棋谱回放，服务端做权威仲裁（防作弊）。
- **对局模型**：房间内双方各自完成 18 位布局 → 进入对局 → 服务端逐手结算并广播。
- **胜负**：吃掉对方基地即胜。计时当前为**无限时间**（M5 后再议）。

## 4. 克隆与本地开发

```bash
git clone --recurse-submodules git@github.com:t1mmoc/duelchess_project.git
cd duelchess_project
# 前端引用 rules 源码（vite alias ../duelchess_rules）：
cd duelchess && npm install && npm run dev
```

## 5. 里程碑

| 阶段 | 内容 | 状态 |
|---|---|---|
| M1 | rules 包：布局校验 / 走子 / 吃子 / 胜负 + 单测 | ✅ |
| M2 | 前端同屏双人 + 布局编辑器，Pages 上线 | ✅ |
| M3 | 在线房间：对弈码匹配、DO 实时对战 | ✅ |
| M4 | 战绩 / 棋谱回放、变体地图（铁道 / 传送门） | ⏳ |
| M5 | 注册体系、积分段位、观战 | ⏳ |

## 6. 规则要点（已定稿）

- 走子：水平 2 格 / 任意斜向 1 格；水平相邻敌子可水平 1 格直接攻击。
- 进出 / 内部经过陷阱区 · 大本营：仅水平 1 格。
- 草丛：孤立草丛有敌驻守须水平 1 格攻击进入，无人驻守视为平地；**边相连草丛整片平地化**。
- 炸弹：撞任何普通子同亡；吃真陷阱自爆（双方消失）；吃空陷阱 / 基地正常结算。
- 5 级阵亡：暴露其所属方基地位置。
- 假基地：大本营格上放置的普通棋子，不可移动、被吃不判负。
