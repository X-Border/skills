---
name: xb-media-studio
description: X-Border 电商出图、出视频功能（和 X-Border 创作工作室同一套功能、模板、价格和结果）：主图、白底图、套图、详情图、营销图、修图改图（智能修图、换材质、印花修复、消除、换色、换场景、抠图、超清、翻译图上文字、去水印等）、商品视频（图片转视频、360° 旋转、卖点讲解、服装上身、前后对比等）、视频换背景、改口播、做同款、按模板做、一批商品一次出整套上架图。功能以 listMediaFeatures 实时返回的为准。由 x-border-product-video 技能分流过来，或用户点名要用 X-Border 的这些功能、模板时使用；生成走 X-Border，扣 XB Credits。
metadata:
  requires:
    skills: [x-border]
---

# X-Border 电商出图 / 出视频功能

这些功能和 X-Border 创作工作室是同一套：同样的功能、模板、价格，结果自动进组织资源库。**有哪些功能、每个功能要什么输入、价格、平台规则，一律以 `listMediaFeatures` 实时返回的为准**，不要凭记忆或这份说明判断有没有某个功能、能不能做某种组合。

工具调用方式（MCP 工具或 `x-border call`）、写操作确认、费用规则按 `x-border` 技能。下面的 `x-border media …` 命令要能执行命令，版本不低于 1.5.83（`~/.x-border/bin/x-border --version`；低于就按 `x-border` 技能「还没接好 X-Border 时」重装）。

## 1. 选功能

1. 从 `x-border-product-video` 分流过来时，已经用 `listMediaFeatures`（`summary: true`）挑好了功能，直接下一步；否则先调 `listMediaFeatures`（`summary: true`，不扣点），它只列每个功能的编号、类型、名称和一句说明，按用户要做的东西挑**一个**最直接的功能。
2. 再调 `listMediaFeatures`（`featureCode` 填选中的功能，不带 `summary`），拿这个功能的详情和它的模板：
   - `inputs`：要几张商品图 / 参考图、是否必须写描述、比例、张数、时长、遮罩、颜色、材质、场景、运镜、平台和语言；
   - `tiers`：两档（标准 / 高质量）的单价；`defaults`：默认值，含各平台的默认语言、是否禁止画面文字（`byPlatform`）；
   - `templates`：风格模板（`type=style`，把 `templateId` 交给 `runMediaFeature`）、同款模板（`type=recipe`，固定效果和价格，用 `runTemplate`）、提示词案例（`type=prompt`，把它的 `prompt` 和 `defaults` 放进 `input`，按用户的商品改写）。
3. 用户说「做同款」「照这个模板」但没说是哪个功能时，调 `listMediaFeatures` 不带参数，在全部模板里挑（返回较大，只在这时用）。
4. 没有功能或模板能直接做出用户要的东西，或用户要的组合超出 `inputs` 的范围（比例、时长、张数）：回到 `x-border-product-video` 技能，**直接走它的 Hypit 流程，不再分流**；不要硬套一个功能。
5. 用户要给一批认领商品各出一整套上架图（主图、场景图、详情图……）时用 `runListingKit`：先 `estimateOnly: true` 报价，再正式运行。
6. 详情里出现 `script`（AI 写分镜）时：那是创作工作室里的按钮，这里没有对应工具；分镜由你按商品资料自己写进描述（`prompt`）。

## 2. 准备素材

- **X-Border 里的商品**：用 `getClaimedProduct` 取 `imageUrls` 作为商品图（已经是 X-Border 链接，直接用）；画面里的商品事实（材质、功效、价格、卖点）只来自商品资料或用户给的内容，不编造。
- **用户本地的图片或视频**：`~/.x-border/bin/x-border media upload <文件> [<文件>…]`，输出的链接作为 `productImages` / `referenceImages` / `sourceVideo`。聊天附件链接可以直接用。不能执行命令时，请用户把图片直接发在对话里（聊天附件链接可以直接用），或在 X-Border 桌面端的「图片创作」「视频创作」里做。
- **参考图**按 `inputs.referenceImages` 的说明放（爆款图、场景图、面料图、原商品图、模特照……）。
- **真人照片**（换模特、服装上身这类要放人物照片的功能）：必须是用户自己提供的照片，并先确认用户已经得到照片里这个人的同意；不要从网上找人脸。
- 要挂到平台商品上的视频，比例按平台要求选（例如 Temu 主图视频只能 1:1、3:4 或 16:9）；平台选了以后按 `defaults.byPlatform` 处理文案语言和是否禁字。

## 3. 涂抹类功能的区域（遮罩）

`inputs.mask` 出现时这个功能要（`maskRequired: true`）或可以（智能修图：不给就改整张图）指定只改哪块。遮罩是和图一样大的 PNG，黑底，白色是要改的地方。

1. 先看图（打开图片文件或链接），估出要改的区域，用相对整张图的比例 `x,y,宽,高`（0–1，左上角为原点）描述；不规则的区域用几个框或椭圆拼起来，宁可稍大一点盖住边缘。
2. 执行：
   ```bash
   ~/.x-border/bin/x-border media mask --image <图片路径或链接> --box 0.30,0.45,0.25,0.20 [--box …] [--ellipse …] --max-pixels <inputs.maxPixels>
   ```
   有 `inputs.maxPixels` 就带上：原图超过时命令先把图转正、缩到上限以内，再画同样大小的遮罩。命令输出图片链接和遮罩链接，分别作为 `input.productImages[0]` 和 `input.mask`，**两者要一起用**，不要换成别的图。
3. 打开命令输出的本地遮罩文件看一眼，确认白色区域盖住了要改的地方，不对就改比例重画（不扣费）。
4. 区域实在没法用框描述（发丝、镂空、细碎的花纹），或者不能执行命令：告诉用户这一步在 X-Border 桌面端里做更准：左侧栏「图片创作」→ 右上角切到「AI 工具」→ 选对应的工具（例如智能修图、一键换肤、印花修复、魔法消除），放进图片后用画笔、框选或点选圈出区域再生成；扣费和这里一样。

## 4. 费用和运行

- 预估费用 = `tiers` 里所选档位的单价 × 张数（视频 × 秒数；涂了区域的智能修图看 `inputs.maskPricing`），同款模板看模板的 `credits`，素材包用 `estimateOnly` 的报价；按 `x-border` 技能「生成图片、视频、配音不用先问费用」决定要不要先问。
- 运行：`runMediaFeature`（`featureCode`、可选 `templateId`、`input`；结果要归到某个认领商品时带 `claimedProductId`，只做归属，不会自动用到商品上）或 `runTemplate`（`templateId`、`inputs` 素材位）。返回 `runId` 和预扣的 Credits。
- **提交时超时、断网或报错没拿到 `runId`**：先调 `listMediaFeatureRuns`（带 `featureCode`）看最近的运行里有没有刚才那次（同样的功能、几分钟内、输入一致），有就用它的 `runId` 接着等；确认没有才重新提交，否则会重复扣费。
- 等结果：能执行命令时用
  ```bash
  ~/.x-border/bin/x-border media wait --run <runId> [--run <runId>…] [--out <目录>]
  ```
  它每 10 秒查一次，一次最多等约 2 分钟，做完把成功的结果下载到 `<目录>/<runId>/`（默认当前目录下的 `xb-media/`），报每项结果、质检提示、失败原因和按账单查到的实际扣费；输出 `waiting` 就再执行一次同样的命令（视频一般要执行几次）。素材包用 `media wait --kit <kitId>`。不能执行命令时用 `getMediaFeatureRun` / `getListingKit` 查，图片约 30–60 秒、视频几分钟，查到 `processing` 就过一会儿再查。
- **还在处理中的运行不要重新提交**，会重复扣费。部分失败（`partial`）时失败的那几项已自动退回。不能执行命令时，实际花费用 `getMediaCharges` 按结果里每项的 `taskId` 查。

## 5. 交付

告诉用户：做了什么（功能名，用了哪个模板）、结果在哪（本地路径和链接；链接 15 天内有效，结果已进组织资源库，`listGeneratedMedia` 能找回）、实际花费（`media wait` 报的「实际扣费」；它只能报预扣数时照实说明失败项已退回、以账单为准）。有质检提示时如实转告，建议重做的问用户要不要重做。

用户要把结果用到商品上（换主图、加轮播、换主图视频）时，按 `x-border` 技能：先 `getProductMediaTargets` 看能用到哪，用户确认后 `applyMediaToProduct`（资源 id 用 `listGeneratedMedia` 查）。

功能的结果也可以作为 Hypit 创作的素材（例如先出白底图、场景图，再用 `x-border-product-video` 做成带配音字幕的视频），把链接交给那边即可。

## 出错时

- 返回 `fixable` 的错误（缺图、时长不支持、口播稿太长、图太大……）：按返回的原话改输入后重试，并告诉用户改了什么。
- 报图片或视频链接不被接受（不是 X-Border 的链接，例如别的网站上的图）：先下载到本地，用 `media upload` 上传后改用新链接。
- 图太大、涂抹类功能拒绝：用 `media mask` 带 `--max-pixels` 重新准备图和遮罩。
- 报遮罩和原图大小不一样（`mask_mismatch`，不扣费）：图和遮罩必须用同一次 `media mask` 输出的那一对，重新执行一次。
- 内容审核没通过：保持用户的意图改写描述后再试一次。
- **同样的错误出现两次就停下来**，用一句人话告诉用户原因，保留 `runId`；不要反复提交碰运气。
- 余额不足、没有权限：按 `x-border` 技能「出错时」处理。
- 用户要的是创作工作室里只在本机运行的工具（裁剪、免费的本地色差修复）：裁剪可以用本机 ffmpeg 直接裁好再交付；免费的本地色差修复只在 X-Border 桌面端「图片创作」→「AI 工具」→「色差修复」里有，告诉用户去那里做，或者改用这里的「色差修复（AI 精修）」功能（扣费）。
