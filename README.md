# Quantumult X Rules

个人维护的 Quantumult X 分流资源。每份规则的覆盖范围、依据和可选依赖见对应说明。

| 规则 | 策略名 | 说明 |
| --- | --- | --- |
| [Anthropic.list](Anthropic.list) | `Anthropic` | Claude 网页、API、桌面预览、Artifacts 与 MCP 内容；[来源与维护说明](docs/Anthropic.md) |
| [HongKongBank.list](HongKongBank.list) | `Bank` | 汇丰香港、大象银行、ZA、Wise；详见下方说明 |

## Anthropic / Claude

核查日期：2026-10-22。默认启用 10 条规则，包含 8 条域名后缀和 2 条官方 API 入站 IP 范围。共享安装、插件、字体、JavaScript 库和遥测主机作为注释保留，可按需启用。

订阅地址：[Anthropic.list](https://raw.githubusercontent.com/z285chen/Quantumult-X-Rules/main/Anthropic.list)。在配置的 `[filter_remote]` 段追加：

```ini
https://raw.githubusercontent.com/z285chen/Quantumult-X-Rules/main/Anthropic.list, tag=Anthropic, force-policy=Anthropic, enabled=true
```

先创建或选择实际使用的策略，并将 `force-policy=Anthropic` 改为该策略名，例如 `force-policy=AI`。`force-policy` 覆盖文件末列的策略；`tag` 只是资源名称。需要专门出口时，将本资源排在覆盖相同目标的综合规则之前，并在请求日志中确认命中结果。

默认域名包括 `anthropic.com`、`claude.ai`、`claude.com`、`claude.app`、`claudeusercontent.com`、`claudemcpcontent.com`、`claude.dev`，以及社区清单中的 `clau.de`。官方依据、社区条目、可选依赖和覆盖边界见 [Anthropic 说明](docs/Anthropic.md)。已做静态校验，尚未在实际 Quantumult X 客户端中验证。

## Hong Kong Bank（自用草稿）

范围：汇丰香港、大象银行、ZA、Wise。不是全部香港银行清单，也不是银行官方允许列表。整理日期：2026-09-18，尚未在用户设备上验证。

### 使用

`HongKongBank.list` 为原生 Quantumult X 分流资源，策略名是 `Bank`。先在 Quantumult X 中创建或选择同名策略，再导入规则文件。出口可手动选择直连或自己的节点；规则本身不承诺哪种出口能避免风控。保持测试期间出口不变，便于比较结果。

订阅地址：[HongKongBank.list](https://raw.githubusercontent.com/z285chen/Quantumult-X-Rules/main/HongKongBank.list)。

在配置的 `[filter_remote]` 中添加：

```ini
https://raw.githubusercontent.com/z285chen/Quantumult-X-Rules/main/HongKongBank.list, tag=Hong Kong Bank, force-policy=Bank, enabled=true
```

需要已存在名为 `Bank` 的策略。若订阅设置使用 `force-policy`，它会覆盖文件中的 `Bank`。实际命中结果还受现有规则、DNS 和资源配置影响，应检查请求日志。

### 取舍与证据

- 汇丰：保留 `hsbc.com.hk`，根据社区 HSBC 清单增加 `hsbc.com`、品牌顶级域 `hsbc`、`hsbc.net`、`hsbcnet.com`、`vv1865.com`。它们扩大了共享服务和 PayMe 范围，不表示香港个人 App 必须访问每项；`hsbc.com` 等也会影响汇丰其他业务。未纳入全部地区分行域名。
- 大象：使用 `elebank.com`，其子域已覆盖公开官网、企业网银和帮助中心。并未证明覆盖全部 App 服务。
- ZA：三个域名来自用户提供的原始规则，尚未独立验证每个域名的具体用途及运行必要性。
- Wise：保留 `wise.com`，加入官方文档列出的 `api.transferwise.com` 精确主机，不扩大到整个历史域名。Wise 作为个人常用金融服务纳入，不代表其具有香港银行身份。
- 第三方：完整保留原规则中的相关候选作为注释，默认不启用。专属名称也不等于用途已验证；共享统计、客服、风控服务不能根据 App 身份隔离。若需要出口一致性，可逐项启用并复测。
- 不加入帖子中的整个 Akamai、Azure、Microsoft 通用域名范围，也不因为域名解析到共享 CDN 就直接把该 CDN 整域加入。
- 删除已被后缀规则覆盖的冗余主机；未纳入帖子中缺乏进一步核验的其他候选域名。

### 验收方式

分别查看 App 启动、登录、账户查询、转账页面、投资页面和客服的请求日志。记录 App/系统版本、测试日期、目标域名、命中规则及出口；无需为测试实际提交转账。只把能关联到具体流程的遗漏加入规则。对可选第三方项目逐项比较启用前后表现。不要把能打开首页当作全功能覆盖，也不要把人脸验证次数直接归因于某条域名规则。

### 参考

- 原始 Bank.list：https://raw.githubusercontent.com/yeahwu/Rules-For-Quantumult-X/refs/heads/main/Rules/Services/Bank.list
- V2EX 讨论（经验线索，并非官方风控结论）：https://www.v2ex.com/t/1178174
- MetaCubeX HSBC 域名清单：https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/hsbc.list
- PayMe 分类参考：https://raw.githubusercontent.com/KIDA-MNESIA/gadgets/main/geosite/hsbc.list
- 汇丰企业银行入口：https://www.hsbcnet.com/
- 大象银行官网：https://www.elebank.com/
- Wise API 文档：https://docs.wise.com/guides/product/send-money/use-cases/affiliates
- Quantumult X 官方配置示例：https://raw.githubusercontent.com/crossutility/Quantumult-X/master/sample.conf
