[![使用 Instagram 爬虫工具 API 抓取 Instagram 数据：个人资料、帖子、Reels 和搜索。按 URL 和用户名采集或发现数据。免费开始使用。](.github/banner.png)](https://www.bright.cn/products/web-scraper/instagram?utm_source=github)

# 适用于 Python 的 Instagram 爬虫工具

[![实时检查](https://github.com/bright-cn/instagram-scraper-python/actions/workflows/live.yml/badge.svg)](https://github.com/bright-cn/instagram-scraper-python/actions/workflows/live.yml) [![最后验证时间](https://img.shields.io/badge/last%20verified-4%20Nov%202026-brightgreen)](https://github.com/bright-cn/instagram-scraper-python/actions/workflows/live.yml) <!-- verified: rewritten by the daily run -->

[快速入门](#如何使用-python-抓取-instagram-数据) · [无需登录](#无需登录或访问令牌即可抓取-instagram) · [CLI](#批量-instagram-抓取-cli) · [端点](#抓取-instagram-帖子个人资料reels-和评论) · [数据](#instagram-帖子数据字段json) · [错误](#instagram-抓取工具错误) · [MCP 和 Claude Code](#instagram-mcp-服务器和-claude-code-技能) · [免费积分](#这个-instagram-抓取工具免费吗) · [文档](https://docs.brightdata.com/products/scrapers/instagram/introduction) · [支持](#支持)

使用 Python 将 Instagram 个人资料、帖子、Reels 和评论获取为 JSON。无需登录 Instagram，也无需浏览器。基于 [Bright Data Instagram 爬虫工具 API](https://www.bright.cn/products/web-scraper/instagram?utm_source=github) 构建。

使用 [Bright Data Python SDK](https://github.com/bright-cn/bright-data-sdk-python)。完整 API 文档：[Instagram 爬虫工具 API](https://docs.brightdata.com/products/scrapers/instagram/introduction)。

此外还提供：用于抓取帖子的单命令 CLI，以及完全不需要 Python 的 [Bright Data CLI](#instagram-mcp-服务器和-claude-code-技能)。

## 如何使用 Python 抓取 Instagram 数据

需要 Python 3.10 或更高版本。

```bash
pip install brightdata-sdk
export BRIGHTDATA_API_TOKEN=YOUR_API_KEY
```

从 [Bright Data 控制面板](https://www.bright.cn/cp/setting/users)获取令牌。只要 virtualenv 位于项目内，也可以使用项目根目录中的 `.env` 文件。

也可以跳过令牌。运行一次 `npx -p @brightdata/cli bdata login`：它会打开浏览器，此后 SDK 将自行找到已存储的凭据，供你以及在该终端中工作的任何编码智能体使用。智能体无法点击完成登录，因此请先自行操作。同一个 CLI 也能直接抓取 Instagram；请参阅 [Instagram MCP 服务器和 Claude Code 技能](#instagram-mcp-服务器和-claude-code-技能)。

还没有账户？[创建账户](https://www.bright.cn/cp/start)；新账户[每月可获得 5,000 个免费积分](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    result = client.search.instagram.posts("https://www.instagram.com/nasa/", num_of_posts=5)
    for post in result.data:
        print(post["likes"], post["num_comments"], post["url"])
```

```
2437761 4692 https://www.instagram.com/p/Db9IVmrDvQ4/
104278 902 https://www.instagram.com/reel/DcMXl1IPNtB/
553843 4101 https://www.instagram.com/p/DcOX3hWFiey/
39490 324 https://www.instagram.com/reel/Dc4u1yKPj_s/
119582 1535 https://www.instagram.com/reel/Dc37MPNSAZq/
```

预计需要一到三分钟：API 会运行一个任务，SDK 则等待任务完成。每个帖子消耗一个[积分](https://www.bright.cn/pricing/web-scraper)。

每次都要传入 `auto_create_zones=False`。如果保持启用状态，SDK 会在启动时为网络解锁器和 SERP 创建区域，而本抓取工具从不使用这两种 Bright Data 产品；未添加付款方式的账户会创建区域失败（[sdk-python#57](https://github.com/brightdata/sdk-python/issues/57)）。

## 无需登录或访问令牌即可抓取 Instagram

Instagram 爬虫工具 API 不需要 Instagram 账户、Meta 访问令牌或 Instagram 企业账户。唯一需要的凭据是 Bright Data API 密钥，可将其设置为 `BRIGHTDATA_API_TOKEN`，或通过 `bdata login` 存储。

## 批量 Instagram 抓取 CLI

此仓库中的命令会同时对多个账户运行 Python 快速入门，并写入一个 JSON 文件。

```bash
pip install git+https://github.com/bright-cn/instagram-scraper-python
instagram-scraper nasa natgeo
```

```
Fetching up to 5 recent posts per account, for: nasa, natgeo
Usually one to three minutes each. One credit per post, 5,000 free per month.
asking  @nasa...
got     @nasa: 5 posts
asking  @natgeo...
got     @natgeo: 5 posts

Saved 10 posts as JSON to instagram.json (34 fields per post)
```

在终端中，`asking` 行会被以下内容原地替换，以便你查看运行状态和已经过的时间：

```
⠹ @nasa 0:01:47
```

```
--limit N    posts per account, default 5, minimum 1
--out PATH   output file, default instagram.json
```

`python -m instagram_scraper` 也可以使用。

也可以导入它而不是直接运行，从而为每个账户获取 `ok`、`note` 和 `error`，而不是原始行。单个无效账户永远不会导致 `scrape` 抛出异常；读取 `posts` 前，请先检查 `ok`：

```python
from instagram_scraper import scrape

for outcome in scrape(["nasa", "zz_not_a_real_account_zz"], limit=1):
    if outcome.ok:
        print(f"{outcome.handle}: {len(outcome.posts)} posts")
    else:
        print(f"{outcome.handle} failed: {outcome.error}")
```

```
nasa: 1 posts
zz_not_a_real_account_zz failed: Crawler error: Cannot read properties of null (reading 'pk')
```

账户最近没有内容时，仍视为成功，只是不返回帖子。原因位于 `note`，而不是 `error`。

## 抓取 Instagram 帖子、个人资料、Reels 和评论

该命令涵盖一个端点。SDK 提供八个端点，详见 [Instagram 爬虫工具 API 文档](https://docs.brightdata.com/products/scrapers/instagram/introduction)。以下每个代码片段都是完整的，且只需要 `brightdata-sdk`：可直接原样粘贴。所有片段都会在每周一通过 Actions 运行，规模较小的检查则每隔一天运行一次，顶部徽章显示最新结果。

| 你拥有 | 你想要 | 调用 |
| --- | --- | --- |
| 个人资料 URL | 近期帖子 | `client.search.instagram.posts(url, num_of_posts=5)` |
| 用户名 | 个人资料 | `client.search.instagram.profiles("nasa")` |
| 个人资料 URL | 个人资料 | `client.scrape.instagram.profiles(url)` |
| 个人资料 URL | 近期 Reels | `client.search.instagram.reels(url, num_of_posts=5, timeout=420)` |
| 个人资料 URL | 所有历史 Reels | `client.search.instagram.reels_all(url)`，该账户发布的每条 Reels 消耗一个积分 |
| 帖子 URL | 这些帖子 | `client.scrape.instagram.posts([url, url])` |
| Reels URL | 这些 Reels | `client.scrape.instagram.reels([url])` |
| 帖子或 Reels URL | 其评论 | `client.scrape.instagram.comments(url)`，每条评论消耗一个积分 |

上述每项调用都是异步任务。SDK 会触发任务、进行轮询，并在任务就绪后返回。这就是一次调用需要一到三分钟，以及 Python 中没有更快路径的原因。API 的[同步端点](https://docs.brightdata.com/api-reference/scrapers/synchronous-requests)仅支持原始 HTTP，限制为 20 个 URL 和一分钟。

之所以会出现上述空时间范围等错误行，是因为 SDK 使用 `include_errors=true` 请求返回这些错误。API 默认不返回。

`search` 调用会在 `result.data` 中返回列表。使用一个 URL 的 `scrape` 调用会以字典形式返回一条记录，使用多个 URL 时则返回列表。

### 在一个任务中抓取多个 Instagram 账户

URL 列表只会创建一个任务，而不是每个账户一个任务。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    result = client.search.instagram.posts(
        ["https://www.instagram.com/nasa/", "https://www.instagram.com/natgeo/"], num_of_posts=1
    )
    for post in result.data:
        print(post["user_posted"], post["url"])
```

```
nasa https://www.instagram.com/p/DcOX3hWFiey/
natgeo https://www.instagram.com/reel/Dbru79IAdH-/
```

### 立即触发，稍后获取

对于超过少量账户的任务，不要让进程阻塞一小时。触发任务、保存快照 ID，并在就绪后获取。快照可在 30 天内下载。

```python
import time

from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    job = client.scrape.instagram.posts_trigger("https://www.instagram.com/p/Dc1W1uFj-CW/")
    print("snapshot:", job.snapshot_id)
    while (status := client.scrape.instagram.posts_status(job.snapshot_id)) not in ("ready", "failed"):
        time.sleep(5)
    print("status:", status)
    record = client.scrape.instagram.posts_fetch(job.snapshot_id)[0]
    print("fetched:", record["url"], "likes:", record["likes"])
```

```
snapshot: sd_mtrapflrzynbfyid9
status: ready
fetched: https://www.instagram.com/p/Dc1W1uFj-CW/ likes: 184
```

### 按日期筛选 Instagram 帖子

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    result = client.search.instagram.posts(
        "https://www.instagram.com/nasa/",
        num_of_posts=3,
        start_date="08-01-2026",   # MM-DD-YYYY
        end_date="09-07-2026",
    )
    for post in result.data:
        print(post["date_posted"], post["content_type"], post["url"])
```

```
2026-08-12T21:28:58.000Z Image https://www.instagram.com/p/Db9IVmrDvQ4/
2026-08-18T19:37:40.000Z Reel https://www.instagram.com/reel/DcMXl1IPNtB/
2026-08-19T14:11:47.000Z Image https://www.instagram.com/p/DcOX3hWFiey/
```

`post_type="Post"` 仅保留帖子，`post_type="Reels"` 仅保留 Reels，两者均已验证。SDK 的文档字符串中写的是 `"Reel"`；这种拼写会返回错误行，且不返回任何 Reels（[sdk-python#59](https://github.com/brightdata/sdk-python/issues/59)）。`posts_to_not_include` 接受帖子 ID 列表。

### 按用户名抓取 Instagram 个人资料

无需 URL。返回的记录包含该账户的关注者数量和帖子数量。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    profile = client.search.instagram.profiles("nasa").data[0]
    print(profile["account"], "followers:", profile["followers"], "posts:", profile["posts_count"])
```

```
nasa followers: 104387874 posts: 4913
```

### 抓取 Instagram 帖子或 Reels 的评论

每条评论消耗一个积分，因此请先检查帖子上的 `num_comments`。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    comments = client.scrape.instagram.comments("https://www.instagram.com/p/Dc1W1uFj-CW/").data
    print(len(comments), "comments; first:", repr(comments[0]["comment"][:60]))
```

```
6 comments; first: 'Those two destroyed jazz'
```

### 从个人资料抓取 Instagram Reels

发现 Reels 的速度较慢。默认的 180 秒超时会过期；420 秒则不会。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    reels = client.search.instagram.reels("https://www.instagram.com/nasa/", num_of_posts=2, timeout=420)
    for reel in reels.data:
        print(reel["date_posted"], reel["url"])
```

```
2026-08-18T19:37:40.000Z https://www.instagram.com/p/DcMXl1IPNtB/
2026-09-05T01:00:10.000Z https://www.instagram.com/p/Dc4u1yKPj_s/
```

## Instagram 帖子数据字段（JSON）

大多数人需要的字段：

```
url  date_posted  description  hashtags  likes  num_comments  user_posted
```

代码不会硬编码任何字段列表。API 返回的所有内容都会写入 `result.data`，以及命令生成的文件中。协作帖子会在 `user_posted` 中包含共同作者的用户名，因此从 nasa 获取的帖子也可能显示 `nasajohnson`；`coauthor_producers` 会列出所有人。

<!-- fields:start -->
<details>
<summary>全部 44 个字段，包括类型和说明</summary>

每天通过 `client.datasets.instagram_posts.get_metadata()` 根据数据集结构重新生成，因此不会过时。帖子会包含适用于它的字段；示例文件包含这 44 个字段中的 34 个。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `url` | url | Instagram 帖子的直接 URL |
| `user_posted` | 文本 | 帖子创建者的用户名 |
| `description` | 文本 | 帖子文本说明 |
| `hashtags` | 数组 | 帖子中使用的标签 |
| `num_comments` | 数字 | 评论数量 |
| `date_posted` | 日期 | 帖子发布日期 |
| `likes` | 数字 | 帖子的点赞数 |
| `photos` | 数组 | 所附照片的 URL；由于 Instagram 政策，这些 URL 可能已过期 |
| `videos` | 数组 | 所附视频的 URL；由于 Instagram 政策，这些 URL 可能已过期 |
| `location` | 数组 | 与帖子关联的地理位置 |
| `location_details` | 对象 | Instagram 返回的详细地理位置元数据 |
| `latest_comments` | 数组 | 帖子的最新评论 |
| `post_id` | 文本 | 唯一帖子标识符 |
| `discovery_input` | 对象 | 用于触发采集的发现输入值 |
| `has_handshake` | 布尔值 | 表示帖子是否具有握手标记（账户之间的协作协议） |
| `display_url` | 文本 | 已弃用：以前用作帖子媒体的显示 URL |
| `shortcode` | 文本 | Instagram 帖子的短代码，用于帖子 URL 路径 |
| `content_type` | 文本 | 内容类型：帖子或 Reels |
| `pk` | 文本 | Instagram 分配给媒体内容的主键 |
| `content_id` | 文本 | 媒体项的内容 ID |
| `engagement_score_view` | 数字 | 用作互动评分指标的视频观看次数 |
| `thumbnail` | 文本 | 帖子显示图片或视频缩略图的 URL |
| `video_view_count` | 文本 | 视频帖子的观看次数 |
| `product_type` | 文本 | 与帖子关联的产品类型，例如 Reels 对应的“clips” |
| `coauthor_producers` | 数组 | 参与帖子协作的共同作者或制作人列表 |
| `tagged_users` | 数组 | 帖子中标记的用户列表 |
| `video_play_count` | 数字 | 视频的播放次数 |
| `followers` | 数字 | 采集时帖子所有者的关注者数量 |
| `posts_count` | 数字 | 采集时该账户发布的帖子总数 |
| `profile_image_link` | 文本 | 直接链接到帖子所有者 Instagram 头像的 URL |
| `is_verified` | 布尔值 | 表示帖子所有者的账户是否已认证 |
| `is_paid_partnership` | 布尔值 | 表示帖子是否为赞助内容或付费合作内容 |
| `partnership_details` | 对象 | 与帖子关联的付费合作品牌详情 |
| `user_posted_id` | 文本 | 发布该帖子的账户所对应的 Instagram 用户 ID |
| `post_content` | 数组 | 帖子所附的媒体项（照片或视频）列表，包括轮播项 |
| `audio` | 对象 | 与帖子或 Reels 关联的音轨元数据 |
| `profile_url` | url | 发布该帖子的 Instagram 个人资料 URL |
| `videos_duration` | 数组 | 帖子中每个所附视频的时长列表 |
| `images` | 数组 | 帖子所附的图片对象列表 |
| `alt_text` | 文本 | 帖子主图的无障碍替代文本：向盲人或视障用户传达图片含义的描述性文本 |
| `photos_number` | 数字 | 帖子所附的照片总数 |
| `audio_url` | url | 帖子所用音轨的直接 URL |
| `thumbnail_array` | 数组 | 已弃用：帖子媒体缩略图 URL 的数组 |
| `country` | 文本 | 部分个人资料受地区限制。请使用 Alpha-2 格式设置国家/地区代码 |

</details>
<!-- fields:end -->

<details>
<summary>来自 <code>instagram-scraper nasa --limit 1</code> 的真实输出文件开头</summary>

```json
{
  "generated_at": "2026-08-14T11:01:05.410349+00:00",
  "handles": [
    {
      "handle": "nasa",
      "posts": [
        {
          "url": "https://www.instagram.com/p/Db_SePSltfz/",
          "user_posted": "nasa",
          "description": "A dance of darkness and light\n\nThis composite image shows the total solar eclipse from beginning to end, as seen in San Mill\u00e1n de los Caballeros, Spain on Aug. 12, 2026.\n\nWhile eclipses are beautiful and magical to experience, they're also a great opportunity for science. Total solar eclipses are particularly important because they allow scientists to see a part of the Sun\u2019s atmosphere known as the corona. The corona is too faint to see except when the bright light of the Sun is blocked.\n\nStudying the corona is key to answering important questions about how heat and energy are transferred from the Sun out into the solar wind, the constant stream of particles that the Sun spews into the solar system.\n\nCredit: NASA\n\n#NASA #Sun #SolarEclipse #Moon",
          "hashtags": [
            "#NASA",
            "#Sun",
            "#SolarEclipse",
            "#Moon"
          ],
          "num_comments": 1109,
          "date_posted": "2026-08-13T17:36:02.000Z",
          "likes": 627573,
          "photos": [
  ...
```

包含一条完整字段帖子的整个文件位于 [examples/sample_output.json](examples/sample_output.json)。

</details>

## Instagram 抓取工具错误

| 你看到的内容 | 含义 |
| --- | --- |
| `API token required but not found.` | 退出码 2，尚未发送任何请求。请设置令牌。 |
| `failed  @name: ...` | 退出码 1。不存在该账户，通常是拼写错误。具体措辞可能不同：“Sorry, this page isn't available.”和“Crawler error: Cannot read properties of null”都表示这种情况。 |
| `got     @name: 0 posts, the account has no public posts in the period searched` | 退出码 0，且结果正确。API 会将空时间范围报告为错误行。 |
| `failed  @name: timeout` | 退出码 1。请求在 180 秒后放弃。请重新运行。 |

任何失败都会以退出码 1 结束，因此可以安全地使用运行结果作为脚本的执行条件。

在 SDK 中，相同情况如下所示：

| 你看到的内容 | 含义 |
| --- | --- |
| `AuthenticationError: Unauthorized (401)` | 令牌已设置，但不正确。 |
| `result.success` 为 `False`，`result.status` 为 `"timeout"` | SDK 放弃等待，默认超时时间为 180 秒。请传入 `timeout=420`，或重新运行。 |
| `result.data` 中含有 `error` 键的一行 | API 对单个输入的响应：时间范围为空，或不存在该账户。其他行均正常。 |

## Instagram MCP 服务器和 Claude Code 技能

无需 Python，也无需安装任何内容。粘贴以下两行；第一行会打开一次浏览器，也可以通过 SSH 和 CI 使用 `bdata login --device`：

```bash
npx -p @brightdata/cli bdata login
npx -p @brightdata/cli bdata pipelines run gd_lk5ns7kz21pck8jpis collect_by_url "https://www.instagram.com/p/Db_SePSltfz/"
```

CLI 按数据集 ID 运行爬虫工具。四个 Instagram 爬虫分别接受帖子、Reels 或个人资料 URL，并返回一条对应记录；`bdata pipelines search instagram` 会列出它们的 ID 和方法。

`npx skills add brightdata/skills` 会让 Claude Code、Cursor 和 Codex 掌握这些命令和文档，此后即可使用自然语言操作。完整指南：[面向编码智能体的 Bright Data](https://docs.brightdata.com/quickstart-coding-agent)。

为托管式助手使用，不方便打开终端？[Bright Data MCP 服务器](https://github.com/bright-cn/brightdata-mcp#which-tool-to-use)提供相同的四个 Instagram 工具，每个工具接受一个 URL；它们位于 `social` 组中，除非你主动请求，否则该组默认关闭：

    https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN&groups=social

智能体也可以自行开通账户，无需填写注册表单：[智能体注册](https://brightdata.com/auth.md)。Bright Data 支持的所有其他连接，从 LangChain 到 Zapier 和 n8n：[集成](https://docs.brightdata.com/integrations/introduction)。

<details>
<summary>CLI 提供的所有 Instagram 命令，均于今日运行</summary>

四个爬虫工具，每个都使用 `bdata pipelines run <dataset_id> collect_by_url` 运行：个人资料 `gd_l1vikfch901nx3by4`、帖子 `gd_lk5ns7kz21pck8jpis`、Reels `gd_lyclm20il4r5helnj` 和评论 `gd_ltppn085pokosxh13`。每个工具都接受 URL、输出 JSON，并按每条记录消耗一个积分。请在每条命令前加上 `npx -p @brightdata/cli`，或使用 `npm i -g @brightdata/cli` 安装一次。

```bash
bdata pipelines run gd_l1vikfch901nx3by4 collect_by_url "https://www.instagram.com/nasa/" --pretty
```

1 条记录、25 个字段、50 秒。`"account": "nasa"`、`"followers": 104396028`。

```bash
bdata pipelines run gd_lk5ns7kz21pck8jpis collect_by_url "https://www.instagram.com/p/Dc1W1uFj-CW/" --format csv -o posts.csv
```

37 秒。CSV 表头包含与下表相同的字段名称。

```bash
bdata pipelines run gd_lyclm20il4r5helnj collect_by_url "https://www.instagram.com/reel/DcMXl1IPNtB/" --pretty
```

1 条记录、25 个字段、23 秒。

```bash
bdata pipelines run gd_ltppn085pokosxh13 collect_by_url "https://www.instagram.com/p/Dc1W1uFj-CW/" --pretty
```

5 条记录、11 个字段、9 秒。每条评论消耗一个积分，因此请先检查帖子的 `num_comments`。

`bdata pipelines info <dataset_id> <method>` 显示爬虫工具的输入字段。`bdata pipelines run --help` 显示 `--format json|csv|ndjson|jsonl`、`-o FILE`、`--timeout` 和 `--async`。CLI 0.4.0 已使用这些命令替代旧的 `bdata pipelines instagram_posts <url>` 形式。

</details>

## 这个 Instagram 抓取工具免费吗？

新的 Bright Data 账户[每月可获得 5,000 个免费积分](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。帖子、Reels 和个人资料按每条记录一个积分计费，评论按每条评论一个积分计费，因此免费积分每月可覆盖 5,000 个帖子。后续费率请参阅[定价](https://www.bright.cn/pricing/web-scraper)。

## 支持

此仓库中的错误：[提交 issue](https://github.com/bright-cn/instagram-scraper-python/issues)，[CONTRIBUTING.md](CONTRIBUTING.md) 说明了需要提供哪些信息。与 API、你的账户或积分相关的任何问题，请联系 [Bright Data 支持团队](https://brightdata.zendesk.com/hc/en-us/requests/new)。

## 许可证

[Apache 2.0](LICENSE)。

## 适用于 Node.js 的 Instagram 爬虫工具

同一款 Instagram 抓取工具也提供 Node.js 版本：[bright-cn/instagram-scraper-node](https://github.com/bright-cn/instagram-scraper-node)。
