<div align="center">

<sub><strong>L0NE-6 · DAILY SIGNAL</strong></sub>

# 🛠️ CodeArts Daily

**CodeArts 每日福利与新人注册礼领取，以回读确认代替“请求已受理”的假成功。**

[简体中文](README.md) · [English](docs/README.en.md) · [繁體中文](docs/README.zh-Hant.md) · [日本語](docs/README.ja.md) · [한국어](docs/README.ko.md) · [Português (BR)](docs/README.pt-BR.md)

<p>
  <img src="https://img.shields.io/github/v/release/L0NE-6/CodeArts-Daily?style=for-the-badge&label=Release&color=2ea44f&logo=github" alt="Release" />
  <img src="https://img.shields.io/github/stars/L0NE-6/CodeArts-Daily?style=for-the-badge&label=Stars&color=FFC75F&logo=github" alt="Stars" />
  <img src="https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python 3.8+" />
  <img src="https://img.shields.io/badge/%E9%9D%92%E9%BE%99-%E5%8D%95%E6%96%87%E4%BB%B6-4EAA25?style=for-the-badge" alt="Qinglong" />
  <img src="https://img.shields.io/badge/GitHub%20Actions-Supported-2088FF?style=for-the-badge&logo=githubactions&logoColor=white" alt="GitHub Actions" />
  <img src="https://img.shields.io/badge/License-MIT-F472B6?style=for-the-badge" alt="MIT License" />
  <a href="https://github.com/L0NE-6/CodeArts-Daily/releases"><img src="https://img.shields.io/badge/%E2%AC%87%20Download-Releases-2ea44f?style=for-the-badge&logo=github" alt="Download Releases" /></a>
</p>

[✨ 30 秒速览](#quick-overview) · [🎯 领取闭环](#claim-flow) · [🚀 青龙部署](#quick-boot) · [☁️ Actions](#actions) · [🔐 登录工具](#credentials) · [🔧 环境变量](#environment) · [✅ 成功判据](#result) · [🚫 使用边界](#usage-policy) · [✦ 星轨](#star-trail)

</div>

---

<a id="quick-overview"></a>

<a id="usage-policy"></a>
## 🚫 使用边界与转载声明

> [!IMPORTANT]
> **官方版本永久免费。** 未经作者书面许可，禁止付费包装、搬运引流、冒充作者，或使用作者身份与二维码制造官方授权假象。违规内容可能被要求下架；作者可暂停更新、归档或删除仓库及相关 Release。

本项目仓库、脚本、部署说明和 Release 下载始终免费公开。未经作者明确书面许可，禁止将本项目包装为付费资源、内部版、独家版、会员权益、收费课程、付费部署或包更新服务；禁止搬运到其他账号、群组、频道、网盘、电商或付费平台进行涨粉、截流、倒卖和引流；禁止删除作者与版权信息后冒充原创；禁止使用作者昵称、头像、二维码和项目名称制造官方授权假象。

> 如需商业合作、品牌授权、联合分发或使用作者名义提供付费服务，请先取得作者明确书面授权。完整规则、收款码防诈骗说明和违规处理方式见 **[NOTICE.md](NOTICE.md)**。

> 官方仓库和 Release 始终免费。第三方以“下载费 / 激活费 / 内部版 / 代部署 / 会员门槛”等名义收费，均与作者无关，请停止付款并保留证据。

> 对上述违规行为，作者有权要求立即停止传播、删除内容并下架相关商品或页面；并可视情况暂停更新、归档或删除本仓库及相关 Release、向平台投诉或采取进一步必要措施。

<a id="support"></a>

---

## ✨ 30 秒速览

| 维度 | 说明 |
| :--- | :--- |
| **用途** | 领取 CodeArts 每日福利，并补领可自动处理的新人注册礼。 |
| **输入** | `CODEARTS_CREDENTIALS`：可自动续期的单行 JSON，或不含续期能力的 `AK\|SK\|STS` 简写。 |
| **输出** | 分账号领取报告，按活动列出实际到账金额、确认状态与失败原因。 |
| **部署** | 青龙面板、GitHub Actions，或本地 Python 运行。 |
| **依赖** | Python 3.8+；`requests`、`cryptography`。 |
| **成功判据** | 活动列表 → claim → confirm → 回读；状态为 `CONFIRMED` 或 `CONSUMED` 才算真正到账。 |

<a id="signals"></a>

## 🎯 核心能力

- ✅ **四步闭环**：桌面口径 `delivery → claim → confirm → delivery 回读`。
- 🔐 **PKCE + DPoP**：本地回调完成官方授权流，取得 AK/SK/STS、refresh_token 与 DPoP 私钥。
- ♻️ **自动续期**：凭据到期前 15 分钟续期并原子落盘。
- 🧯 **终态保护**：一次性 refresh_token 已消费、落盘失败或返回 `STS5.1806` 时停止重试并要求重新登录。
- 🎁 **新人注册礼**：与每日福利共用领取执行体；学生认证、邀请好友不进入自动领取链路。
- 🧾 **幂等与限流**：幂等键先落盘，每日最多 6 次写尝试，两次至少间隔 10 分钟。
- 👥 **多账号与推送**：支持 PushPlus、Bark、企业微信、钉钉和青龙通知。

<a id="claim-flow"></a>

## 🔁 领取闭环

```text
GET  /v1/ops/delivery?channel=DESKTOP
POST /v1/ops/claim
POST /v1/ops/confirm
GET  /v1/ops/delivery?channel=DESKTOP   # 回读 CONFIRMED / CONSUMED
```

- 请求使用 `SDK-HMAC-SHA256` 签名，并按桌面客户端口径包含 `Host`。
- 已领取但缺少 `userBenefitId` 时停止确认，不重放 claim、不拿 `campaignId` 冒充确认 ID。
- HTTP 200 空 body 视为会话过期并进入终态；普通 401/403 仍有独立处理。
- v2 本地台账可平滑迁移到 v3，已有幂等键继续复用。

## 📦 文件说明

| 文件 | 用途 |
| :--- | :--- |
| `codearts_login.py` | PKCE + DPoP 网页登录，输出可直接填入变量的 JSON |
| `codearts_daily.py` | 每日福利与新人注册礼领取、续期、回读验证 |
| `tests/test_codearts_core.py` | 离线核心逻辑回归测试 |
| `requirements.txt` | Python 依赖清单 |

<a id="quick-boot"></a>

## 🚀 青龙部署

1. 安装依赖：`python3 -m pip install -r requirements.txt`。
2. 上传 `codearts_daily.py` 到青龙「脚本管理」。
3. 将一行或多行凭据 JSON 放入 `CODEARTS_CREDENTIALS`。
4. 用青龙运行器建立定时任务。

```text
# 每天北京时间 08:40
40 8 * * * task codearts_daily.py
```

> 青龙运行在远程服务器时，请在本地电脑运行 `codearts_login.py`，再把输出 JSON 填入面板。登录回调指向 `127.0.0.1`，远程面板无法直接接收本机回调。

<a id="actions"></a>

## ☁️ GitHub Actions

1. Fork 或导入仓库到自己的账号。
2. 在 **Settings → Secrets and variables → Actions** 新建 Secrets。
3. 从 **Actions → CodeArts Daily → Run workflow** 手动验证。
4. 默认仅手动触发；需要定时时，在 `.github/workflows/daily.yml` 取消 `schedule:` 两行注释。cron 使用 UTC。

| Secret | 必填 | 说明 |
| :--- | :---: | :--- |
| `CODEARTS_CREDENTIALS` | 是 | 单行 JSON；多个 JSON 必须每个账号一行 |
| `PUSHPLUS_TOKEN` / `BARK_URL` | 可选 | PushPlus / Bark |
| `WECOM_WEBHOOK` / `DINGTALK_WEBHOOK` / `DINGTALK_SECRET` | 可选 | 企业微信 / 钉钉 |

<a id="credentials"></a>

## 🔐 获取凭据

```bash
python3 -m pip install cryptography
python codearts_login.py
python codearts_login.py --paste --no-browser
python codearts_login.py --note 主号
```

工具会在本机启动回调监听，浏览器完成登录后自动换取凭据，并输出一行 JSON。常用参数：

| 参数 | 作用 |
| :--- | :--- |
| `--paste` | 手动粘贴完整回调地址或 `code=...&secret=...` |
| `--no-browser` | 不自动打开浏览器 |
| `--no-verify` | 跳过换取凭据后的校验 |
| `--note 主号` | 在凭据中加入账号备注 |
| `--port 8765` | 指定本机回调端口 |

JSON 包含 refresh_token、PKCE 材料与 DPoP 私钥，请按密码级别保管。

凭据可带可选字段 `oauth_context.client_id`：续期时优先使用其中的非空字符串；缺省、空串或非字符串自动回退 `vscode-codebot`。旧凭据无需迁移。

<a id="environment"></a>

## 🔧 环境变量与参数

多账号使用换行或 `&` 分隔。JSON 凭据必须每个账号独占一行；简写形式可以用 `&` 串接。

| 环境变量 | 格式 | 说明 |
| :--- | :--- | :--- |
| `CODEARTS_CREDENTIALS` | 单行 JSON（推荐） | 含 refresh_token 与 DPoP 私钥时可自动续期 |
| `CODEARTS_CREDENTIALS` | `AK\|SK\|STS` | 可领取，但约 1 小时后到期，无法自动续期 |
| `PUSHPLUS_TOKEN` / `BARK_URL` | Token / URL | 可选推送 |
| `WECOM_WEBHOOK` / `DINGTALK_WEBHOOK` / `DINGTALK_SECRET` | Webhook / 加签密钥 | 可选推送 |
| `QL_URL` / `QL_TOKEN` | 面板地址 / 旧版面板 Token | 可选青龙通知 |

| 参数 | 作用 |
| :--- | :--- |
| `--preview` | 只读活动列表，不发写请求 |
| `--force` | 跳过本地写请求限流 |
| `--only 2` | 只跑第 2 个账号 |
| `--no-notify` | 关闭本轮推送 |

<a id="result"></a>

## ✅ 成功判据

- `claim` 与 `confirm` 的受理结果只代表流程推进，不直接宣布成功。
- 回读活动状态为 `CONFIRMED` 或 `CONSUMED`，才记为确认到账。
- 每日福利与新人注册礼同时存在时，按活动分别提交，并在回读后统一汇报。
- 本地台账限制写请求频率，避免重复领取或高频重试。

```text
[08:40:01] 👤 [1] 主号  AK=AKID****abcd
[08:40:04]    ✅ 福利已确认到账：每日福利 1000；新人注册礼 4000（本次提交 2 笔）
[08:40:05] 账号1 主号: ✅ 福利已确认到账：每日福利 1000；新人注册礼 4000
```

<a id="data-privacy"></a>

## 🗂️ 数据文件与隐私

| 文件 | 内容 | 是否提交 |
| :--- | :--- | :---: |
| `codearts_credentials.json` | 续期后的完整凭据，含 DPoP 私钥 | 否，已忽略 |
| `codearts_welfare_ledger.json` | 幂等键、限流计数与一次性福利结清状态 | 否，已忽略 |

- 凭据由环境变量或 GitHub Secrets 注入，仓库不含真实账号材料。
- DPoP 私钥与 refresh_token 同等敏感，请不要粘贴到 Issue。
- 删除本地台账前先确认没有正在进行的领取，否则可能丢失幂等键。

## ❓ 常见问题

**为什么必须回读？**
`claim` / `confirm` 仅说明上游受理；状态变成 `CONFIRMED` 或 `CONSUMED` 才代表真正到账。

**新人注册礼会自动领吗？**
会。它与每日福利共用领取链路，一次性领完并记录在本机；学生认证与邀请好友不自动领取。

**为什么提示本地限流？**
默认每天最多 6 次写尝试，两次至少间隔 10 分钟。需要立即重试时可加 `--force`。

**`AK|SK|STS` 简写能续期吗？**
不能。只有包含 refresh_token、PKCE 与 DPoP 私钥的 JSON 才具备续期条件。

**提示终态或需要重新登录怎么办？**
重新运行 `codearts_login.py` 获取新凭据。一次性 refresh_token 被消费后，本地旧凭据无法恢复。

<a id="release"></a>

## 📦 版本与发布

当前 CHANGELOG 记录最新版本为 `v1.0.6`（2026-11-04）。每个版本单独发布 Release，附完整包、脚本单文件与 `SHA256SUMS.txt`。下载页：[Releases](https://github.com/L0NE-6/CodeArts-Daily/releases)。

## ⚠️ 说明

本项目用于学习与个人自动化。请遵守对应平台的服务条款，并自行承担运行风险。

## ☕ 支持与反馈

脚本完全免费，支持不影响任何功能。

- [提交 Issue](https://github.com/L0NE-6/CodeArts-Daily/issues)：请附运行日志、复现步骤与脱敏后的错误。
- [发起 Pull Request](https://github.com/L0NE-6/CodeArts-Daily/pulls)：欢迎修复、测试与文档贡献。
- [点一个 Star](https://github.com/L0NE-6/CodeArts-Daily)：让更多使用者找到这个项目。

<p align="center">
  <img src="assets/donate-wechat.png" width="240" alt="微信赞赏码" />
  &nbsp;&nbsp;&nbsp;&nbsp;
  <img src="assets/donate-alipay.jpg" width="240" alt="支付宝收款码" />
</p>
<p align="center"><sub>微信支付（左） · 支付宝（右）</sub></p>

> ⚠️ 收款码仅用于向作者自愿赞赏。任何第三方以代下载、代部署、内部资源或会员群为由收费，均与作者无关，请停止付款并保留证据。

<a id="star-trail"></a>

## ✦ 星轨 Star Trail

> 每一次确认到账，都是闭环留下的一个坐标；每一颗 Star，都让这条轨迹更清晰。

<a href="https://star-history.com/#L0NE-6/CodeArts-Daily&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=L0NE-6/CodeArts-Daily&type=Date&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=L0NE-6/CodeArts-Daily&type=Date&theme=light" />
    <img alt="CodeArts Daily Star Trail" src="https://api.star-history.com/svg?repos=L0NE-6/CodeArts-Daily&type=Date&theme=light" />
  </picture>
</a>

---

<div align="center">
  <sub>L0NE-6 · DAILY SIGNAL</sub>
  <br />
  <a href="https://github.com/L0NE-6/CodeArts-Daily/releases">Releases</a>
  ·
  <a href="https://github.com/L0NE-6/CodeArts-Daily/issues">Issues</a>
  ·
  <a href="https://github.com/L0NE-6/CodeArts-Daily">Star</a>
</div>
