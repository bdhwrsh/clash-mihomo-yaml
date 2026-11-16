# Clash / Mihomo 通用版配置

主包本人从多年日常魔法练习中整理出来的 Mihomo（Clash Meta）通用配置模板。国内服务以直连为主，境外服务按用途与出口稳定性精细分组，兼顾防误判、低延迟、满速下载、省电量开销与开箱即用。

每一条配置、规则(rules)都经过了长期探索、精心打磨。敬请品鉴！

**节点、密码、公钥和订阅链接全部为示例占位值，请填写自己的参数后再使用。**

👉 [查看配置模板](./clash-mihomo.yaml) · [下载原始 YAML](https://raw.githubusercontent.com/bdhwrsh/clash-mihomo-yaml/main/clash-mihomo.yaml)

## 适用范围

- 面向支持本文件字段的 **Mihomo 内核客户端**（桌面端、移动端或软路由外壳），例如：[Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev)、[FlClash](https://github.com/chen08209/FlClash)、[Clash Mi](https://github.com/KaringX/clashmi)、[OpenClash](https://github.com/vernesong/OpenClash) 等；端口、系统代理、TUN 与控制面板由客户端接管。
- **不承诺原样兼容非 Mihomo 内核**：旧版 Clash、Stash、sing-box 等对 `geox-url`、`support-x25519mlkem768`、MRS 规则集或 `sniffer` 的支持存在差异。
- Reality 节点需核对客户端与服务端的 TLS 指纹及 MLKEM 要求（参考 [Mihomo TLS / Reality 说明](https://wiki.metacubex.one/config/proxies/tls/)）。
- `PROCESS-NAME` 规则能否生效取决于运行平台；OpenClash 等路由器环境看不到本机进程，会静默忽略或删除这些行。

## 核心调校亮点

- **根治国内 App 误判「海外版」**：内置微信海外加速 CDN（`43.160.0.0/16`）与阿里云海外接入段（`8.208.0.0/12`、`47.235..246`）裸 IP 直连，解决挂代理后淘宝/闲鱼等无域名长连接被判成海外站的问题。
- **Apple 全家桶四档精细分流**：
  - Siri / Apple Intelligence（含 PCC 中继、`gspe1-ssl.ls.apple.com` 与专用裸 IP 池 `17.248.128.0/17`）及 iCloud Private Relay 走 `🇺🇳 稳定出口`；
  - Apple TV+ / News / Fitness+ 与 iCloud 邮件收发（`mail.me.com`）走 `🔰 节点选择`；
  - App Store、系统更新、iCloud 照片/文件大流量同步与 APNs 推送（含 `17.0.0.0/8` 与 5 段全球 IPv6 根网段）归入 `🍎 苹果服务`（默认直连），避免大流量白跑代理；
  - `captive.apple.com` 与 `time.apple.com` 置顶直连，保证公共 Wi-Fi 登录页弹出与时间同步。
- **AI 与风控服务固定出口**：OpenAI、Claude（含桌面客户端进程）、海外 AI 类目归入 `🇺🇳 稳定出口`（`fallback` 模式平时固定首选节点，仅在节点断连时按序切换，避免 `url-test` 频繁跳 IP 触发风控）；对归属地风控敏感的国际金融/支付服务归入 `🏦 专用出口`（`lazy: true` 按需检测）。
- **满速下载与云游戏裸 IP 兜底**：`steam@cn`、`microsoft@cn` 前置直连跑满本地带宽；内置 GeForce NOW 调度域名、桌面进程与全球串流机房裸 IP UDP 段，避免串流流量漏入兜底绕远路。
- **开箱即用的防漏与无损嗅探**：前置完整私有/保留网段、Captive Portal 与 NTP 直连；域名嗅探采用 `override-destination: false` + `parse-pure-ip: true`，只为无域名连接恢复分流能力，不破坏自建服务或内网解析。

## 开始使用

1. 下载 `clash-mihomo.yaml`，在自己的设备上另存一份（例如 `config.local.yaml`）。
2. 按下方任一场景填入自己的节点或订阅链接，并同步调整分组引用。
3. 首次启动需要下载 GEO 数据和两个远程 `.mrs` 规则集（Apple ASN 与 Telegram GeoIP），请确保首次拉取时网络可达。
4. 导入客户端后，检查 `🔰 节点选择`、`🇺🇳 稳定出口` 和 `🏦 专用出口` 的默认落点是否符合预期。

> 请保存并使用自己的本地修改版；若直接订阅原始模板链接，后续更新会覆盖你的参数。

### 场景一：自建节点为主（可搭配远程订阅）

`proxies` 保留了三台示例服务器的六个位置，顺序统一为 **`A Reality`、`A Hysteria2`、`B Reality`、`B Hysteria2`、`C Reality`、`C Hysteria2`**：
- **示例A / B**：代表常用主力低延迟节点（默认加入 `♻️ 自动选择`）；
- **示例C**：代表特定地区或备用节点（默认用于 `🌍 地区选择` 与 `🏦 专用出口`）。

| 位置 / 字段 | 应填写的内容 |
| --- | --- |
| `server`、`port` | 自己的服务器域名或 IP、监听端口 |
| Reality `uuid` | 服务端分配的 UUID |
| Reality `servername`、`public-key`、`short-id` | 与服务端一致的 SNI、公钥和 short-id（`short-id` 保留引号） |
| Hysteria2 `password`、`sni` | 服务端密码和证书对应的域名 |
| Hysteria2 `up`、`down` | 按实际带宽与服务端限制填写（模板数值仅作示例） |
| `proxy-providers.subscription.url` | 自己的 Clash / Mihomo 格式节点订阅地址 |

- 删掉或重命名节点时，记得同步修改下方各 `proxy-groups` 中的 `proxies` 列表。
- `hosts` 中的示例映射默认关闭；仅需固定节点域名解析时再取消注释并启用。
- 如果**完全不用远程订阅**：删除整个 `proxy-providers` 段、`🚀 订阅优选` 组，以及其他组中所有 `- 🚀 订阅优选` 引用。

### 场景二：只使用远程订阅

1. 将 `proxy-providers.subscription.url` 替换为自己的 Clash / Mihomo 订阅链接。
2. 删除六个示例节点，将顶层节点段改为 `proxies: []`。
3. 从**所有策略组**的 `proxies` 列表中删除 `示例A`、`示例B`、`示例C` 开头的条目。
4. 将 `♻️ 自动选择` 和 `🇺🇳 稳定出口` 中变空的 `proxies` 字段替换为：

   ```yaml
   use:
   - subscription
   ```

5. 在客户端将 `🔰 节点选择` 切到 `🚀 订阅优选`（或使用已接入订阅的 `♻️ 自动选择`）。

> 注意：`🏦 专用出口` 此时将通过 `🇺🇳 稳定出口` 回退，**不保证特定国家/地区出口**；如有地区锁区需求，请为该组筛选或指定对应地区节点。

## 策略组说明

按 YAML 内的排列顺序：

| 分组 | 类型 | 用途 |
| --- | --- | --- |
| 🔰 节点选择 | `select` | 常用代理总入口，手动选节点或自动测速组 |
| ♻️ 自动选择 | `url-test` | 对常用主力节点（示例 A/B）定时测速选优（10 分钟 / 150ms 容差） |
| 🚀 订阅优选 | `url-test` | 对远程订阅中的节点自动测速选优 |
| Ⓜ️ 微软服务 | `select` | Microsoft / GitHub 服务（`microsoft@cn` 已前置直连） |
| 🎬 社媒视频 | `select` | YouTube、Netflix、Spotify、TikTok、X、Meta、Discord 等 |
| 🌍 地区选择 | `select` | 为有特定地区要求的服务手动选择出口（组名本身不筛选地区） |
| 🎮 游戏代理 | `select` | Steam、EVE Online、GeForce NOW 云游戏及裸 IP 串流段 |
| 🍎 苹果服务 | `select` | 常规 Apple 服务与 iCloud 大流量同步，默认直连 |
| 🇺🇳 稳定出口 | `fallback` | AI、Siri 等对频繁换 IP 敏感的服务：平时锁定首位可用节点，断连才按序切换 |
| 🏦 专用出口 | `fallback` | 境外金融/支付等专用出口（默认隐藏，开启 `lazy: true` 按需检测） |
| 🎯 全球直连 | `select` | 统一直连出口（默认隐藏） |
| 🐟 漏网之鱼 | `select` | 以上所有规则均未命中时的最终兜底 |

## 调整时留意

- **规则从上到下匹配**：私有网段、Captive Portal 探测、NTP 时间同步以及 `@cn` 子类目（`steam@cn`、`microsoft@cn`、`apple-cn`）必须留在对应主类目或兜底规则之前。
- **DNS 解析与流量出口分离**：`nameserver-policy` 控制域名由哪个上游解析；当前 `respect-rules: false` 表示 DNS 查询本身不跟随 `rules` 走代理。有内网自建域时请将 `geosite:private` 下的 DNS 换成内网 DNS（参考 [Mihomo DNS 文档](https://wiki.metacubex.one/config/dns/)）。
- **组名不自动过滤地区或 UDP**：需要特定国家或低延迟通话/游戏时，请确认对应节点实际出口支持情况与 `udp: true`。
- **客户端覆写会改变行为**：若导入后效果不符，优先检查客户端是否开启了 DNS、TUN 或 Sniffer 覆写。

## 检查自己的修改

修改完成后，建议使用支持本模板字段的 Mihomo 内核执行语法与引用校验：

```sh
mihomo -t -f config.local.yaml
```

`-t` 仅能检查 YAML 语法、字段解析与节点/策略组引用完整性，不能替代真实凭据和节点连通性测试。`.gitignore` 已默认排除 `config.local.yaml`、`*.local.yaml`、`providers/` 与本地数据文件，切勿将真实订阅 URL、UUID、密码或服务器地址提交到公开仓库。

## 上游参考

- [MetaCubeX / mihomo](https://github.com/MetaCubeX/mihomo) · [Mihomo 官方配置文档](https://wiki.metacubex.one/)
- [MetaCubeX / meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat)：GEO 数据与 `.mrs` 规则集上游
