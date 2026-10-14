---
name: xb-data-analysis
description: X-Border 数据分析：查亚马逊、沃尔玛、Shopee、TikTok Shop、Temu、1688 的类目大盘、竞品、商品销量与趋势、关键词、货源等市场数据，用来选品、做竞品分析、判断价格带，走 X-Border，每次查询扣 XB Credits。用户要销量、排名、竞品价格、类目或关键词数据、找货源，或用 /xb-data-analysis 时使用。
metadata:
  requires:
    skills: [x-border]
    shell: required
---

# X-Border 数据分析

市场数据来自 X-Border 数据分析，用 X-Border 命令行查（`~/.x-border/bin/x-border`，Windows 上是 `%USERPROFILE%\.x-border\bin\x-border.cmd`；没装或没登录时按 `x-border` 技能装好、登录）。

## 费用

- 列工具、看参数（`tools`）不扣费。
- **每次 `call` 扣 10 XB Credits，参数写错、没查到数据也照样扣**，所以调用前一定先看参数格式。
- 一个问题通常要查 2 到 5 次。预计不超过 200 XB Credits（用户给了预算时按用户的预算）、余额也够，就直接查；超过时先说明要查什么、大约多少，用户同意后再查。查完告诉用户这次一共查了几次、扣了多少。

## 步骤

数据分析的工具也可能直接是 MCP 工具，客户端会给工具名加前缀。先确定能不能直接用（看到搜索结果之前不要执行第 1 步）：有工具搜索时，按平台和要查的东西搜一次（例如搜 `walmart product detail`）；工具列表或搜索结果里有名字以下面第 2 步这类工具名结尾的工具（例如以 `walmart_product_detail_by_product_id` 结尾；有几个都以同一个工具名结尾时选名字最短的那个，例如要亚马逊的 `product_search` 时，以 `temu_product_search` 结尾的是 Temu 的），就不执行第 1 步和命令行：第 2、3 步用工具搜索找工具、看工具的说明和参数，第 4 步直接调用这个工具（扣费一样）。都没有时按下面执行：

1. 执行 `~/.x-border/bin/x-border --version`：执行出错或版本低于 1.5.71 时，按 `x-border` 技能「还没接好 X-Border 时」重新安装。
2. 列出工具：`~/.x-border/bin/x-border tools --source data-analysis`。工具名的前缀是平台：`walmart_` 沃尔玛、`shopee_` Shopee、`tiktok_` TikTok Shop、`temu_` Temu、`ali1688_` 1688、`reddit_` / `quora_` 社区讨论；没有平台前缀的是亚马逊。
3. 看要用的工具的说明和参数：`~/.x-border/bin/x-border tools --source data-analysis <工具名>`。必填参数、站点参数（例如亚马逊的 `amz_site`：US、UK、DE、JP…）都按这里的格式写，不要猜。用户没说站点又必填时，问用户。
4. 调用：`~/.x-border/bin/x-border call --source data-analysis <工具名> '<JSON 参数>'`（参数长、含引号时写进文件，用 `--input <文件>`）。
5. 很多查询要先拿到 ID 再往下查：先按名字查到类目的 `node_id`、商品的 ASIN 或商品 ID，再用它查报告、趋势、评论。每个工具的说明里写了它的结果能给哪些工具用。

## 规则

- **数字只来自查询结果**，不编造、不估算；报数据时写明平台、站点和数据的时间范围。查不到就说查不到。
- **不要用名字里带 `favorite` 的收藏类工具**：它们改的是服务端共用的收藏夹，不属于用户。
- 回复里只说「X-Border 数据分析」，不提数据的上游服务商。
- 用户只是问一般性的选品思路、不需要具体数据时，不用查。

## 出错时

- 提示组织还没开通这项功能：告诉用户请组织管理员开通「数据分析」，不要换别的办法绕过。
- 余额不足：告诉用户需要充值，不要继续查。
- 返回参数错误：按 `tools --source data-analysis <工具名>` 的说明改正后重试一次；同一个错误出现两次就停下来，告诉用户原因。
