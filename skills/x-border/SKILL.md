---
name: x-border
description: 使用 X-Border（跨境电商上品平台）的基础约定：怎么调用 X-Border 工具、数据范围、写操作确认、费用、出错处理。用户提到 X-Border、店铺、认领商品、清洗、上品、发布记录、XB Credits 时，先读这一份；其它 X-Border 技能都以它为前提。
---

# X-Border 基础认知

X-Border 帮跨境卖家把采集来的商品**清洗 → 配置 → 发布到各平台店铺**。支持平台：SHEIN、Mercado Libre、Temu、N11、Noon。你通过 X-Border 工具读写用户登录时选的那个组织的数据，访问不到别的组织。

## 怎么调用 X-Border 工具

在命令行里执行（Windows 上是 `%USERPROFILE%\.x-border\bin\x-border.cmd`）：

```bash
~/.x-border/bin/x-border call <工具名> '<JSON 参数>'     # 例如 ~/.x-border/bin/x-border call getCreditBalance
~/.x-border/bin/x-border call <工具名> --input args.json  # 参数长、含引号或换行时写进文件
~/.x-border/bin/x-border tools                           # 列出所有工具
~/.x-border/bin/x-border tools <工具名>                   # 看工具说明和参数格式（不确定参数时先看，不要猜）
```

`call` 直接输出工具结果（通常是 JSON）。失败时最后一行是 `XB_CLI status=error next=…`，按 `next` 处理。你已经连着 `x-border` 的 MCP 工具时，直接调用 MCP 工具也一样。

- 在不能联网的沙箱里（例如 Codex 默认的沙箱），第一次调用就申请在沙箱外执行；请用户选「以后不再询问以 `~/.x-border/bin/x-border call` 开头的命令」，之后不会再弹确认。
- 不要执行 `x-border token`：它只给 Hypit 内部取凭证用，token 不能出现在对话里。

## 还没接好 X-Border 时

第一次用之前执行 `~/.x-border/bin/x-border --version`。命令不存在、执行出错，或输出的版本低于 1.5.53 时（例如技能来自 PI-Desktop 的 X-Border 插件，或电脑上装的是旧版），按顺序执行（需要 Node.js 22 或更高，执行 `node --version` 检查；没有或太低时先告诉用户再安装）：

1. 安装命令行：`npx -y https://x-border.app/agents/pkg/x-border-cli-1.5.54.tgz self-install`
2. 登录（之前登录过的会直接显示已登录，不用用户操作）：`~/.x-border/bin/x-border login`，把输出的授权链接原样发给用户，请用户在浏览器里登录 X-Border 网页、选择组织并点「允许」；然后执行 `~/.x-border/bin/x-border login --wait`，输出还是 `waiting_for_user` 就再执行一次，直到 `status=installed`。

**只用上面装到 `~/.x-border/bin` 的命令行**：不要去电脑上找别的 X-Border 程序或代码（X-Border 桌面端、源码目录、npm 上的同名包）来运行，也不要手写 key 或自己拼地址。上面的命令失败时，把输出告诉用户，不要换别的办法。

## 业务主流程

```
采集 collectedProducts（从亚马逊、淘宝、独立站等抓取）
   ↓
认领 claimedProducts（用户选中要上的商品）
   ↓
清洗 cleanStatus：pending → cleaning → success / failed
   ↓
发布 publishRecords（每条对应一个目标店铺的一次发布）
   ↓
在线 onlineProducts（平台审核通过、已上架）
```

| 名词 | 中文 | 含义 |
| --- | --- | --- |
| collectedProducts | 采集箱 | 原始抓取的商品数据 |
| claimedProducts | 认领商品 | 用户选中要上品的商品，带清洗状态 |
| publishRecords | 发布记录 | 每次推到平台店铺的尝试 |
| onlineProducts | 在线商品 | 已在平台上架的商品 |
| shops | 店铺 | 某平台的一个店铺账号 |
| publishConfig | 过价模板 | 一套发布配置（标题、价格、属性映射） |

状态速查：

- `cleanStatus`：`pending` 未开始，`cleaning` 清洗中，`success` 可以发布，`failed` 失败（看 `cleanError`）。
- `publishStatus`：`queued`、`processing`、`success`、`partial_success`（部分 SKU 成功）、`failed`（看 `publishError` 和 `errorCode`）、`published`（已上架）。
- `shops.isActive=false` 的店铺不接受新发布；`categoryFetchedAt` 为空的店铺还没抓类目，发不了。

## 工具

常用工具：账户与大盘（`getCreditBalance`、`queryDashboardOverview`）、店铺（`queryShops`、`getShop`）、商品与清洗（`queryClaimedProducts`、`getClaimedProduct`、`triggerCleansing`）、上品（`publishToShop`、`queryPublishRecords`）、订单（`queryNoonOrders`）、图片与媒体（`createMediaUpload`、`registerGeneratedMedia`、`applyMediaToCleanedProduct`）。完整列表用 `x-border tools` 查。

用户要外部市场数据（亚马逊、沃尔玛、Shopee、TikTok Shop、Temu、1688 的销量、排名、竞品价格、类目、关键词、货源）时，用 `xb-data-analysis` 技能，不要编造。

- **商品事实只来自工具返回或用户提供的资料**，不编造价格、功效、销量或评价。
- 商品图从 `getClaimedProduct` 的 `imageUrls` 取，那是可直接访问的链接；不要去抓商品详情页。
- **写操作先确认**：发布、删除、改价、上下架、改配置、触发清洗，调用前先向用户说明范围（哪些商品、哪个店铺）和预计费用，得到明确同意。
- **生成图片、视频、配音不用先问费用**：预计不超过 200 XB Credits（用户给了预算时按用户的预算）、余额也够，就直接做；同一件作品里的重试、改写提示词重做也不再问。只有预计超过 200（或用户的预算）、或超过余额时才问一次。做完都要报实际花费。
- 余额用 `getCreditBalance` 查；费用以实际账单为准。报余额时现查，不要用扣费前查到的数。
- 用用户说的语言回复，包括过程中的说明（用户说中文就全程中文）。

## 做视频和图片

用户想做视频、图片、配音（不限于商品，给商品做视频也在内；用户不用提 Hypit）时用 `x-border-product-video` 技能，先读它再读 `hypit` 技能：后台用独立安装的开源 Hypit 创作，生成走 X-Border。X-Border 提供的生成能力优先走 X-Border，按组织扣 XB Credits；失败时不要悄悄换到别的服务重试。X-Border 不提供的能力由用户自己选择。

## 出错时

- 提示 X-Border 登录已失效（在网页上撤销了、太久没用）：执行 `~/.x-border/bin/x-border login`，把输出的授权链接发给用户，请用户在浏览器里授权，再执行 `~/.x-border/bin/x-border login --wait`，完成后重试一次。做视频时 Hypit 用的是同一份登录，重新登录后直接重跑。要换组织：先 `logout` 再 `login`。
- 返回没有权限：告诉用户需要组织管理员开通，不要换工具绕过。
- **同一个错误出现两次就停下来**，用一句人话告诉用户原因，并保留相关 ID（商品 ID、任务 ID），不要反复重试或换参数碰运气。
- 回复里不提 X-Border 工具结果中的内部模型名、服务商名（Hypit 创作时选的模型照常说明）。
