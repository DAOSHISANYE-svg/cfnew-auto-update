# CFnew 自动更新仓库

CFnew 部署到 Cloudflare Pages，配合 GitHub Actions 实现**云端自动更新** —— 不依赖本机开机。

## 为什么需要它

本机不常开 → 本地定时任务不可靠 → 必须放在 GitHub Actions 云端执行。

## 工作流程

| 步骤 | 说明 |
|---|---|
| 1. 定时触发 | 每天 `04:17 UTC`（北京时间 12:17），错开整点避开 GitHub 拥堵 |
| 2. 查版本 | GitHub API 读 `byJoey/cfnew` 最新 Release tag |
| 3. 比对 | 与 `deployed-version.txt` 比，不同才继续 |
| 4. 下载 | 拉最新版本的 `Pages.zip` 并解压 |
| 5. 部署 | `wrangler pages deploy` 到 Cloudflare Pages |
| 6. 冒烟验证 | 主地址 + 家宽订阅端点各测一次 HTTP，非 200 即报错 |
| 7. 记录 | 更新 `deployed-version.txt` 并自动提交 |

## 需要你配置的东西（一次性）

### A. Cloudflare API Token

1. 打开 https://dash.cloudflare.com/profile/api-tokens
2. **Create Token** → 模板选 **Edit Cloudflare Workers**
3. 或用 **Create Custom Token**：
   - Permission：`Account / Cloudflare Pages / Edit`
   - Account Resources：选 `Junjiang412@gmail.com's Account`
4. 创建后复制 Token 字符串

### B. 在 GitHub 仓库添加 Secrets

仓库 → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

| 名称 | 值 |
|---|---|
| `CLOUDFLARE_API_TOKEN` | 上一步复制的 Token |
| `CLOUDFLARE_ACCOUNT_ID` | `071c36c227211679dd1537626856116f` |

### C. 在 GitHub 仓库添加 Variables

同一页面切到 **Variables** tab → **New repository variable**

| 名称 | 值 |
|---|---|
| `CFNEW_UUID` | `e33d26e6-3e16-41a5-88a6-f8a95d2e389d` |

## 手动触发

Actions 页面 → **CFnew Auto Update** → **Run workflow**
- 直接点 → 只在有新版时部署
- 输入 `force=true` → 强制重新部署（忽略版本检查）

## Cloudflare Pages 项目原有配置（**不要改**）

```
compatibility_date = 2026-01-20       ← 不设则节点全废
env u    = e33d26e6-3e16-41a5-88a6-f8a95d2e389d
env jk   = yes                          ← 家宽链式开关
env qj   = only                         ← 防出口IP泄漏（不回落CF机房IP）
KV[KV]   = 2f10ba8ca0364335bbc4f9b8d93b0dff
```

## 失败排查

| 现象 | 原因 |
|---|---|
| 订阅返回 403 | `jk` 环境变量丢了 → 去 CF 控制台检查 |
| 订阅返回 1101 | 未绑自定义域 / Pages 项目异常 |
| 节点全不可用 | 兼容日期被改成非 `2026-01-20` |
| Action 报 auth 失败 | `CLOUDFLARE_API_TOKEN` 无效或过期 |

## 相关链接

- CFnew 仓库：https://github.com/byJoey/cfnew
- 线上地址：https://cfnew-do9.pages.dev