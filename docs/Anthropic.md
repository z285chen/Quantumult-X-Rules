# Anthropic / Claude（Quantumult X）

核查日期：2026-10-22。规则文件：[Anthropic.list](../Anthropic.list)。默认启用 10 条：8 条域名后缀、2 条官方 API 入站 IP 范围。

## 导入

订阅地址：

```text
https://raw.githubusercontent.com/z285chen/Quantumult-X-Rules/main/Anthropic.list
```

在 Quantumult X 中先创建或选择代理策略，然后在配置的 `[filter_remote]` 段追加：

```ini
https://raw.githubusercontent.com/z285chen/Quantumult-X-Rules/main/Anthropic.list, tag=Anthropic, force-policy=Anthropic, enabled=true
```

将 `force-policy=Anthropic` 改为自己的策略名，例如 `force-policy=AI`。该策略必须已经存在；`tag` 只是资源名称。`force-policy` 会覆盖文件末列的 `Anthropic`。

需要 Claude 使用专门出口时，将本资源排在会覆盖这些域名的综合 AI、通用代理或直连资源之前，并在请求日志中确认实际命中的规则与策略。策略最终选择哪个出口由用户配置决定。

## 默认范围与证据

| 目标 | 用途 | 依据 |
| --- | --- | --- |
| `anthropic.com` 及子域 | API、MCP 连接器代理、应用静态资源等 | [官方 CLI 网络要求][network]、[官方桌面端网络要求][desktop] |
| `claude.ai` 及子域 | 网页、账户认证、原生安装与更新 | [network]、[desktop] |
| `claude.com` 及子域 | 登录跳转、控制台、SSO、帮助与文档 | [network]、[desktop] |
| `claude.app` 及子域 | 桌面端资源和动态预览域名 | [desktop] |
| `claudeusercontent.com` 及子域 | Artifacts 内容、文件预览、Chrome 扩展桥接 | [network]、[desktop] |
| `claudemcpcontent.com` 及子域 | MCP Apps 等对话内交互组件 | [network]、[desktop] |
| `claude.dev` 及子域 | 开发者博客 | [Claude 官网][home]的 Developer blog 链接直接指向该域名 |
| `clau.de` 及子域 | 短链接候选 | [v2fly 社区清单][community]；未出现在本轮核对的官方网络要求中，未独立确认每个入口的用途 |
| `160.79.104.0/23` | 官方 API 入站 IPv4 | [官方 IP 文档][ips] |
| `2607:6bc0::/48` | 官方 API 入站 IPv6 | [ips] |

域名后缀规则同时匹配根域和各级子域，因此不再单列 `api.anthropic.com`、`platform.claude.com`、`sso.claude.com`、`downloads.claude.ai` 等重复条目。动态预览与 MCP 内容子域也由后缀规则覆盖。

IP 规则使用 Quantumult X 原生的三列 `IP-CIDR` / `IP6-CIDR` 写法。这里采用官方 API 入站范围；`160.79.104.0/21` 是官方公布的服务端出口范围，整个 IPv6 `/32` 也不是官方公布的 API 入站范围，因此均未扩大纳入。IP 规则作为可识别目标地址的补充，不能替代域名规则或覆盖全部共享 CDN。

## 可选共享依赖

文件中保留了以下 14 条精确主机规则作为注释。它们来自当前官方网络要求，按使用功能决定是否启用：

| 功能 | 主机 | 何时需要 |
| --- | --- | --- |
| GCS | `storage.googleapis.com` | 插件安装量和元数据；Claude Code 2.1.116 之前的原生安装与自动更新 |
| npm | `registry.npmjs.org` | npm / bun 安装、插件包和依赖、通过 npx 启动的 MCP 服务 |
| GitHub | `github.com`、`raw.githubusercontent.com` | GitHub 插件市场和插件、更新日志 |
| Homebrew | `formulae.brew.sh` | Homebrew 安装方式的更新版本检查 |
| Artifacts 字体 | `fonts.googleapis.com`、`fonts.gstatic.com` | 使用 Google Fonts 的内容；不可达时可回退到系统字体 |
| Artifacts JavaScript 库 | `cdnjs.cloudflare.com`、`cdn.jsdelivr.net`、`cdn.tailwindcss.com`、`code.jquery.com`、`unpkg.com` | 加载对应外部库的内容；库不可达时相关功能可能无法运行 |
| 可选遥测与错误上报 | `http-intake.logs.us5.datadoghq.com`、`browser-intake-us5-datadoghq.com` | 官方列出的 Claude Code 遥测端点；具体发送条件和关闭方法见 [network] |

注释表示本资源默认不为这些主机选择策略，后续规则仍可匹配它们。如果现有 Google、GitHub、npm 或通用代理规则已提供可用出口，无需在本资源中重复启用。

需要启用某项时，编辑自己仓库中的 `Anthropic.list`，移除对应行开头的 `# `，更新头部启用条数和核查日期，再刷新 Quantumult X 资源。共享主机上的其他应用请求也会受到影响；精确主机匹配无法区分同一 GCS 主机下的 bucket 或同一 CDN 上的不同产品。

文件末尾另有两条默认注释的生态补充：`claudemcpclient.com` 来自社区清单，端点用途尚未独立核实；`modelcontextprotocol.io` 是 MCP 规范网站，可根据文档访问需要启用。

## 覆盖边界与维护

当前范围面向直接访问 Anthropic 服务的网页、桌面端和 Claude Code。自定义 MCP、企业外部 SSO 身份提供商、第三方模型供应商、网关以及其他安装依赖可能需要自己的规则。

后续新增目标时，记录产品功能、目标主机、来源或实际请求日志。共享服务优先列出具体主机，并说明影响范围。宽泛的 `datadog`、`sentry`、`sift` 关键词以及整个第三方后缀、ASN、NTP 集合不属于这份默认清单。

已完成静态字段、域名、CIDR、重复条目和官方主路径覆盖检查；尚未在实际 Quantumult X 客户端中导入或验证完整功能。使用时分别查看登录、对话、文件与 Artifacts、MCP 组件、安装更新的请求日志，确认所需目标的命中与出口。规则只选择网络策略，不承诺账户状态或风控结果。

## 来源

- [Claude Code 网络要求][network]
- [Claude Desktop 网络要求][desktop]
- [Anthropic 官方 IP 地址][ips]
- [Claude 官网][home]（开发者博客链接）
- [v2fly Anthropic 清单快照][community]，2026-10-21 提交 `bdf45ac`
- [Quantumult X 官方配置示例][qx]（规则类型、策略字段与 force-policy）

[network]: https://code.claude.com/docs/en/network-config#network-access-requirements
[desktop]: https://code.claude.com/docs/en/desktop#network-access-requirements
[ips]: https://platform.claude.com/docs/en/api/ip-addresses
[home]: https://claude.com/
[community]: https://github.com/v2fly/domain-list-community/blob/bdf45acfdeaec2644257bb8947e0a0294276d5ef/data/anthropic
[qx]: https://github.com/crossutility/Quantumult-X/blob/master/sample.conf
