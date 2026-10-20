# Shopoll 协作者 Codex 工作手册

这份手册用于在 Shopoll 上协作。先把本地开发、代码审查和生产发布分开：
Codex 可以帮你完成阅读、实现、测试、提交和 PR；生产环境始终通过受控的
GitHub Actions 发布。

## 先记住这四件事

1. 当前是 Harbor 的单店 dogfood 应用，不是可直接给任意商家安装的 App Store
   产品。
2. 所有问卷 placement、Theme block、Checkout block、Klaviyo Flow 和 Web Pixel
   默认关闭。没有明确业务确认，不要启用它们。
3. 不要把 `.env`、Railway token、Shopify token、Klaviyo key、订单导出或客户资料
   放进 Git、GitHub Issue、PR、Codex 对话或截图。
4. 每个代码任务使用自己的分支和 PR。不要直接修改 `main`，不要使用
   `git reset --hard` 或覆盖其他人的改动。

## 第一次在电脑上启动

需要 Node.js 22.12 或更高版本、Git 和 Codex。先 clone 仓库并安装依赖：

```powershell
git clone https://github.com/Bean-Harbor/shopoll.git
cd shopoll
npm ci
```

仅查看界面或做前端工作时，不需要 Shopify、PostgreSQL 或生产凭据。运行：

```powershell
pwsh -File scripts/start-demo.ps1
```

然后打开 `http://127.0.0.1:3000/demo`。Demo mode 只提供本地演示界面，不会给
`/app` 或生产 API 绕过认证。

需要真实 Shopify 开发店和本地数据库时，复制 `.env.example` 为 `.env`，只填写
自己的开发凭据。不要使用或索取 Harbor 生产 `.env`。

## 在 Codex 中开始一个任务

在 Codex 打开 clone 下来的 `shopoll` 文件夹。第一次任务先发下面这段：

```text
这是 Shopoll 的 Harbor dogfood 仓库。先阅读 README.md、CONTRIBUTING.md、
SECURITY.md、docs/CODEX-COLLABORATOR-GUIDE.md，以及与任务相关的 docs/ 文件。
先解释你会改哪些文件和风险，再开始编辑。不要读取、创建、输出或提交生产密钥；
不要启用问卷 placement、Theme/Checkout block、Klaviyo Flow 或 Web Pixel。
请在 codex/<简短任务名> 分支工作，保留不相关的现有改动，并在完成前运行相关测试。
```

再追加具体任务。例如：

```text
检查购买动机问卷编辑器的德语翻译完成度。先复现问题，提出最小修复，补测试，
然后创建 draft PR。不要改变已发布 SurveyVersion 的内容或历史回答的快照。
```

```text
为商品页退出意图的触发条件增加一个回归测试。先查看现有 Theme extension 测试，
不要修改生产 placement，也不要扩大到其他触达面。
```

```text
阅读这个 GitHub Issue 的验收条件，定位相关模块，完成最小实现和测试。若任务涉及
订单、奖励、Klaviyo、隐私删除或数据库迁移，先说明数据和上线风险，再等我确认。
```

## 日常开发流程

1. 先同步代码并创建分支：

   ```powershell
   git switch main
   git pull --ff-only
   git switch -c codex/<任务名>
   ```

2. 让 Codex 先阅读相关代码、测试和文档，再做小范围修改。涉及界面时先用
   `/demo` 检查；涉及 API、数据和权限时先检查对应测试。
3. 完成前至少运行与改动相符的检查。普通应用代码通常运行：

   ```powershell
   npm test
   npm run test:extensions
   npm run typecheck
   npm run lint
   npm run build
   ```

   Theme、Checkout 或 Web Pixel 改动应保留 `npm run test:extensions`；客户可见
   流程改动再运行 `npm run test:e2e`。只改文档时，至少让 Codex 检查链接、格式和
   `git diff --check`。
4. 提交前让 Codex 展示 `git status` 和 diff，确认没有 `.env`、构建产物、客户数据
   或他人的不相关改动。
5. 推送分支并创建 **draft PR**。PR 说明要写清楚：改了什么、为什么、测试结果，及
   Shopify/隐私/迁移影响。等 CI 通过和 review 后再合并。

## 哪些事情必须停下来确认

下面任一项出现时，不要让 Codex 自行继续，先在 GitHub Issue 或团队群里说明影响：

- 需要真实 Shopify、Railway、Klaviyo 或 DNS 凭据。
- 需要创建或修改折扣、订单、客户、Klaviyo Profile 或真实问卷回答。
- 需要数据库 migration、删除数据、修改数据保留策略，或处理隐私 webhook。
- 需要发布 Shopify App 版本、变更权限 scopes，或启用任何 customer-facing surface。
- 工作区有不属于当前任务的改动，或者测试失败原因不清楚。

已发布的 `SurveyVersion` 不可编辑。需要改文案、逻辑或翻译时，应让 Codex 创建或
编辑新草稿版本，保留历史回答快照。

## 生产发布

首次基础设施和 Shopify 安装由 Harbor 负责人完成，详细步骤见
[GitHub 一键部署](GITHUB-DEPLOYMENT.md) 与 [Railway 部署](DEPLOYMENT.md)。没有
GitHub `production` Environment 的授权与变量时，不要尝试发布，也不要把凭据贴到
Codex 中。

日常发布在 GitHub 网页完成：**Actions > Deploy Shopoll > Run workflow**。

- `deploy_railway=true` 部署 Web 和 Worker，并检查 `/healthz`、`/readyz`。
- `deploy_shopify=true` 创建 Shopify App 版本和 extensions。
- `release_shopify=false` 是默认安全选项，只创建未发布版本。
- 只有负责人明确批准后才可选择 `release_shopify=true`。

这个工作流不会自动启用问卷、Theme App Embed、Checkout block、Klaviyo Flow 或
Web Pixel。首次安装后仍要确认七个模板均为草稿、所有 placement 为关闭。业务上线
从无奖励购买动机问卷的 10% 流量开始，观察 48 小时后才讨论扩大范围。

## 发生问题时怎么提问

把以下信息贴到 GitHub Issue，不要贴密钥或客户资料：

- 你想完成的结果和验收标准。
- 复现步骤、预期结果和实际结果。
- 页面或接口路径、浏览器/设备、测试环境。
- 已运行的命令和脱敏后的错误输出。
- 是否影响 Shopify、Klaviyo、奖励、订单、隐私或生产发布。

安全问题、疑似泄露、客户数据问题不要开公开 Issue。按
[SECURITY.md](../SECURITY.md) 的私密漏洞流程处理。

## 有用的入口

- [项目概览](../README.md)
- [贡献规则](../CONTRIBUTING.md)
- [发布清单](LAUNCH-CHECKLIST.md)
- [GitHub 一键部署](GITHUB-DEPLOYMENT.md)
- [公开 API 合约](API.md)
