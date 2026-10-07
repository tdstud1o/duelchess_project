# 烽决（duelchess）项目记忆

> 本文件由总知识库（cnb.cool/t1mmoc/knowledge）下沉而来。**本仓为公开仓：只放文档/宣发物料/回放 JSON/子模块指针，禁止写入凭据、密钥、内网地址、本机路径。** 完整凭据与平台账号见总知识库。

## 项目概况

- 品牌名：烽决（原「决斗王！」，2026-09-27 重命名）
- 类型：军棋类陆战棋变体（6×13 棋盘，撞雷同归于尽，攻入大本营即胜）
- 五仓：duelchess(前端) / duelchess_rules(规则引擎) / duelchess_server(后端) / duelchess_mapbuilder(地图工具) / duelchess_project(聚合子模块，本仓)；另有 duelchess_skill(仅 CNB)
- 仓库布局：**CNB 为权威源**，main push 经 `git-sync` 单向同步到 GitHub（2026-10-07 起）；GitHub 侧再镜像 GitCode、并跑部署 Actions。两侧不再手工对拷
- 部署：CF Worker `duelchess-server.tmoc.qzz.io` + Pages `duel.tmoc.qzz.io`；版本口径 `v1.0.0-alpha.1`

## 里程碑状态（v1）

| 阶段 | 内容 | 状态 |
|-|-|-|
| M1 | 规则引擎（布局校验/移动吃子/胜负/草丛/炸弹/5 级揭示 + 单测） | ✅ |
| M2 | 前端同屏双人 | ✅ |
| M3 | 在线房间（对弈码匹配、DO 实时对战、四内置图、双端 CI） | ✅ 已部署 |
| M4 | 战绩/回放/变体地图 | 🟡 机制完成，缺前端回放/战绩浏览页 + 变体地图上传联调 |
| M5 | 注册/段位/观战 | ⏳ 未启动 |

## 关键约定与踩坑（可复用）

- **文档版本管理**：docs/v1/ 存里程碑交付物；docs/ 根只留跨里程碑总览。
- **提交纪律**：多会话并行 push 前 fetch+status；绝不用 `git add -A`；未跟踪 .env/tmp 用 `git add -u` 收口。
- **部署链路**：CF 资源（KV SESSIONS/MAP_CACHE、D1、R2 回放）绑定见 wrangler 配置；D1 表由 migrations/0001_init.sql 建；ADMIN_TOKEN 走 wrangler secret put。
- **前端连后端**：生产地址由 VITE_SERVER_BASE 注入（net.ts）；.env.production 入库 / .env.local 本机覆盖且 gitignore。
- **规则引擎**：铁路滑行 railSlide 与传送门 teleport 全套；假基地独立等级 FakeTrap=12；回放 REPLAY_VERSION → RULES_VERSION 兼容窗口严格校验。
- **信息隐藏（暗棋）**：敌方等级全程隐藏，战斗只公布事件不亮等级；**本地暗棋不显示己方等级是故意设计**（faceDownAll 显式配置），勿再当 bug 上报。
- **沙箱出网**：duelchess-server.tmoc.qzz.io 曾被 SNI reset（沙箱内不可达），duel.tmoc.qzz.io 可直连；curl 被代理接管会假 502，测真实网络加 --noproxy "*"。
- **子模块 URL 用相对路径**（`../duelchess_rules.git`）：两端各自解析（CNB→cnb.cool、GitHub→github.com），
  绝对地址会在单向同步时把对侧覆盖掉、对侧 clone --recurse-submodules 全挂。
  ⚠️ 流水线侧的坑另算：公开聚合仓的 push 事件凭证限地本仓，拉不动私有子模块 → 同步/NPC 流水线必须 `git.submodules.enable: false`（配置在流水线，与 .gitmodules 无关）。
- **NPC 触发**：NPC 改代码必须触发在**持有代码的子仓**（会话令牌限地=当前仓库，聚合仓触发拉不到私有子模块）；触发评论必须开工作模式（--work-mode / 「替我上班」）；官方 CodeBuddy 完整召唤格式 `@npc/CodeBuddy(deepseek-v4.1-flash)`。

## 决策记录

- **Turnstile 已移除**（2026-10-01 用户拍板）：/api/session 不再校验；经验=第三方人机验证误报代价 > 防刷收益，谨慎引入。若复用，错误码：110200=域名未授权、600010=数据中心 IP 被 bot 判定、API 端点用 /accounts/{id}/challenges/widgets（/turnstile/widgets 会 7003）。
- **托管决策**（2026-10-03 用户拍板）：测试用 EO Maker（临时链接约 3 小时），生产用 CF；大陆可替代方案（SCF/FC/CFC/FunctionGraph/火山）全部否决（免费限期/需备案/DDoS 账单风险）；EO 不含大陆加速区域已实测否决。SCF WebSocket 单实例单连接，多人房间需外挂 Redis，不做联机主力；CloudBase 免费体验版每 6 个月需手动 0 元续期（上海地域）。
- **可见性**：本仓 public（经全量敏感扫描后转公开）；四个子仓保持 private；公开访客拉不了私有子模块（--recurse-submodules 会失败）。

## 开放事项

- 前端回放/战绩浏览页（M4 缺口）；变体地图上传联调（mapbuilder → POST /api/maps）。
- 线上 E2E 冒烟（建房→对局→认输再战→棋谱落盘）需真人网络验证。
- M5 未启动；DevLog 正式版视频待开发录屏到位；发布节奏等用户指令。

## 宣发现状与口径

- 已上线：四支 B 站视频（15s 高光 BV1wJa66eE57 / 30s 教学 BV1cJa66eEf7 / 60s 综合 BV1cJa66eEfv / DevLog BV1cJa66eEkb）+ 博客《一个人 + 一个 Agent》 https://t1mmoc.github.io/posts/how-i-use-agent-for-marketing/
- 统一口径：v1.0.0-alpha.1 可玩 alpha；「1 个人 + 1 个 Agent 两天半」；硬数据 130 提交 / 9,453 行 / 74 单测 / 173 美术 / 30 音效 / 4 地图；定位「灵感来自军棋，棋盘更像斗兽棋」，禁用「军棋变体」「完成/稳定」。
- 玩法机制已冻结，不再新增。
