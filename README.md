# 流枢 / FlowHub

一个常驻 macOS 菜单栏的运维工作台，解决五件每天都要做的事：

| 热键 | 面板 | 解决什么 |
|---|---|---|
| `⌘⇧T` | 任务 | 随手记下今天要做的事，按医院分组（群组可自己增删改）；没做完的第二天继续挂着，做完的第二天自动收走，复制清单带上医院名 |
| `⌘⇧V` | 剪切板 | 文本 / 图片 / 文件的历史记录、搜索、置顶，回车直接粘回原应用 |
| `⌘⇧K` | k8s 工作台 | 启动或点一下自动跑取票命令拿 token，并打开配置好的多个站点（URL 带参 / 剪切板 / 网页注入） |
| `⌘⇧H` | 医院系统 | 地址、账号、密码统一管理，别名/拼音秒搜，一键打开并自动登录，可联动 DBeaver 连接 |
| `⌘⇧J` | 文本存档 | 常用 SQL / 命令 / 笔记按 Markdown 存档，代码与语句一眼分得清；按群组归类（同「任务」），点一条即复制原文 |

原生 Swift / SwiftUI 实现，**零第三方依赖**、离线可用；密码与 token 只进 macOS 钥匙串，配置文件里没有任何明文密码。

## 快速开始

```bash
./scripts/build_app.sh --install     # 编译并安装到 /Applications/流枢.app
open /Applications/流枢.app          # 启动；菜单栏与 Dock 都会出现图标
```

首次启动后：

1. 面板顶部「去授权」→ 勾选辅助功能（否则回粘/自动输入降级为复制）。
2. Chrome → 查看 → 开发者 → 勾选「允许 Apple 事件中的 JavaScript」（网页自动填充需要）。
3. 设置 → DBeaver 连接：填好工作区目录后，点「导入为医院档案」可把现有连接批量变成医院条目（只读，不写 DBeaver 任何文件）。

自检：

```bash
FLOWHUB_HOME=/tmp/flowhub-check "/Applications/流枢.app/Contents/MacOS/流枢" --self-test
```

## 文档

| 文档 | 内容 |
|---|---|
| [docs/01-需求梳理与场景.md](docs/01-需求梳理与场景.md) | 需求拆成功能点、用户故事、验收标准 |
| [docs/02-架构设计.md](docs/02-架构设计.md) | 技术选型、模块划分、数据流、关键决策 |
| [docs/03-数据模型与存储.md](docs/03-数据模型与存储.md) | config.json 结构、SQLite 表、钥匙串条目、群组与 Markdown 约定 |
| [docs/04-工作流设计.md](docs/04-工作流设计.md) | 五条工作流的完整流程 |
| [docs/05-安全与权限.md](docs/05-安全与权限.md) | 威胁模型、权限清单、降级策略 |
| [docs/06-开发路线图与验收.md](docs/06-开发路线图与验收.md) | 里程碑、任务拆解、验收用例、风险 |
| [docs/07-使用与配置手册.md](docs/07-使用与配置手册.md) | 安装、配置、常见问题 |
| [examples/config.sample.json](examples/config.sample.json) | 完整样例配置（含两个 k8s 工作台示例） |

## 代码结构

```
Sources/FlowHub/
├── App/        菜单栏、弹窗、全局热键、状态装配、自检
├── Core/       配置中心、钥匙串、SQLite 封装、命令执行、AppleScript/键盘事件、拼音搜索、通用群组逻辑
├── Clipboard/  需求 1：采集、去重、落盘、回粘
├── Ops/        需求 2：取票解析、站点打开与 token 注入
├── Hospital/   需求 3：医院档案、三级自动登录
├── Texts/      需求 4：文本存档（Markdown 正文、代码高亮、群组归类）、DBeaver 连接读取
├── Tasks/      需求 5：任务清单（未完成跨天累积、当天完成当天显示、按医院分组）
└── UI/         SwiftUI 面板
```

```
./scripts/build_app.sh            # swiftc 直接编译（默认，最快）
./scripts/build_app.sh --spm      # 用 SwiftPM 编译
swift build                       # 也可直接用 SPM
```

## 已知边界

- 医院系统若有验证码 / 双因子，自动填充只做到提交前，验证码仍需人工输入。
- 网页注入依赖「允许 Apple 事件中的 JavaScript」，被策略禁用时自动降级为键盘模拟。
- ad-hoc 签名下，每次重新编译后 macOS 可能要求重新授权辅助功能。
- 文本存档只做「存 + 看 + 一键复制」：不连数据库、不执行语句、不回写任何外部文件；DBeaver 连接仅只读读取用于医院档案导入。
- v1 为单机单人使用，不做云同步与团队共享（见路线图 v2 候选）。
