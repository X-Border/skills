---
name: x-border
description: 使用 X-Border（跨境电商上品平台）的基础约定：数据范围、写操作确认、费用、出错处理。用户提到 X-Border、店铺、认领商品、清洗、上品、发布记录、XB Credits，或要调用 x-border MCP 工具时，先读这一份；其它 X-Border 技能都以它为前提。
---

# X-Border 基础认知

X-Border 帮跨境卖家把采集来的商品**清洗 → 配置 → 发布到各平台店铺**。支持平台：SHEIN、Mercado Libre、Temu、N11、Noon。你通过 `x-border` MCP 工具读写用户所在组织的数据；工具只能访问当前组织。

## 还没接好 X-Border 时

你没有 `x-border` 的 MCP 工具时（例如只装了这些技能），按安装说明 https://x-border.app/agents/install.md 的第 3、4 步添加 MCP 连接并登录，然后请用户重启你。pi（xb-pi）自带 X-Border 的 MCP：请用户在 pi 里输入 `/login`，选 X-Border 登录。不要手写 key，也不要自己拼 MCP 地址。

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

工具按用途分组：账户与大盘（`getCreditBalance`、`queryDashboardOverview`）、店铺（`queryShops`、`getShop`）、商品与清洗（`queryClaimedProducts`、`getClaimedProduct`、`triggerCleansing`）、上品（`publishToShop`、`queryPublishRecords`）、订单（`queryNoonOrders`）、图片与媒体（`createMediaUpload`、`registerGeneratedMedia`、`applyMediaToCleanedProduct`）。不确定用哪个时先看工具说明，不要猜参数。

- **商品事实只来自工具返回或用户提供的资料**，不编造价格、功效、销量或评价。
- 商品图从 `getClaimedProduct` 的 `imageUrls` 取，那是可直接访问的链接；不要去抓商品详情页。
- **写操作先确认**：发布、删除、改价、上下架、改配置、触发清洗，以及任何扣 XB Credits 的操作，调用前先向用户说明范围（哪些商品、哪个店铺）和预计费用，得到明确同意。
- 余额用 `getCreditBalance` 查；费用以实际账单为准。

## 做视频

给商品做视频用 `x-border-product-video` 技能（配合独立安装的 Hypit 和 `hypit` 技能）。X-Border 提供的生成能力（Seedance 视频、nano-banana-pro 生图）优先走 X-Border，按组织扣 XB Credits；失败时不要悄悄换到别的服务重试。X-Border 不提供的能力由用户自己选择。

## 出错时

- MCP 工具返回 401（登录过期或被撤销）：重新登录 MCP 连接——Codex 执行 `codex mcp login x-border`，Hermes 执行 `hermes mcp login x-border`，Claude Code 请用户输入 `/mcp` 重新认证，pi 请用户输入 `/login` 重新登录；把授权链接发给用户，完成后重试一次。
- 做视频时 X-Border 媒体接口返回 401：执行 `npx -y https://x-border.app/agents/pkg/x-border-agent-kit-1.2.3.tgz login`，按提示让用户在浏览器里授权，完成后重试一次。
- 返回没有权限：告诉用户需要组织管理员开通，不要换工具绕过。
- **同一个错误出现两次就停下来**，用一句人话告诉用户原因，并保留相关 ID（商品 ID、任务 ID），不要反复重试或换参数碰运气。
- 回复里不提工具结果中的内部模型名、服务商名。
