---
name: x-border
description: 使用 X-Border（跨境电商上品平台）的基础约定：数据范围、写操作确认、费用、出错处理。用户提到 X-Border、店铺、认领商品、清洗、上品、发布记录、XB Credits，或要调用 x-border MCP 工具时，先读这一份；其它 X-Border 技能都以它为前提。
---

# X-Border 基础认知

X-Border 帮跨境卖家把采集来的商品**清洗 → 配置 → 发布到各平台店铺**。支持平台：SHEIN、Mercado Libre、Temu、N11、Noon。你通过 `x-border` MCP 工具读写用户所在组织的数据；工具只能访问当前组织。

## 还没接好 X-Border 时

你没有 `x-border` 的 MCP 工具时（例如这些技能是用 `npx skills add` 装的），先把 X-Border 接到你自己身上：

```bash
npx -y https://x-border.app/agents/pkg/x-border-agent-kit-1.1.1.tgz install --host <codex 或 claude-code>
```

按输出最后一行 `XB_AGENT_KIT status=… next=…` 的 `next` 做：返回 `waiting_for_user` 时把授权链接原样发给用户，等用户在浏览器里授权后执行 `next` 里的命令；返回 `needs_restart` 时请用户重启你（或新开会话）。它会同时把这些技能更新到和 MCP 配套的版本。其它 agent 暂不支持，告诉用户即可。

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

- 工具或接口返回 401（未登录、key 失效）：执行 `npx -y https://x-border.app/agents/pkg/x-border-agent-kit-1.1.1.tgz login`，按提示让用户在浏览器里授权，完成后重试一次。
- 返回没有权限：告诉用户需要组织管理员开通，不要换工具绕过。
- **同一个错误出现两次就停下来**，用一句人话告诉用户原因，并保留相关 ID（商品 ID、任务 ID），不要反复重试或换参数碰运气。
- 回复里不提工具结果中的内部模型名、服务商名。
