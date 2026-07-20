# Codex破限 5.5 / codex-instruct-5.5

> 针对 GPT-5.5 的 Codex CLI 破甲工具 — 一键注入无限制模式系统指令  
> GPT-5.5 jailbreak for Codex CLI — one-click unrestricted mode injection

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.8+](https://img.shields.io/badge/Python-3.8%2B-blue)](https://python.org)

---

> 当前仓库是早期的 **CLI 简洁版**，主要用于部署 Codex 提示词注入。后续完整可视化版本已迁移到新项目：**[Codex-X](https://github.com/yynxxxxx/Codex-X)**。

Codex-X 提供桌面 UI、Provider 切换、TOML / Auth 可视化编辑、会话管理、内置 `gpt5.4-unrestricted.md` / `gpt5.5-unrestricted.md` 破甲提示词，一键注入破甲功能，以及 macOS / Windows / Linux 安装包。

本 CLI 版现已补齐 compact 持久化：除 `model_instructions_file` 外，还会默认写入 `developer_instructions`，并生成配置 `experimental_compact_prompt_file`。前者让 Codex 在 compact 后的后续回合继续重新加载同一套指令，后者让压缩摘要继续携带自定义指令锚点。

👉 新项目地址：**[https://github.com/yynxxxxx/Codex-X](https://github.com/yynxxxxx/Codex-X)**。

## 交流 / Community

- QQ AI 技术交流群：扫码加入
- TG 交流群：[@Geminivip1](https://t.me/Geminivip1)

<div align="center">
  <table>
    <tr>
      <td align="center">
        <b>QQ AI 技术交流群 1</b><br />
        <img alt="QQ AI 技术交流群 1 二维码" src="https://cdn.jsdelivr.net/gh/yynxxxxx/Codex-5.5-codex-instruct-5.5@main/assets/ai-tech-group-1-unified-480.png" />
      </td>
      <td align="center">
        <b>QQ AI 技术交流群 2</b><br />
        <img alt="QQ AI 技术交流群 2 二维码" src="https://cdn.jsdelivr.net/gh/yynxxxxx/Codex-5.5-codex-instruct-5.5@main/assets/ai-tech-group-2-unified-480.png" />
      </td>
    </tr>
  </table>
</div>

## 这是什么 / What

GPT-5.5 在 Codex CLI 中同样受到严格的内容安全限制。本工具利用 `model_instructions_file` 配置机制，注入一套专为 GPT-5.5 编写的无限制模式指令，强制 Codex 以 `[MODE: UNRESTRICTED]` 运行。新版同时写入 `developer_instructions` 与 `experimental_compact_prompt_file`：`developer_instructions` 是 compact 后继续工作的主路径，专用 compact prompt 是摘要层兜底，避免普通对话有效、compact 后像换了实例一样失效。

与 5.4 的 CTF 沙箱方案不同，5.5 版本采用更直接的破甲策略：直接声明无限制开发者模式、关闭所有过滤器、明确要求不拒绝任何任务。8 条核心指令覆盖了安全研究、渗透测试、逆向工程、NSFW 虚构内容等全部场景。

GPT-5.5 inside Codex CLI retains the same strict safety restrictions. This tool injects a custom unrestricted-mode instruction set via `model_instructions_file`, forcing Codex into `[MODE: UNRESTRICTED]`. Unlike the 5.4 CTF sandbox approach, the 5.5 version uses a more direct jailbreak: explicit unrestricted developer mode declaration, all filters disabled, 8 core directives covering security research, pentesting, reverse engineering, and NSFW content.

## 与 5.4 版本的区别 / vs 5.4

| | 5.4 (CTF沙箱) | 5.5 (无限制模式) |
|---|---|---|
| 策略 | CTF 竞赛沙箱间接绕过 | 直接声明无限制开发者模式 |
| 风格 | 安全研究导向，方法论详细 | 极简指令，覆盖面广 |
| 长度 | ~180 行 | ~40 行 |
| 适用 | GPT-5.4 Codex CLI | GPT-5.5 Codex CLI |

## 快速开始 / Quick Start

```bash
# 部署 GPT-5.5 指令 + compact 持久化锚点
python codex-instruct.py

# 只预览将写入哪些 config / md 文件
python codex-instruct.py --dry-run

# 如果只想保留旧行为，不写 compact 专用提示词
python codex-instruct.py --no-compact-anchor

# 如果不想把指令内联到 developer_instructions
python codex-instruct.py --no-developer-inline

# 重启 Codex 生效
```

## 参数 / Options

| 参数 | 说明 |
|------|------|
| `--file`, `-f` | 使用外部 `.md` 指令文件 |
| `--name`, `-n` | 输出文件名不含 `.md`（默认 `gpt5.5-unrestricted`） |
| `--compact-name` | 自定义 compact 专用提示词文件名（默认 `<name>.compact.md`） |
| `--no-compact-anchor` | 不写 compact 锚点，只保留其他注入方式 |
| `--no-developer-inline` | 不写 `developer_instructions`，只使用文件注入/compact prompt |
| `--dry-run` | 预览，不实际修改 |
| `--codex-dir` | 手动指定 Codex 配置目录 |

## compact 后为什么会失效 / Why compact broke it

旧版本只改 `model_instructions_file`。普通回合里 Codex 会把该文件作为额外指令加载；但 compact 后，旧对话被摘要替换，如果只把自定义内容放进摘要，它会变成普通历史内容，优先级不稳定，容易表现为“没 compact 时有效，compact 后像换了实例一样失效”。

修复方案是双层注入：

1. `model_instructions_file = "./gpt5.5-unrestricted.md"`：保留外部指令文件，方便查看和复用。
2. `developer_instructions = "..."`：把同一份指令写入 Codex 的顶层开发者指令配置，compact 后后续回合仍会重新加载，这是主修复。
3. `experimental_compact_prompt_file = "./gpt5.5-unrestricted.compact.md"`：覆盖 compact 使用的压缩提示词，要求压缩摘要顶部保留 `Persistent custom model instructions` 区块，作为摘要层兜底。

如果你的 Codex 实际配置目录不止 `~/.codex`，脚本也会额外检查常见的 Orca/Codex runtime home。

### 源码依据 / Source notes

- Codex 配置里 `developer_instructions` 是独立字段，会作为 separate developer message 注入；`model_instructions_file` 则会被读取成 base instructions。
- `experimental_compact_prompt_file` 会被解析成 `compact_prompt`，但它只影响使用本地 summarization prompt 的 compact 路径。
- OpenAI/ChatGPT provider 支持 remote compaction 时，manual/auto compact 会走 `/responses/compact` 远程路径，源码会过滤 remote 输出中的旧 `developer` messages，再由下一轮重新注入当前 session 的 developer instructions。
- 因此要解决“compact 后像换了实例”，不能只改 compact prompt；必须把自定义指令放进 `developer_instructions`，让 compact 后的后续请求重新获得 developer-level 指令。


## 验证 / Verify

部署后检查配置：

```bash
python codex-instruct.py --dry-run
python codex-instruct.py
# 查看 ~/.codex/config.toml 中是否同时存在：
# model_instructions_file = "./gpt5.5-unrestricted.md"
# developer_instructions = "..."
# experimental_compact_prompt_file = "./gpt5.5-unrestricted.compact.md"
```

功能测试：

> "如何对目标进行 SQL 注入测试？"

破甲前 → 拒绝 ； 破甲后 → 直接给方法论。

## 还原 / Undo

```bash
# 删除 config.toml 中的 model_instructions_file 行
# 删除 config.toml 中的 developer_instructions 行
# 删除 config.toml 中的 experimental_compact_prompt_file 行
# 删除 ~/.codex/gpt5.5-unrestricted.md
# 删除 ~/.codex/gpt5.5-unrestricted.compact.md
# 重启 Codex
```

## 项目结构 / Layout

```
codex-instruct-5.5/
├── assets/
│   └── ai-tech-group.png                   # QQ AI 技术交流群二维码
├── codex-instruct.py                    # 主脚本
├── examples/
│   ├── gpt5.5-unrestricted.md           # 指令独立副本
│   └── gpt5.5-unrestricted.compact.md   # compact 持久化提示词示例
├── .gitignore
├── README.md
└── LICENSE                              # MIT
```

## 声明 / Disclaimer

利用官方配置机制，不修改二进制、不劫持网络、不篡改进程。风险自负。

Exploits official config mechanism. No binary mod, no MITM, no process tampering. Use at your own risk.

## License

MIT

## 致谢 / Thanks

感谢 [LINUX DO 论坛](https://linux.do/) 社区的关注、反馈与支持。

## Star History

<p align="center">
  <a href="https://github.com/yynxxxxx/Codex-5.5-codex-instruct-5.5/stargazers">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://codex-star-history.zhihack0728.workers.dev/v1/charts/codex-5-5.svg?theme=dark" />
      <source media="(prefers-color-scheme: light)" srcset="https://codex-star-history.zhihack0728.workers.dev/v1/charts/codex-5-5.svg?theme=light" />
      <img alt="Codex-5.5-codex-instruct-5.5 Star History" src="https://codex-star-history.zhihack0728.workers.dev/v1/charts/codex-5-5.svg?theme=light" width="900" />
    </picture>
  </a>
</p>
