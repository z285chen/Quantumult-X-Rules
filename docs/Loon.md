# Loon 版本说明

首次转换于 2026-11-14，最近同步于 2026-11-25。Quantumult X 和 Loon 两种格式同步维护；Loon 文件位于 [`loon/`](../loon)。

| Loon 文件 | 建议策略 | 对应 Quantumult X 文件 | 有效规则 |
| --- | --- | --- | --- |
| [`loon/Anthropic.list`](../loon/Anthropic.list) | `Anthropic` | [`Anthropic.list`](../Anthropic.list) | 23 |
| [`loon/HongKongBank.list`](../loon/HongKongBank.list) | `Bank` | [`HongKongBank.list`](../HongKongBank.list) | 12 |

## 导入

Loon 的订阅规则文件只写规则本身，不写策略名；策略由 `[Remote Rule]` 中的 `policy=` 指定。把对应行加入配置的 `[Remote Rule]` 段：

```ini
[Remote Rule]
https://raw.githubusercontent.com/z285chen/Quantumult-X-Rules/main/loon/Anthropic.list, policy=Anthropic, enabled=true
https://raw.githubusercontent.com/z285chen/Quantumult-X-Rules/main/loon/HongKongBank.list, policy=Bank, enabled=true
```

`policy=` 必须指向 Loon 中已存在的策略或策略组。需要单独出口就使用 `Anthropic` / `Bank`；不单独出口时可改为 `PROXY`、`DIRECT` 或其他已有策略。规则命中的最终结果还受本地规则、其他订阅规则和 DNS 影响，应在 Loon 的请求记录中确认。

## 语法映射

| Quantumult X | Loon 远程规则 | 说明 |
| --- | --- | --- |
| `HOST,host` | `DOMAIN,host` | 精确匹配完整域名 |
| `HOST-SUFFIX,domain` | `DOMAIN-SUFFIX,domain` | 按域名标签边界匹配，`app-apple.com` 不会被 `apple.com` 命中 |
| `HOST-KEYWORD,word` | `DOMAIN-KEYWORD,word` | 两份源文件均未使用 |
| `IP-CIDR,cidr` | `IP-CIDR,cidr` | 保留原 CIDR，不扩大或缩小 |
| `IP6-CIDR,cidr` | `IP-CIDR6,cidr` | Loon 的 IPv6 类型名为 `IP-CIDR6` |
| `IP-ASN,...` | `IP-ASN,...` | 两份源文件均未使用，未做转换 |
| `USER-AGENT` / `URL-REGEX` | 同名类型 | 两份源文件均未使用，未做转换 |

每行末尾的 Quantumult X 策略字段（`Anthropic` / `Bank`）已移除。Loon 的订阅规则按无策略语法解析，策略只来自 `[Remote Rule]` 的映射；保留策略列会造成字段歧义。

注释和空行保留。默认注释掉的可选规则也同步改成 Loon 类型并去掉策略列，需要启用时直接去掉行首的 `# `，不需要再补策略名。

## IP 规则语义

`Anthropic.list` 的三条官方 API 入站网段在 Loon 中使用以下写法。

```text
IP-CIDR,160.79.104.0/23
IP-CIDR,153.61.224.0/22
IP-CIDR6,2607:6bc0::/48
```

源文件没有 `no-resolve`，因此 Loon 版本也没有添加。按 Loon 3.0.3 及以上的匹配顺序，域名请求先匹配域名规则；只有域名规则都没命中时才会解析 DNS 再匹配 IP 规则。加上 `no-resolve` 会改变语义：只匹配目标地址本来就是 IP 的请求，不再覆盖解析到这些范围的域名。这里优先保持与 Quantumult X 源文件一致；如果更在意避免额外 DNS 查询，可以自行追加 `,no-resolve`，但要接受上述覆盖范围的收缩。

两份规则均使用官方公布的入站范围。ASN 和正则规则在两份源文件中都不存在，因此没有可转换或需要核对的条目。

## 顺序与例外

两份文件都没有 `DIRECT` / `REJECT` 例外行。Loon 与 Quantumult X 源文件保持逐条对应的顺序。2026-11-18 新增的 10 个社区后缀和启用的 2 个精确遥测主机均放在 IP 规则之前；原有有效域名之间的相对顺序保留。后续若在 Quantumult X 文件中加入例外，应同步保持它在 Loon 文件中的相对位置。

## 校验

- 有效条目：`Anthropic` 23 条（18 条 `DOMAIN-SUFFIX` + 2 条 `DOMAIN` + 2 条 `IP-CIDR` + 1 条 `IP-CIDR6`）；`HongKongBank` 12 条（11 条 `DOMAIN-SUFFIX` + 1 条 `DOMAIN`）。
- 注释中的可选规则：`Anthropic` 13 条，`HongKongBank` 14 条，均已按同一映射改写。
- 逐条核对：去掉注释和空行后，两个 Loon 文件的规则数量、顺序、类型和值均与源文件一一对应；Loon 行末没有残留 `Anthropic` 或 `Bank` 策略字段。
- 已按 Loon 官方文档核对 `DOMAIN`、`DOMAIN-SUFFIX`、`DOMAIN-KEYWORD`、`IP-CIDR`、`IP-CIDR6`、`IP-ASN` 的写法，以及订阅规则不含策略、策略由 `[Remote Rule]` 映射的规则。
- 本次新增条目的实际请求、命中策略和最终出口尚未在客户端逐项验收；静态匹配检查不等于完整功能验收。

## 疑似源文件问题

本次没有静默修改业务含义。以下是检查时的观察：

- `DOMAIN-SUFFIX,hsbc` 是单标签后缀，只匹配以 `.hsbc` 结尾的主机。它来自 Quantumult X 源文件中的 `HOST-SUFFIX,hsbc`，README 已说明这是有意加入的品牌顶级域；Loon 的同名规则语义一致，故原样保留。
- `DOMAIN,api.transferwise.com` 不能由 `wise.com` 后缀覆盖，源文件单列是必要的，不是重复条目。
- `Anthropic.list` 中官方产品范围、社区补充和共享遥测主机的依据不同，详见 [Anthropic 说明](Anthropic.md)。2026-11-18 的 10 个社区差集域名未逐项核实归属和用途；两个 Datadog 精确主机的可选遥测用途已按官方 CLI 文档重新核对。

## 维护

修改 Quantumult X 文件后，需要按上表同步 Loon 文件，并更新两边的 `CONVERTED` 日期与有效条数。若新增 IP、ASN、正则或例外规则，先确认 Loon 对应类型的写法，不要直接把 Quantumult X 行尾的策略名带过去。

## 来源

- [Loon 域名规则](https://nsloon.app/docs/Rule/domain_rule)
- [Loon IP 规则](https://nsloon.app/docs/Rule/ip_rule)
- [Loon 订阅规则](https://nsloon.app/docs/Rule/sub_rule)
- [Loon 官方配置示例](https://github.com/Loon0x00/LoonExampleConfig/blob/master/example.conf)
- [Loon 官方规则示例](https://github.com/Loon0x00/LoonExampleConfig/blob/master/Rule/ExampleRule.list)
