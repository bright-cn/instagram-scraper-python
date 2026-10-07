[![使用 Instagram 爬虫 API 抓取 Instagram 数据：个人资料、帖子、Reels 短视频和搜索结果。可按 URL 或用户名采集或发现数据。免费开始使用。](.github/banner.png)](https://www.bright.cn/products/web-scraper/instagram?utm_source=github)

# instagram-scraper-python

[快速开始](#快速开始) · [命令行使用](#作为命令运行) · [API 接口](#其他-api-接口) · [数据](#数据) · [错误处理](#出错时) · [编程智能体](#编程智能体) · [文档](https://docs.brightdata.com/products/scrapers/instagram/introduction) · [支持](#支持)

使用 Python 将 Instagram 个人资料、帖子、Reels 短视频和评论获取为 JSON。无需登录 Instagram，也无需浏览器。基于 [Bright Data Instagram 爬虫 API](https://www.bright.cn/products/web-scraper/instagram?utm_source=github) 构建。

本项目使用 [Bright Data Python SDK](https://github.com/bright-cn/sdk-python)。完整 API 文档：[Instagram 爬虫 API](https://docs.brightdata.com/products/scrapers/instagram/introduction)。

本仓库还提供用于获取帖子的单命令 CLI，以及完全无需编写 Python 的 [Bright Data CLI](#编程智能体)。

## 快速开始

需要 Python 3.10 或更新版本。

```bash
pip install brightdata-sdk
export BRIGHTDATA_API_TOKEN=YOUR_API_KEY
```

从 [Bright Data 控制面板](https://www.bright.cn/cp/setting/users)获取令牌。如果虚拟环境位于项目内，也可以将 `.env` 文件放在项目根目录。

也可以不手动设置令牌。先运行一次 `npx -p @brightdata/cli bdata login`：它会打开浏览器。此后，SDK 会自行找到已保存的凭据，供你以及在该终端工作的任何编程智能体使用。智能体无法自行完成浏览器中的登录操作，因此请先亲自登录。同一个 CLI 也能直接抓取 Instagram 数据；详见[编程智能体](#编程智能体)。

还没有账户？[创建账户](https://www.bright.cn/cp/start)；新账户每月可获得 [5,000 免费积分](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    result = client.search.instagram.posts("https://www.instagram.com/nasa/", num_of_posts=5)
    for post in result.data:
        print(post["likes"], post["num_comments"], post["url"])
```

```text
2437761 4692 https://www.instagram.com/p/Db9IVmrDvQ4/
104278 902 https://www.instagram.com/reel/DcMXl1IPNtB/
553843 4101 https://www.instagram.com/p/DcOX3hWFiey/
39490 324 https://www.instagram.com/reel/Dc4u1yKPj_s/
119582 1535 https://www.instagram.com/reel/Dc37MPNSAZq/
```

预计需要一至三分钟：API 会运行一个任务，SDK 则会等待任务完成。每条帖子消耗 [1 个积分](https://www.bright.cn/pricing/web-scraper)。

每次都要传入 `auto_create_zones=False`。如果保留默认设置，SDK 会在启动时为网络解锁器和搜索引擎 API 创建区域；这两款 Bright Data 产品与此爬虫工具无关，而且未添加付款方式的账户无法创建区域（[sdk-python#57](https://github.com/bright-cn/sdk-python/issues/57)）。

## 作为命令运行

此仓库中的命令可以处理多个账户，并将结果写入一个 JSON 文件。

```bash
pip install git+https://github.com/bright-cn/instagram-scraper-python
instagram-scraper nasa natgeo
```

```text
Fetching up to 5 recent posts per account, for: nasa, natgeo
Usually one to three minutes each. One credit per post, 5,000 free per month.
asking  @nasa...
got     @nasa: 5 posts
asking  @natgeo...
got     @natgeo: 5 posts

Saved 10 posts as JSON to instagram.json (34 fields per post)
```

在终端中，`asking` 行会被下面的进度显示替换，并原地更新，让你知道任务仍在运行以及已经运行了多久：

```text
⠹ @nasa 0:01:47
```

```text
--limit N    posts per account, default 5, minimum 1
--out PATH   output file, default instagram.json
```

也可以运行 `python -m instagram_scraper`。

还可以导入它，而不是将它作为命令运行。这样，每个账户都会得到 `ok`、`note` 和 `error` 状态，而不只是原始数据行。单个账户出错不会使 `scrape` 抛出异常；读取 `posts` 前请先检查 `ok`：

```python
from instagram_scraper import scrape

for outcome in scrape(["nasa", "zz_not_a_real_account_zz"], limit=1):
    if outcome.ok:
        print(f"{outcome.handle}: {len(outcome.posts)} posts")
    else:
        print(f"{outcome.handle} failed: {outcome.error}")
```

```text
nasa: 1 posts
zz_not_a_real_account_zz failed: Crawler error: Cannot read properties of null (reading 'pk')
```

如果账户近期没有帖子，调用仍算成功，只是返回零条帖子。原因会写入 `note`，而不是 `error`。

## 其他 API 接口

上述命令使用其中一个接口。SDK 共提供八个接口，详见 [Instagram 爬虫 API 文档](https://docs.brightdata.com/products/scrapers/instagram/introduction)。下面的每段代码都是完整示例，只需要 `brightdata-sdk`，可直接粘贴运行。所有示例每周一都会在 Actions 中运行，其他日子还会执行规模较小的检查。页面顶部的徽章显示最近一次结果。

| 已有信息 | 想获取 | 调用方式 |
| --- | --- | --- |
| 个人资料 URL | 近期帖子 | `client.search.instagram.posts(url, num_of_posts=5)` |
| 用户名 | 对应个人资料 | `client.search.instagram.profiles("nasa")` |
| 个人资料 URL | 对应个人资料 | `client.scrape.instagram.profiles(url)` |
| 个人资料 URL | 近期 Reels 短视频 | `client.search.instagram.reels(url, num_of_posts=5, timeout=420)` |
| 个人资料 URL | 该账户发布过的所有 Reels 短视频 | `client.search.instagram.reels_all(url)`；每条 Reels 短视频消耗 1 个积分 |
| 帖子 URL | 对应帖子 | `client.scrape.instagram.posts([url, url])` |
| Reels 短视频 URL | 对应 Reels 短视频 | `client.scrape.instagram.reels([url])` |
| 帖子或 Reels 短视频 URL | 对应评论 | `client.scrape.instagram.comments(url)`；每条评论消耗 1 个积分 |

这些调用都会创建异步任务：SDK 触发任务、轮询状态，并在任务就绪后返回。这就是一次调用需要一至三分钟，以及 Python 中没有更快调用方式的原因。API 的[同步接口](https://docs.brightdata.com/api-reference/scrapers/synchronous-requests)仅供直接通过 HTTP 调用，最多接受 20 个 URL，且有一分钟的时间限制。

上文所示的错误行之所以会出现，是因为 SDK 使用 `include_errors=true` 请求返回错误。API 默认不会返回这些错误行。

`search` 调用在 `result.data` 中返回列表。`scrape` 调用在传入单个 URL 时返回一条字典形式的记录，传入多个 URL 时返回列表。

### 两个账户，一个任务

一个 URL 列表只会创建一个任务，不会为每个账户各建一个任务。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    result = client.search.instagram.posts(
        ["https://www.instagram.com/nasa/", "https://www.instagram.com/natgeo/"], num_of_posts=1
    )
    for post in result.data:
        print(post["user_posted"], post["url"])
```

```text
nasa https://www.instagram.com/p/DcOX3hWFiey/
natgeo https://www.instagram.com/reel/Dbru79IAdH-/
```

### 现在触发，稍后获取

如果要处理的不止几个账户，不要让进程阻塞一小时。先触发任务并保存快照 ID，待任务就绪后再获取结果。快照可在 30 天内下载。

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

```text
snapshot: sd_mtrapflrzynbfyid9
status: ready
fetched: https://www.instagram.com/p/Dc1W1uFj-CW/ likes: 184
```

### 指定日期范围

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

```text
2026-08-12T21:28:58.000Z Image https://www.instagram.com/p/Db9IVmrDvQ4/
2026-08-18T19:37:40.000Z Reel https://www.instagram.com/reel/DcMXl1IPNtB/
2026-08-19T14:11:47.000Z Image https://www.instagram.com/p/DcOX3hWFiey/
```

`post_type="Post"` 只保留普通帖子，`post_type="Reels"` 只保留 Reels 短视频；两者均已验证。SDK 的文档字符串写的是 `"Reel"`，但使用这个拼法会返回错误行，而不会返回 Reels 短视频（[sdk-python#59](https://github.com/bright-cn/sdk-python/issues/59)）。`posts_to_not_include` 接受帖子 ID 列表。

### 通过用户名获取个人资料，无需 URL

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    profile = client.search.instagram.profiles("nasa").data[0]
    print(profile["account"], "followers:", profile["followers"], "posts:", profile["posts_count"])
```

```text
nasa followers: 104387874 posts: 4913
```

### 获取帖子的评论

每条评论消耗 1 个积分，因此请先查看帖子的 `num_comments`。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    comments = client.scrape.instagram.comments("https://www.instagram.com/p/Dc1W1uFj-CW/").data
    print(len(comments), "comments; first:", repr(comments[0]["comment"][:60]))
```

```text
6 comments; first: 'Those two destroyed jazz'
```

### 获取近期 Reels 短视频

发现 Reels 短视频所需时间更长。默认的 180 秒超时未能等到结果；设为 420 秒则成功返回。

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient(auto_create_zones=False) as client:
    reels = client.search.instagram.reels("https://www.instagram.com/nasa/", num_of_posts=2, timeout=420)
    for reel in reels.data:
        print(reel["date_posted"], reel["url"])
```

```text
2026-08-18T19:37:40.000Z https://www.instagram.com/p/DcMXl1IPNtB/
2026-09-05T01:00:10.000Z https://www.instagram.com/p/Dc4u1yKPj_s/
```

## 数据

大多数人关心的字段：

```text
url  date_posted  description  hashtags  likes  num_comments  user_posted
```

代码没有硬编码字段列表。API 返回的所有字段都会进入 `result.data`，也会写入命令生成的文件。

如果帖子由多个账户合作发布，`user_posted` 中可能是合作作者的用户名。因此，从 `nasa` 获取的帖子也可能显示 `nasajohnson`；`coauthor_producers` 会列出所有合作作者。

<!-- fields:start -->
<details>
<summary>全部 44 个字段及其类型和说明</summary>

以下内容每天都会通过 `client.datasets.instagram_posts.get_metadata()` 从数据集的数据结构重新生成，因此不会过时。每条帖子只包含适用于它的字段；示例文件包含这 44 个字段中的 34 个。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `url` | URL | Instagram 帖子的直接 URL |
| `user_posted` | 文本 | 帖子发布者的用户名 |
| `description` | 文本 | 帖子的文字说明 |
| `hashtags` | 数组 | 帖子使用的话题标签 |
| `num_comments` | 数字 | 评论数量 |
| `date_posted` | 日期 | 帖子发布日期 |
| `likes` | 数字 | 帖子获赞数量 |
| `photos` | 数组 | 所附图片的 URL；根据 Instagram 政策，这些 URL 可能过期 |
| `videos` | 数组 | 所附视频的 URL；根据 Instagram 政策，这些 URL 可能过期 |
| `location` | 数组 | 与帖子关联的地理位置 |
| `location_details` | 对象 | Instagram 返回的详细地理位置元数据 |
| `latest_comments` | 数组 | 帖子的近期评论 |
| `post_id` | 文本 | 帖子的唯一标识符 |
| `discovery_input` | 对象 | 用于触发数据采集的发现类输入值 |
| `has_handshake` | 布尔值 | 帖子是否带有账户之间的合作发布标识 |
| `display_url` | 文本 | 已弃用：此前用于表示帖子媒体的展示 URL |
| `shortcode` | 文本 | Instagram 帖子的短代码，用于帖子 URL 路径 |
| `content_type` | 文本 | 内容类型：普通帖子或 Reels 短视频 |
| `pk` | 文本 | Instagram 为媒体内容分配的主键 |
| `content_id` | 文本 | 媒体项目的内容 ID |
| `engagement_score_view` | 数字 | 用作互动评分指标的视频观看次数 |
| `thumbnail` | 文本 | 帖子展示图片或视频缩略图的 URL |
| `video_view_count` | 文本 | 视频帖子的观看次数 |
| `product_type` | 文本 | 与帖子关联的产品类型，例如 Reels 短视频对应的 `clips` |
| `coauthor_producers` | 数组 | 参与合作发布帖子的作者或制作人列表 |
| `tagged_users` | 数组 | 帖子中标记的用户列表 |
| `video_play_count` | 数字 | 视频播放次数 |
| `followers` | 数字 | 数据采集时，帖子发布者的粉丝数量 |
| `posts_count` | 数字 | 数据采集时，该账户发布的帖子总数 |
| `profile_image_link` | 文本 | 直达帖子发布者 Instagram 头像的 URL |
| `is_verified` | 布尔值 | 帖子发布者的账户是否已获得认证 |
| `is_paid_partnership` | 布尔值 | 帖子是否属于赞助内容或付费合作 |
| `partnership_details` | 对象 | 与帖子关联的付费合作品牌详情 |
| `user_posted_id` | 文本 | 帖子发布账户的 Instagram 用户 ID |
| `post_content` | 数组 | 帖子所附媒体项目（图片或视频）的列表，包括轮播内容 |
| `audio` | 对象 | 与帖子或 Reels 短视频关联的音轨元数据 |
| `profile_url` | URL | 帖子发布者的 Instagram 个人资料 URL |
| `videos_duration` | 数组 | 帖子所附各个视频的时长列表 |
| `images` | 数组 | 帖子所附图片对象的列表 |
| `alt_text` | 文本 | 帖子主图的无障碍替代文本：用于向盲人或视障用户描述图片含义 |
| `photos_number` | 数字 | 帖子所附图片总数 |
| `audio_url` | URL | 帖子所用音轨的直接 URL |
| `thumbnail_array` | 数组 | 已弃用：帖子媒体缩略图 URL 的数组 |
| `country` | 文本 | 某些个人资料受到地区限制。请使用 Alpha-2 格式设置国家或地区代码 |

</details>
<!-- fields:end -->

<details>
<summary>真实输出文件的开头，来自 <code>instagram-scraper nasa --limit 1</code></summary>

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

包含一条帖子及其全部字段的完整文件见 [examples/sample_output.json](examples/sample_output.json)。

</details>

## 出错时

| 看到的信息 | 含义 |
| --- | --- |
| `API token required but not found.` | 在发送任何请求之前以状态码 2 退出。请设置令牌。 |
| `failed  @name: ...` | 以状态码 1 退出。账户不存在，通常是名称输入有误。提示可能有所不同：`Sorry, this page isn't available.` 和 `Crawler error: Cannot read properties of null` 都可能表示这种情况。 |
| `got     @name: 0 posts, the account has no public posts in the period searched` | 以状态码 0 退出，属于正确结果。API 将指定时间范围内没有帖子报告为错误行。 |
| `failed  @name: timeout` | 以状态码 1 退出。请求在等待 180 秒后放弃。请重新运行。 |

任何失败都会以状态码 1 退出，因此脚本可以安全地依据运行结果决定是否继续。

在 SDK 中，相同情况表现如下：

| 看到的信息 | 含义 |
| --- | --- |
| `AuthenticationError: Unauthorized (401)` | 已设置令牌，但令牌不正确。 |
| `result.success` 为 `False`，`result.status` 为 `"timeout"` | SDK 等待超时；默认超时时间为 180 秒。请传入 `timeout=420`，或重新运行。 |
| `result.data` 中某一行包含 `error` 键 | 这是 API 对某条输入返回的结果，可能表示指定时间范围内没有帖子，或账户不存在；其他行不受影响。 |

## 编程智能体

无需编写 Python，也无需预先安装任何内容。粘贴以下两行即可；第一行会打开一次浏览器。如果通过 SSH 或在 CI 中使用，请改用 `bdata login --device`：

```bash
npx -p @brightdata/cli bdata login
npx -p @brightdata/cli bdata pipelines instagram_posts "https://www.instagram.com/p/Db_SePSltfz/"
```

CLI 的四种 Instagram 数据管道分别接受帖子、Reels 短视频或个人资料 URL，并返回该 URL 对应的一条记录。若要获取某份个人资料的近期帖子，请使用「快速开始」中的 SDK 调用；CLI 没有对应命令。

运行 `npx skills add brightdata/skills`，可以让 Claude Code、Cursor 和 Codex 学会这些命令并了解相关文档，此后就能用自然语言提出需求。完整指南：[让编程智能体使用 Bright Data](https://docs.brightdata.com/quickstart-coding-agent)。

使用托管助手、没有终端？[Bright Data MCP 服务器](https://github.com/bright-cn/brightdata-mcp#which-tool-to-use)的 `social` 工具组中也提供这四种 Instagram 工具，每次处理一个 URL。该工具组默认关闭，需要明确启用：

```text
https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN&groups=social
```

智能体还可以自行创建账户，无需填写注册表单：[智能体注册](https://www.bright.cn/auth.md)。如需了解 Bright Data 与 LangChain、Zapier、n8n 等工具的其他集成方式，请参阅[集成文档](https://docs.brightdata.com/integrations/introduction)。

<details>
<summary>CLI 提供的全部 Instagram 命令，均于今日运行验证</summary>

共四种类型：`instagram_profiles`、`instagram_posts`、`instagram_reels` 和 `instagram_comments`。它们接受 URL、输出 JSON，并按每条记录 1 个积分计费。运行每条命令时都可以加上前缀 `npx -p @brightdata/cli`；也可以通过 `npm i -g @brightdata/cli` 安装一次后直接使用。

```bash
bdata pipelines instagram_profiles "https://www.instagram.com/nasa/" --pretty
```

1 条记录，25 个字段，用时 50 秒。`"account": "nasa"`，`"followers": 104396028`。

```bash
bdata pipelines instagram_posts "https://www.instagram.com/p/Dc1W1uFj-CW/" --format csv -o posts.csv
```

```text
Triggered collection with snapshot ID:sd_mtrcu6qpyebqgtcjg
Output written to posts.csv
```

用时 37 秒。CSV 表头使用与下表相同的字段名。

```bash
bdata pipelines instagram_reels "https://www.instagram.com/reel/DcMXl1IPNtB/" --pretty
```

1 条记录，25 个字段，用时 23 秒。

```bash
bdata pipelines instagram_comments "https://www.instagram.com/p/Dc1W1uFj-CW/" --pretty
```

5 条记录，11 个字段，用时 9 秒。每条评论消耗 1 个积分，因此请先查看帖子的 `num_comments`。

`bdata pipelines list` 会列出所有类型。`bdata pipelines --help` 会显示 `--format json|csv|ndjson|jsonl`、`-o FILE` 和 `--timeout` 等选项。

</details>

## 支持

发现此仓库中的问题？请[提交 issue](https://github.com/bright-cn/instagram-scraper-python/issues)，并参照 [CONTRIBUTING.md](CONTRIBUTING.md) 说明提供必要信息。

有关 API、账户或积分的问题，请联系 [Bright Data 支持团队](https://brightdata.zendesk.com/hc/en-us/requests/new)。

## 许可证

MIT。
