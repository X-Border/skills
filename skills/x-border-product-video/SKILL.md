---
name: x-border-product-video
description: 给 X-Border 里的商品做视频：从 x-border MCP 取商品信息和商品图，用 Hypit 在本机创作（生成走 X-Border，扣 XB Credits），成片存回 X-Border 并可挂到商品上。用户说"给这个商品做个视频""做主图视频""做个带货短视频"时使用。
metadata:
  requires:
    skills: [hypit, x-border]
    shell: required
---

# 给 X-Border 商品做视频

这个技能负责 X-Border 这一侧：选商品、取素材、把 Hypit 接到 X-Border、存回成片。**视频怎么创作按 `hypit` 技能来做**，本技能不重复它的内容。

**视频只用 Hypit 做。** X-Border MCP 不提供视频创作工具；Hypit 不可用时告诉用户，不要找别的办法代替。

## 开始前检查

1. 你必须能执行 shell 命令。不能的话，直接告诉用户：这个功能需要能执行命令的 agent（如 Codex、Claude Code、pi）。
2. 需要 `hypit` 技能。没有的话自己安装（先用一句话告诉用户），不用新开会话：

   ```bash
   npx -y skills add hypit-ai/hypit -g -y -a <codex 或 claude-code>
   ```

   装好后读取 `~/.agents/skills/hypit/SKILL.md`（Claude Code 是 `~/.claude/skills/hypit/SKILL.md`），按它创作。
3. 建好视频项目目录，把 Hypit 接到 X-Border：

   ```bash
   npx -y https://x-border.app/agents/pkg/x-border-agent-kit-1.1.1.tgz hypit setup --project <项目目录>
   ```

   没有 Hypit 时它会装进项目目录（不要全局安装：npm 全局目录常没有写权限）。之后所有 Hypit 命令都在项目目录里用 `npx hypit …` 执行。重复执行没有副作用。不要自己改 Hypit Profile 里的 X-Border 配置，也不要读取或输出 key。命令最后一行是 `XB_AGENT_KIT status=… next=…`，失败时按 `next` 处理。

## 流程

### 1. 确定商品

- 用户没指定时，用 `queryClaimedProducts` 按用户的描述（标题关键词、店铺、状态）查，列出候选让用户选。
- 用 `getClaimedProduct` 取详情：`title`、`imageUrls`、`cleanData` 里的属性和卖点。
- **视频里的商品事实只来自这里或用户提供的资料**，不编造功效、材质、价格、销量。资料不够时问用户。

### 2. 确定成片要求

先问清楚（用户已经说了的不再问）：

- **用途**。要挂到 Temu 商品上作为主图视频时，比例只能是 1:1、3:4 或 16:9（**不能是 9:16**），文件不超过 20MB。其它用途按用户要求。
- 时长、画幅、有没有配音或字幕。
- 预算上限。

### 3. 创作

- 每个商品单独一个项目目录，例如 `xb-videos/<商品简称>-<日期>/`。命令先 `cd` 进项目目录再用相对路径，不要反复手写很长的绝对路径。
- 把 `imageUrls` 里的商品图作为参考图交给 Hypit。出现商品的画面必须用真实商品图做参考，不用生成的图代替。
- 画面里不要加入会让买家误以为包含在商品里的东西。例如标题写明 Without Container，就不要把商品放进花瓶展示；确实需要道具时先告诉用户。
- 按 `hypit` 技能写 Brief、规划和生成。当前通过 X-Border 可用的模型：视频 Seedance 2.0 / 2.5，图片 nano-banana-pro。
- **只有一个生成镜头、不做后期时，不要搭合成**（时间线、Film、渲染都不需要），直接把生成的镜头作为输出。项目里放两个文件，按商品改 `prompt`、图片路径和参数即可：

  `production.svml`：

  ```xml
  <?svml using="@hypit/markup@1"?>
  <svml>
    <import as="media" from="@hypit/media@1"/>
    <import as="text" from="@hypit/text@1"/>
    <import as="seedance" from="@hypit/seedance@1"/>

    <media:Image id="product" src="./assets/product.jpg"/>
    <text:Value id="prompt">画面描述（英文效果更稳定）</text:Value>
    <seedance:ReferenceVideo id="shot" model="standard" prompt={prompt} duration="5" resolution="480p" aspect-ratio="1:1" generate-audio="false">
      <seedance:Reference image={product} person-reference="false"/>
    </seedance:ReferenceVideo>
  </svml>
  ```

  `build.svrun`：

  ```xml
  <?svml using="@hypit/run-markup@1"?>
  <svrun version="1">
    <author source="./production.svml"/>
    <target output="shot.video"/>
  </svrun>
  ```

  商品图先下载到 `assets/`。`model` 是 `standard`（Seedance 2.0）或 `2.5`；参考图里有人物时 `person-reference="true"`。然后执行 `npx hypit plan ./build.svrun`（不用加 `--runtime`，项目已经选好 X-Border）→ `npx hypit build ./build.svrun --follow` → `npx hypit get <build-id> --output shot.video --to ./final.mp4`。
- 需要合成（多镜头拼接、字幕、图形）时，本地渲染要先下载浏览器，文件较大、首次要几分钟：先告诉用户，再执行 `npx hypit programs prepare --endpoint hyperframes.local`。只有一个生成镜头、不做后期时不需要。
- **付费生成前**，先用 `npx hypit plan` 确认要生成什么，并检查每个生成请求都由 `xborder.default` 处理；向用户说明镜头数、时长、分辨率和预计 XB Credits，得到明确同意再提交。`npx hypit pricing` 目前只给出价格页链接。
- **实际花费这样查**：Build 结束后执行 `npx hypit logs <build-id> --lines 1000`（默认只显示最后 50 条），每个 X-Border 生成请求都有一行 `X-Border 回执 <id>`；把这些 id 一起传给 MCP 的 `getMediaCharges`，`totalCredits` 就是这些生成实际扣的 XB Credits（失败的已退回）。**不要用前后余额的差值**：它还包含你这段对话使用模型的费用。
- `npx hypit plan` 报告某个请求不支持（比例、时长、分辨率、参考方式）时，按支持的范围调整方案并告诉用户，不要悄悄把 X-Border 能做的生成换到别的服务。
- X-Border 目前只提供上面三个模型。配音、字幕对齐、渲染等 X-Border 不提供的能力，按 `hypit` 技能用本地方案，或由用户自己选择服务；用了会另外收费的服务时，先告诉用户。

### 4. 存回 X-Border

成片完成后：

1. 调用 `createMediaUpload`（`mediaType: "video"`，`contentType: "video/mp4"`，`fileName` 填本地成片路径），执行返回的 `command` 上传，响应里的 `url` 就是 X-Border 媒体链接。上传链接 15 分钟有效，过期就重新申请。
2. 调用 `registerGeneratedMedia` 登记：`mediaType: "video"`、`url`、`aspectRatio`，`prompt` 写一句视频内容摘要。以后可以用 `listGeneratedMedia` 找回。
   - 带上 `claimedProductId` 会把视频关联到商品，**这个商品下次清洗时会自动用上这个视频**。只有用户想让商品用这个视频时才带，并先告诉用户这一点；只是存档就不带。
3. 用户要把视频挂到 Temu 商品上时，调用 `applyMediaToCleanedProduct`（`claimedProductId`、`shopId` 为目标 Temu 店铺、`video.url`）。这是写操作，调用前先确认。注意：
   - 商品必须已清洗成功（`cleanStatus=success`）；
   - 只改待发布的内容。商品已经发布上架的，线上视频不会变，要告诉用户；
   - 之后如果强制重新清洗这个商品，挂上的视频会丢失。

交付时给用户：成片链接、时长、比例、实际花费（`getMediaCharges` 的 `totalCredits`），以及是否已挂到商品。

## 出错时

- 生成失败时，用一句人话说明原因，保留 Hypit 的 Build 信息和相关 ID；**同一个错误出现两次就停下来**告诉用户，不要反复重新提交。
- Build 因网络、超时等原因中断时，**不改任何文件**直接再执行一次 `npx hypit build`：同样的请求 X-Border 会返回原来的任务和成片，不重复扣费（可以用 `getMediaCharges` 核对）。改了提示词、参考图或参数就是新请求，会再扣费，先征得用户同意。
- 返回 401：按 `x-border` 技能的说明重新登录。
- 返回余额不足：告诉用户当前余额和这次需要的费用，不要降低质量自动重试。
