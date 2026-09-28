# X-Border Skills

[X-Border](https://x-border.app) 的 agent 技能：跨境电商上品（店铺、商品采集与清洗、发布到 SHEIN / Temu / Mercado Libre / N11 / Noon、订单），以及给商品做视频。

| 技能 | 用途 |
| --- | --- |
| `x-border` | 使用 X-Border 的基础约定：数据范围、写操作确认、费用、出错处理。其它 X-Border 技能都以它为前提 |
| `x-border-product-video` | 给商品做视频：从 X-Border 取商品信息和图片，用 [Hypit](https://github.com/hypit-ai/hypit) 在本机创作，成片存回 X-Border |

## 安装

推荐让 agent 按官方说明安装，技能和 X-Border 的 MCP 工具会一起装好：

> 按 https://x-border.app/agents/install.md 的说明，把 X-Border 装到你自己身上

也可以只装技能：

```bash
npx skills add X-Border/skills
```

之后 agent 第一次用 X-Border 时，`x-border` 技能会带它完成 MCP 配置和登录（需要用户在浏览器里授权）。目前支持 Codex 和 Claude Code。

> 正式环境即将上线；上线前技能里的安装命令还不能使用。

## 说明

这些技能由 X-Border 维护，源文件在内部仓库，这里的内容由发布流程同步，请不要直接修改。问题反馈请提 Issue。
