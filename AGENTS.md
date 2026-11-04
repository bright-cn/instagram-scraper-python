# 供在此仓库中工作的编码智能体阅读

更改任何内容前，请先阅读本文。以下每一行均已于 2026-09-08 根据实时 API 或已安装的 SDK 验证，其中大部分内容与 SDK 自身的文档字符串相矛盾。

## 身份验证

- SDK 会从环境变量、从其安装目录开始向上搜索找到的 `.env` 文件（因此，当 virtualenv 位于项目内时，就是项目根目录），或从 Bright Data CLI 登录信息中读取 `BRIGHTDATA_API_TOKEN`。设置该值或运行一次 `bdata login`，是唯一需要人工完成的步骤；切勿将密钥粘贴到代码中。
- 始终以 `SyncBrightDataClient(auto_create_zones=False)` 的方式构造客户端。如果保持启用状态，SDK 会在启动时尝试创建网络解锁器和 SERP 区域。此仓库从不使用区域，而且未添加付款方式的账户无法创建区域。

## API 的行为方式

- 每次 SDK 调用都是异步任务：触发、轮询、返回。每个账户预计需要一到三分钟。SDK 不提供同步路径。
- 一次调用中包含多个 URL 仍然只算一个任务。应优先使用这种方式，而不是循环。对于大规模运行，请使用 `posts_trigger`、`posts_status`、`posts_fetch`，并保留快照 ID。
- 每条记录消耗一个积分。评论按每条评论一个积分计费；`reels_all` 会按该账户曾发布的每条 Reels 消耗一个积分。每月免费提供 5,000 个积分。
- 发现 Reels 需要设置 `timeout=420`；默认的 180 秒会超时。
- `post_type` 接受 `"Post"` 或 `"Reels"`。SDK 文档字符串中的 `"Reel"` 会返回错误行，且不返回任何 Reels。
- 空日期范围会返回一个错误行，其消息包含“There are no public posts in the profile for the specified period”。这代表成功，但记录数为零。请匹配消息，而不是 `error_code`；已停用账户也使用相同的代码。
- 数据结构可能随时更改。切勿硬编码字段列表。
- 完整文档索引，每个条目对应一个 `.md` 页面：https://docs.brightdata.com/llms.txt

## 在此仓库中工作

- `pytest` 可离线运行，且不需要令牌。`ruff check .` 必须通过。
- CI 会在空白机器上使用 README 中的命令进行安装。每周工作流会针对真实 API 执行 README 中的每个围栏代码块，每日工作流还会运行规模较小的实时检查。代码片段必须完整且可独立粘贴运行，展示的输出也必须真实。
- README 中的字段表位于 `<!-- fields:start -->` 和 `<!-- fields:end -->` 之间，每天都会重新生成。请勿手动编辑。
- README 顶部的“last verified”徽章行会由每日运行重写。请勿手动编辑。
- 保持项目精简：15 个文件、450 行 Python 代码。请勿添加重试、去重、调度、数据库、异步示例或并发功能。
