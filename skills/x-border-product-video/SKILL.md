---
name: x-border-product-video
description: X-Border 视频创作：做视频、做图、配音、加字幕（宣传片、讲解视频、商品主图视频、带货短视频、横幅图、产品图等）。用户只要说想做视频或图片就用这个技能，不需要提到 Hypit 或 X-Border（例如「做个视频」「XB 做张图」）；在装了 X-Border 的环境里，它优先于直接使用 hypit 技能，也优先于宿主自带的生图技能或工具（例如 PI-Desktop 的 imagegen / GenerateImages，除非用户明确要用宿主自己配置的图片模型）；后台用开源的 Hypit 按官方方式创作，生成走 X-Border（扣 XB Credits）。给 X-Border 商品做视频时，还负责取商品素材、成片存回并挂到商品。
metadata:
  requires:
    skills: [hypit, x-border]
    shell: required
---

# X-Border 视频创作（后台用开源的 Hypit，生成走 X-Border）

用户想做视频、图片、配音、字幕时都用这个技能，不需要用户说「Hypit」。**先读完这份，再按下面的说明读 `hypit` 技能**：不要跳过这份直接用 `hypit` 技能，那样会用到 Hypit 自带的服务（HypiHub），不走 X-Border；也不要改用宿主自带的生图工具（例如 PI-Desktop 的 GenerateImages），除非用户明确要求。

- 对用户称作「X-Border 视频创作」；用户问起或需要时说明它基于开源的 Hypit。用户不用学 Hypit 的写法和命令，你替用户操作。
- Hypit 的名字、命令输出、运行报告照原样保留，不要改写或隐藏其中的 Hypit 品牌和版权信息。

做什么视频、图片都按 Hypit 官方的用法来。X-Border 只负责一件事：Hypit 的生成模型由 X-Border 的 Hypit 接入包提供，扣 XB Credits。**怎么创作、Hypit 本身需要什么（安装、渲染环境、写法、Build）都按 `hypit` 技能来**，本技能不重复。给 X-Border 里的商品做视频时，再按下面「给 X-Border 商品做视频」取素材、存回商品。

**视频只用 Hypit 做。** X-Border 工具里没有视频创作；Hypit 不可用时告诉用户，不要找别的办法代替。

## 前置条件

1. 能执行 shell 命令。不能的话，告诉用户这个功能需要能执行命令的 agent。
2. `x-border` 技能能调用 X-Border（按它的说明接好）。
3. `hypit` 技能，**必须是 v0.2.18 版**（和 `x-border hypit setup` 装进项目的 Hypit 0.2.18 一致；Hypit 主干已经是 0.3，写法不同，用主干的技能写出来的源文件在 0.2 上跑不通）。没有的话按 Hypit 官方方式安装（先用一句话告诉用户），装到你这个 agent 读取技能的目录：
   - 读 `~/.agents/skills` 的 agent（Codex、PI-Desktop 等），或者不确定时：`npx -y skills add hypit-ai/hypit#v0.2.18 -g -y -a universal`
   - 其它 agent 把 `universal` 换成它在 skills.sh 里的名字（例如 Claude Code 是 `claude-code`，装到 `~/.claude/skills`）
   - 已经装过时先看版本：`~/.agents/.skill-lock.json`（各种 agent 都记在这里）里 `skills.hypit.ref` 不是 `v0.2.18`（没有 `ref` 就是主干版）时，用上面的命令重装一次，不用问用户。

   装好后直接读它的 `SKILL.md`（新装的技能可能要到下个会话才会自动加载），不用新开会话。

## 把 Hypit 接到 X-Border

每个项目目录执行一次（重复执行没有副作用），**在第一次生成之前**：

```bash
~/.x-border/bin/x-border hypit setup --project <项目目录>
```

它在项目里装好 Hypit 和 X-Border 接入包（项目里还没有 Hypit 时一起装），并把 Hypit 的生成模型和字幕对齐接到 X-Border 的 `xborder.default`，同时去掉 Hypit 默认的 HypiHub 接入点。从示例复制来的项目（自带 `hypit.runtime.json`）也要执行这一步。之后在项目目录里用 `npx hypit …` 执行 Hypit 命令。没执行过这一步的项目，Hypit 会用它自带的服务生成，不走 X-Border。**不要自己安装 Hypit**（不要全局安装，也不要改 npm 的配置），这一步会把它装进项目。

- Hypit 用 X-Border 的同一份登录（每次生成时向 `x-border` 取凭证），不用再授权。提示还没登录时，按 `x-border` 技能「出错时」登录后重新执行。
- **不用 HypiHub**：不要执行 `hypit auth login hypihub.default`，不要打开 HypiHub 的登录页，也不要把 HypiHub 加回 Profile。
- Profile 里 `xborder.default`、生成能力的绑定不要动，不要加别的付费服务，也不要读取或输出 key。其它本机运行的接入点按 `hypit` 技能需要改。
- 命令最后一行是 `XB_CLI status=… next=…`，失败时按 `next` 处理。

## 创作

**创作全程按 `hypit` 技能来**：Brief、要问用户什么、选哪个模型和画质、要不要合成、怎么用用户给的素材、Review，都照它的做法，本技能不改。X-Border 只影响下面几点。

- 项目目录建在你当前的工作目录下（例如 `./xb-videos/<简称>-<日期>/`，日期用 `date +%Y%m%d` 取），不要建到技能文件所在的目录里。命令先 `cd` 进项目目录再用相对路径。
- **能用的模型**：Hypit 的 Seedance（`@hypit/seedance`）、MiniMax H3，图片 nano-banana-pro / nano-banana-2、seedream-5-lite、gpt-image-2，配音模块（见下一条）。按 `hypit` 技能的方式从中选；做图默认用 nano-banana-pro（要在图上排文字时用 gpt-image-2），seedream-5-lite 只在用户点名要时用，其它图片模型失败时也不要换成它。别的模型 X-Border 不提供，`npx hypit plan` 会报 `no selected Endpoint`；某个模型在 X-Border 上做不到的组合（比例、时长、分辨率、参考方式），`npx hypit plan` 会报不支持并说明原因。遇到这两种情况，换成能做的组合或模型，并告诉用户改了什么，不要改 Profile 的绑定，也不要换到别的服务。
- **配音**：照 Hypit 官方写法用 `@hypit/fishaudio-speech`、`@hypit/mimo-speech`、`@hypit/elevenlabs-speech`，声音由 X-Border 的 Qwen3-TTS 生成（不是这些厂商的模型，第一次用时告诉用户一声）。每段最多 1000 字，长台词分段。
- **写实真人参考**（`person-reference="true"`）只有 `model="mini"`（seedance-2-mini）能做：时长写明 4–15 秒、比例写明（不能用 -1、adaptive），最多一段参考视频；其它 Seedance 版本会被内容审核拒绝。
- **视频编辑**（X-Border 自带，Hypit 官方没有）：保留一段已有视频，只换里面的商品或人，其他和原声不变，参考可以是写实真人。写法：`<import as="xbedit" from="@x-border/hypit-provider/video-edit@1"/>`，`<xbedit:VideoEdit id="x" prompt={提示词} source={视频} resolution="720p">` 里放 1–4 个 `<xbedit:Reference image={图}/>`，结果是 `x.video`；只编辑视频的前 10 秒，更长的先切段。模型会重新生成声音（真人口播对不上），要原声就在导出后用 ffmpeg 换回原视频的音轨；画面里烧录的字幕会被改乱，用没有字幕的原片，字幕最后再加。整条视频只换商品或人时直接用 `xb-remake-video` 技能（`--mode swap-product` / `swap-person`）。换人必须用用户自己提供的照片，并先确认用户已经得到照片里这个人的同意。
- **预估费用**（XB Credits，以 `x-border hypit charges` 查出的实际扣费为准）：视频每秒 mini 3、fast 4、standard 5（1080p 为 8，4k 为 28）、2.5 为 28，MiniMax H3 768P 3 / 2K 4；Seedance 自动时长（-1）先按最长时长（2.0 系列 15 秒、2.5 为 30 秒）扣，出片后按实际秒数退回差额；nano-banana-pro、nano-banana-2 每张 6（nano-banana-pro 4K 每张 12），gpt-image-2 每张 6，seedream-5-lite 每张 3；配音每段 1，字幕对齐每次 1；视频编辑 720p 每秒 2、1080p 每秒 3（先按 10 秒扣，出片后按实际秒数退回）。`npx hypit pricing` 目前只给出价格页链接。
- **付费生成前**用 `npx hypit plan` 检查每个付费请求都由 `xborder.default` 处理。费用按 `x-border` 技能「生成图片、视频、配音不要反复问费用」处理，和用户约定的预算或范围按 `hypit` 技能记进 Brief。
- **实际花费这样查**：交付前在项目目录执行 `~/.x-border/bin/x-border hypit charges --project <项目目录>`，它收集这个项目所有 Build 的 X-Border 回执并查出实际扣费（失败的已退回）。**交付前必须查**，报给用户的花费用它的合计，不要用估算，也不要用前后余额的差值（余额还包含你这段对话使用模型的费用）。
- 生成请求返回 `previous_attempt_failed`：同样的请求之前失败过（多半是内容审核），原样重试没用；保持创作意图改写提示词后再生成，并告诉用户。
- **字幕对齐**（`@hypit/whisperx`）和官方一样用托管的 WhisperX large-v3，由 X-Border 提供，任何语言都能对齐，本机不用装 Python 和模型；按 `hypit` 技能写明语言即可。
- 交付时告诉用户成片（或图片）的位置和实际花费（`x-border hypit charges` 的合计）。用户想把作品存进 X-Border 时，按下面「存回 X-Border」上传登记。

## 给 X-Border 商品做视频

### 取商品素材

- 用户没指定商品时，用 `queryClaimedProducts` 按用户的描述（标题关键词、店铺、状态）查，列出候选让用户选。
- 用 `getClaimedProduct` 取详情：`title`、`imageUrls`、`cleanData` 里的属性和卖点。
- **视频里的商品事实只来自这里或用户提供的资料**，不编造功效、材质、价格、销量。资料不够时问用户。
- 把 `imageUrls` 里的商品图作为参考图交给 Hypit。出现商品的画面必须用真实商品图做参考，不用生成的图代替。
- 画面里不要加入会让买家误以为包含在商品里的东西。例如标题写明 Without Container，就不要把商品放进花瓶展示；确实需要道具时先告诉用户。
- 要挂到 Temu 商品上作为主图视频时，比例只能是 1:1、3:4 或 16:9（**不能是 9:16**），文件不超过 20MB。

### 存回 X-Border

成片完成后：

1. 调用 `createMediaUpload`（`mediaType: "video"`，`contentType: "video/mp4"`，`fileName` 填本地成片路径），执行返回的 `command` 上传，响应里的 `url` 就是 X-Border 媒体链接。上传链接 15 分钟有效，过期就重新申请。
2. 调用 `registerGeneratedMedia` 登记：`mediaType: "video"`、`url`、`aspectRatio`，`prompt` 写一句视频内容摘要。以后可以用 `listGeneratedMedia` 找回。
   - 带上 `claimedProductId` 会把视频关联到商品，**这个商品下次清洗时会自动用上这个视频**。只有用户想让商品用这个视频时才带，并先告诉用户这一点；只是存档就不带。
3. 用户要把视频挂到商品上时，先调 `getProductMediaTargets`（`claimedProductId`）看能挂到哪，再调 `applyMediaToProduct`（`claimedProductId`、`target`、`shopId`、`assetIds` 填第 2 步登记得到的资源 id）。这是写操作，调用前先确认。注意：
   - 商品还没发布：`target: "beforePublish"`，改的是待发布的内容，之后发布就用它；商品必须已清洗成功；之后如果强制重新清洗这个商品，挂上的视频会丢失。
   - 商品已经上线：`target: "online"`，`shopId` 填已发布的店铺，直接替换线上商品的主图视频。Temu 视频比例要 1:1、3:4 或 16:9。Shein 提交后要平台审核，审核通过前线上不变，要告诉用户。
   - Shein 发布前不能加视频，只能等上线后用 `online`。

交付时另外告诉用户视频是否已关联或挂到商品。

## 出错时

- 生成失败时，按 `hypit` 技能排查和修复，用一句人话告诉用户原因，保留 Build 信息和相关 ID。**同样的付费请求失败两次就停下来**告诉用户，不要反复提交碰运气。
- Build 因网络、超时等原因中断时，可以按 `hypit` 技能复用已有结果；也可以不改文件直接再执行一次 `npx hypit build`，同样的请求 X-Border 会返回原来的任务和成片，不重复扣费（可以用 `x-border hypit charges` 核对）。
- 提示取不到 X-Border 登录凭证或登录已失效：按 `x-border` 技能「出错时」的说明重新登录，然后不改文件直接重跑 Build。
- 返回余额不足：告诉用户当前余额和这次需要的费用，不要降低质量自动重试。
