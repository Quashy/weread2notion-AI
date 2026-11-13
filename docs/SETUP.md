# 手动部署指南

[返回项目首页](../README.md) · [让 Agent 帮你搭建](AI_SETUP.md)

你需要自己的 GitHub、Notion 和微信读书账号。默认通过 GitHub Actions 运行，无需自建服务器或在电脑安装 Python。

## 1. 复制 Notion 模板

打开 [公共模板](https://titanium-echidna-59b.notion.site/Template-9304aad76d888294b90981a37bbbc42c)，点击 Duplicate，选择自己的工作区并等待复制完成。

保留新副本的完整 URL，后面作为 `NOTION_PAGE`。不要使用公共模板地址，也不要在复制完成前修改数据库名称或属性类型。

## 2. Fork 仓库

在 [Quashy/weread2notion-AI](https://github.com/Quashy/weread2notion-AI) 右上角点击 **Fork**，复制到自己的 GitHub 账号。

后续所有 Secrets 和 Actions 操作都在你自己的 Fork 中完成。Fork 的代码默认公开，Secrets 不会从原仓库继承。

## 3. 获取微信读书 API Key

在 [微信读书官方入口](https://weread.qq.com/r/weread-skills)登录并获取 API Key，稍后直接填入 GitHub Secrets。

不要把 Key 放到 README、Issue 或公开聊天中。

## 4. 创建 Notion 集成

1. 打开 [Notion Integrations](https://www.notion.so/profile/integrations)，创建内部集成。
2. 选择与模板副本相同的工作区，例如命名为 `WeRead Sync`。
3. 启用 `Read content`、`Insert content` 和 `Update content`。
4. 将访问范围限定到刚复制的图书馆根页面及其子内容。可通过集成的 Content access，或页面右上角菜单中的 Connections 配置，以当前界面为准。
5. 将 Integration Secret 保存到下一步的 `NOTION_TOKEN`。

授权根页面，而非只授权其中一个数据库。同步器需要发现书架、统计和设置等子数据库；不需要访问你工作区中的其他内容。

## 5. 保存 GitHub Secrets

在自己的 Fork 打开：

`Settings → Secrets and variables → Actions → New repository secret`

添加以下三个 Repository secrets，名称必须完全一致：

| 名称 | 填写内容 |
| --- | --- |
| `WEREAD_API_KEY` | 微信读书 API Key |
| `NOTION_TOKEN` | Notion Integration Secret |
| `NOTION_PAGE` | 你复制后的 Notion 图书馆根页面完整 URL |

保存后只能查看 Secret 名称，无法再次查看原值。这是正常行为；首次运行会验证配置是否有效。

## 6. 首次同步

1. 在自己的 Fork 打开 **Actions**，按提示启用工作流。
2. 选择 **weread sync**。
3. 点击 **Run workflow**，保持 `full` **未勾选**。
4. 等待本次运行结束，首次导入历史记录可能需要数分钟或更久，期间不要重复触发。
5. 打开运行 Summary 与同步日志，查看同步数量及失败书籍。
6. 刷新 Notion，抽查一本书的进度、时长和章节划线。

部分书读取失败时，程序会保留旧数据并继续处理其他书，任务可能仍显示绿色。查看 Summary 的“需要重试的书籍”，稍后再运行普通同步。

## 7. 定时运行

当前工作流默认每天北京时间 **04:00** 执行，UTC cron 是 `0 20 * * *`，无需电脑常开。需要提前更新时，仍可手动运行普通同步。

GitHub 定时触发可能延迟；公开仓库连续 60 天无活动可能停用定时工作流。请查看自己的 Actions 页面确认状态。[GitHub 官方说明](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule)

Actions 的额度与费用按账号套餐计算。[计费说明](https://docs.github.com/en/billing/concepts/product-billing/github-actions)

## 日常使用与设置

在微信读书里阅读、划线、写想法；在 Notion 里查找、回顾和补充心得。程序按章节生成自动正文，建议把自己的重要心得保存在独立页面，或自动同步标记之外。

模板“设置 → 同步设置”提供：

| 配置 | 默认值 | 用途 |
| --- | --- | --- |
| 阅读完成进度强制改为100% | 关闭 | 已标记读完的书在 Notion 展示为 100% |
| 只同步我的书架书籍 | 开启 | 从微信书架移除的书页在 Notion 中归档 |
| 同步划线和笔记 | 开启 | 将划线和个人想法写入书页正文 |
| 保存阅读快照 | 开启 | 保存每日每本书的状态快照 |
| 阅读统计起始年份 | 2023 | 决定历史统计读取范围 |

“已读”以微信读书的人工读完标记为准。进度达到 100% 不会单独触发“已读”。

如果想把 Notion 当长期资料库，可关闭“只同步我的书架书籍”，保留移出微信书架的历史书页。配置开关使用 Checkbox，年份使用 Number；保留“同步配置版本（不可删除）”等系统属性。

首页布局、视图、筛选、排序可以按喜好调整。程序管理的属性和自动正文则会以微信读书数据更新。

## 同步遇到问题

### Notion 提示找不到页面或没有权限

确认 `NOTION_PAGE` 是自己的模板副本；集成和页面属于同一工作区；集成拥有根页面及子内容的访问权限。

### 微信读书认证或接口版本报错

检查 Secret 名称和 Key 是否被重置。通过安全配置页面更新凭据；若是接口版本变化，先查看项目更新，不通过打印 Key 排查。

### Actions 没有自动运行

确认工作流位于默认分支且已启用；查看任务是否排队、延迟，或账号额度／账单是否阻止运行。不要通过重复点击产生多次并发同步。

### 同步成功，但某个视图没有内容

先检查数据库记录与筛选条件。查看 Summary 是否有失败书籍，再运行普通同步；不要直接选择 full 重建。

### 如何更新自己的版本？

查看原仓库变更后，在自己的 Fork 同步更新；已经有自定义修改时，先在分支中解决冲突和测试。Notion 模板副本不会自动更新，涉及字段结构的升级要单独核对。

## 高级命令与数据安全

本地运行和代码结构见 [技术文档](TECHNICAL.md)。

- `sync --dry-run`：读取微信读书并预览，不写 Notion；不能据此确认 Notion 写入能力。
- `check`：检查模板，但可能补建设置和快照库，不是严格只读。
- `sync --full`：归档并重建记录，适用于明确需要重建的场景。其数据库行 JSON 备份不是完整页面正文备份；操作前另行保护自己的心得，不作为日常排障的默认步骤。
- 导出文件和备份可能包含私人阅读信息，避免提交到公开仓库。

遇到问题可提交 [Issue](https://github.com/Quashy/weread2notion-AI/issues)，附复现步骤和脱敏后的日志。不要附 API Key、Token 或私人笔记全文。
