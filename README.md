# astrbot_plugin_group_active_distiller
此项目灵感与参考来于https://github.com/Shawlei/astrbot_plugin_group_distiller 但其并未拥有主动拉旧 蒸馏自己 蒸馏私聊 只能干等所以出现本项目
astrbot蒸馏群友+蒸馏自己插件
此readme并非最新！
蒸？蒸！还在蒸！想化身赛博生命？来试试这个 推荐接入各大llm api 本地效果也凑合
 群友主动蒸馏器 · AstrBot 插件

> 主动扫描机器人所在的 QQ 群，抓取历史消息，分批次喂给 LLM，蒸馏出详细人设 /
> 说话习惯 / 表情包习惯画像。自带可视化蒸馏台（AstrBot 插件拓展页）。
>
> 与被动蒸馏插件的区别：不靠等群里慢慢攒数据，而是你给一个 QQ 号，它主动跨群把所有
> 能采到的记录拉回来，全量分批蒸馏。

- 插件 ID：`astrbot_plugin_group_active_distiller`
- 版本：1.0.0 · 作者：yg
- 依赖：零第三方依赖（纯标准库 `sqlite3` / `asyncio` / `json` / `re`）

---

 一、安装

把整个 `astrbot_plugin_group_active_distiller` 文件夹放进：

```
AstrBot/data/plugins/astrbot_plugin_group_active_distiller/
```

重启 AstrBot。后台左侧「插件拓展页」会出现 群友主动蒸馏器 / 蒸馏台。

前提：机器人账号已加入目标群，且 OneBot 实现支持 `get_group_msg_history`
（NapCat / Lagrange / go-cqhttp 均可）。

---

 二、核心用法：只给一个 QQ 号

 界面操作（推荐）

1. 打开「🔍 按QQ号蒸馏」页签
2. 输入 QQ 号 → 点「查覆盖」（列出这个 QQ 在哪些群说过话）
3. 勾选要用的群（支持一键全选）——勾选的群是语料范围，只影响「去哪找这个号的发言」，永远不会把群里的别人也一起蒸
4. 点「开始蒸馏」或「按选中的群蒸这个号」
   - 勾「先跨群采集」= 先强制重拉全部群的历史（`full` 全量）
   - 勾「忽略消息数门槛」= 即使这个号自己发言不多也强行蒸馏
   - 不勾采集 = 只用库里已有的数据，速度快
5. 任务卡会实时显示：进度条 / 当前阶段 / 已分析批次 / 预计剩余
6. 完成后画像直接显示在同一页，可切历史版本、复制 system_prompt

> 关于「消息门槛」：`auto_distill_min_messages` 算的是这个号自己发了多少条，
> 不是群里总共多少条。例如库里 794 条消息里他本人只说了 149 条，门槛 500 就会拦住他；
> 这时勾上「忽略消息数门槛」即可强行蒸馏。

 群聊指令

| 指令 | 说明 |
|---|---|
| `/蒸馏 追踪 <QQ号>` | 跨群采集 + 蒸馏（最常用） |
| `/蒸馏 追踪 <QQ号> --full` | 强制全量重拉后再蒸 |
| `/蒸馏 追踪 <QQ号> --no-collect` | 只用本地已有数据直接蒸 |
| `/蒸馏 搜qq <QQ号>` | 只查这个 QQ 出现在哪些群，并采集 |
| `/蒸馏 我自己` | 蒸馏机器人自己（群聊 + 私聊，含自己发出的消息） |
| `/蒸馏 搜群 <关键词>` | 搜索机器人所在的群 |
| `/蒸馏 采集 [群号]` | 采集指定群 / 全部群 |
| `/蒸馏 挖老 [群号] [条数]` | 向更早翻页，把该群最老的历史聊天记录挖出来 |
| `/蒸馏 群列表` | 列出已登记的群 |
| `/蒸馏 对象 <群号>` | 列出该群发言排行 |
| `/蒸馏 蒸馏 <群号> <QQ号>` | 群内单人蒸馏 |
| `/蒸馏 批量 <群号> [数量]` | 群内 Top N 批量蒸馏 |
| `/蒸馏 画像 <群号> <QQ号>` | 查看画像 |
| `/蒸馏 投放 <QQ号>` | 生成可直接用的 system_prompt |
| `/蒸馏 状态` | 运行状态 |
| `/蒸馏 后台` | 后台地址 |

---

 三、可视化后台（8 个页签）

| 页签 | 作用 |
|---|---|
| 🔍 按QQ号蒸馏 | 主入口。查覆盖 → 勾群 → 蒸馏 → 实时进度 → 画像内嵌显示；支持「私聊也算语料」与「蒸馏我自己」 |
| 🧬 人设库 | 所有蒸馏成果的存放与查看中心。左侧列表（对象 / 来源 / 版本 / 素材 / 置信度 / 时间），右侧完整详情 + 历史版本切换 + 复制 prompt |
| 📋 群管理 | 群列表、搜索群、单个/批量开关「自动采集」「自动蒸馏」、一键全选、一键全量采集、一键挖最老历史 |
| 💬 私聊管理 | 与群管理完全同构的私聊版：按私聊名/QQ号过滤、全选、批量开关自动采集/蒸馏、一键全量采集、增量采集、挖最老历史、发现私聊会话 |
| ⚙️ 蒸馏任务 | 任务列表 + 点开看阶段明细、每群命中、耗时、失败原因、可取消 |
| 🎛 参数配置 | 34 个配置项，按组展示，改完即时生效 |
| 💾 数据库 | 文件大小 / 各表行数 / 可回收空间 / 一键清理 / VACUUM 压缩 |
| 📜 运行日志 | 实时运行日志 |

 人设存在哪？去哪看？

- 存：SQLite 表 `personas`，每蒸一次写一条新版本（`version` 自增），
  跨群蒸馏的 `group_id` 记为 `global`。私聊语料的 `group_id` 记为 `private:<QQ号>`。
- 看：
  1. 「🧬 人设库」页签 —— 全部成果的总列表 + 完整详情；
  2. 「🔍 按QQ号蒸馏」页 —— 蒸完自动在下方内嵌展示，不用跳页；
  3. 群聊指令 `/蒸馏 画像 <群号> <QQ号>`、`/蒸馏 投放 <QQ号>`。
- 旧版本不会被覆盖，随时可在详情里用「版本下拉」回看。

 私聊与「蒸馏我自己」

- 勾选「私聊也算语料」后，除了群聊，还会拉取与该号的私聊记录（双方发言）。
- 点「🪞 蒸馏我自己（全部群 + 全部私聊）」/ 群聊发 `/蒸馏 我自己`：
  - 自动用 `get_login_info` 取机器人自己的 QQ 号；
  - 所有群里机器人说过的话 + 全部私聊会话（对方发的 + 机器人自己发的）一起算语料；
  - 「全部私聊」不是只扫好友列表：好友列表 ∪ 最近会话 ∪ 库里已登记的会话，
    所以机器人主动发起、好友列表里没有的会话也能覆盖；
  - 每个会话都是双通道拉取（`get_friend_msg_history` + `get_friend_msg_history_with_send`），
    对方说的和自己说的都会入库；
  - 生成的人设 `system_prompt` 可以直接拿去给别的号做人设投放。
- 私聊会话在库里是独立的「群」，`group_id` 形如 `private:10001`，不与真实群混在一起，
  也不会被「一键采集全部群」误当成群去拉。
- 私聊会话的发现来源有三个：`get_friend_list`（好友）、`get_recent_contact`（最近会话）、
  本地已登记的 `chat_type='private'` 记录。后端不支持最近会话接口时会自动跳过，不影响好友部分。
- 「💬 私聊管理」页的按钮与群管理一一对应：
  - 全选 / 批量开启·关闭自动采集 / 批量开启·关闭自动蒸馏 / 刷新列表；
  - ⚡ 一键全量采集（全部私聊）= 按「首次采集条数」重拉每个会话；
  - 增量采集全部私聊 / 只采集勾选的私聊（全量）；
  - ⏳ 一键挖最老历史（全部私聊）/ 只挖勾选的私聊 = 双通道一路向更早翻页。

---

 四、三个关键参数

 1. 每次喂 LLM 多少条（你关心的核心）

| 配置项 | 默认 | 说明 |
|---|---|---|
| `llm_batch_size` | 150 | 每批塞多少条群消息给 LLM。越大单批越准、调用次数越少 |
| `llm_max_batches` | 0 = 不限 | 0 表示不限制批数，所有素材全部喂进去 |
| `llm_sample_char_limit` | 0 = 不截断 | 0 表示不按字数截断 |
| `llm_batch_overlap` | 8 | 相邻批次重叠条数，避免上下文被切断 |
| `llm_concurrency` | 2 | 同时跑几批（1 = 串行，稳妥） |

「全部分批喂」就是靠 `llm_max_batches=0` + `llm_sample_char_limit=0` 实现的：
素材有多少就切多少批，一批不漏，尾部也一定覆盖。

 2. 收录多少条

| 配置项 | 默认 | 说明 |
|---|---|---|
| `default_fetch_count` | 3000 | 首次采集每群拉多少条历史 |
| `oldest_fetch_count` | 3000 | 「挖最老历史」时每群单次最多翻多少条（挖到底自动提前结束） |
| `incremental_fetch_count` | 300 | 之后增量补多少条 |
| `fetch_page_size` | 100 | 单页大小（OneBot 一次 100 最稳） |
| `fetch_page_interval_ms` | 260 | 页间隔，防触发风控 |
| `recent_sample_window_days` | 0 = 不限 | 只取最近 N 天，0 = 全部历史 |
| `auto_distill_min_messages` | 30 | 这个号自己至少说多少条才值得蒸（不算别人说的；不够时可勾「忽略门槛」强蒸） |

 3. 调度

| 配置项 | 默认 |
|---|---|
| `enable_auto_collect` | true |
| `collect_interval_minutes` | 30 |
| `discover_interval_minutes` | 120 |
| `max_groups_per_round` | 8 |
| `auto_distill_after_collect` | true |
| `auto_distill_top_n` | 3 |

---

 五、产出的画像长什么样

```json
{
  "summary": "群里最活跃的抬杠选手，嘴硬心软，情绪表达夸张。",
  "personality": {
    "traits": ["嘴硬心软", "情绪外放", "抬杠专业户"],
    "social_role": "话题发起者 / 气氛组",
    "energy": "高",
    "humor_style": "阴阳怪气 + 自嘲"
  },
  "speaking_style": {
    "avg_length": "极短",
    "sentence_pattern": "短句连发，常用逗号或空格断句",
    "punctuation": "基本不加句号，爱用感叹号和省略号",
    "typing_quirks": ["爱打错别字", "常连发多条"],
    "emoji_usage": "少用 emoji，多用小黄脸和狗头"
  },
  "catchphrases": [
    {"text": "笑死 根本笑不死", "usage": "别人说蠢话时", "frequency": "高"}
  ],
  "vocabulary": {
    "favorite_words": ["离谱", "绝了", "有点东西"],
    "slang": ["破防", "摆烂", "属于是"],
    "particles": ["吧", "啊", "捏"],
    "taboo_words": ["尊敬的", "您好"]
  },
  "sticker_habit": {
    "frequency": "高频",
    "preferred_types": ["熊猫头", "滑稽", "doge"],
    "usage_pattern": "词穷时和怼人收尾时甩图",
    "text_faces": ["(手动狗头)", "[捂脸]"]
  },
  "interaction_style": {
    "reply_speed": "秒回",
    "at_habit": "爱 @ 熟人抬杠",
    "topic_driven": ["游戏", "吃瓜", "吐槽工作"],
    "conflict_style": "直接硬刚然后光速和好"
  },
  "scenario_examples": [
    {"scene": "被人调侃", "typical_reply": "笑死 你行你上"}
  ],
  "avoid": ["不要用敬语", "不要长篇大论", "不要自称 AI"],
  "confidence": 0.82,
  "source_groups": [{"group_id": "123456", "group_name": "技术交流群", "messages": 421}]
}
```

同时会自动渲染成投放用的 `system_prompt`（含说话方式、口头禅、表情包习惯、禁忌）。

---

 六、实现要点

- 分页游标不做 `-1` 位移：NapCat 的 `message_seq` 不连续，位移会报"消息不存在"。
  锚点优先级 `message_seq > real_id > seq > message_id`。
- 全量分批：`sample_lines(..., max_batches=0, char_limit=0)` 按 `batch_size`
  逐步切窗，并额外补一段尾部窗口，保证最后的消息也能进 LLM。
- JSON 三级兜底：直接 `json.loads` → 剥 ```` ```json ```` 围栏 → 括号配对截取。
- 跨群聚合：`fetch_context_global` 按群取上下文窗再合并，保留他人发言作语境。
- 实时进度：后台每阶段回调 `store.update_job()` + `store.add_step()`，
  前端 2s 轮询 `/jobs/detail`，ETA = `已耗时 × (100-进度) / 进度`。
- 版本管理：每次蒸馏 `version = MAX+1`，可回看任意历史版本。
- 去重键：`platform:group:message_id`，跨轮采集不重复入库。
- 自己的数据库：`data/distiller.db`（SQLite，WAL 模式），9 张表，支持一键清理 + VACUUM。
- 挖最老历史：`fetch_group_history_forward` 用当前最早一条的 `message_seq` 作锚点
  一路往前翻，锚点推不动就判 `reached_end`，天然防死循环；
  支持 `count` 上限、`stop_before_ts` 目标日期、`anchor` 断点续挖。
  群管理页的「最早」列由 `message_time_range`（`MIN(time)`）实时给出。

---

 七、自测

```bash
cd astrbot_plugin_group_active_distiller
python selftest.py
```

31 组 / 120+ 项断言，覆盖：JSON 兜底、人设归一化、搜群、全量/增量采集、表情包识别、
跨群查询、全量分批、进度与 ETA、批量开关、勾选群蒸馏、画像可见性、数据库清理/VACUUM、
页面元素与接口一致性。不需要真 QQ、不需要真 LLM。

「挖最老历史」另有 6 个场景的独立验证（假 OneBot 端，5000 条历史群）：
一路翻到最早一条、`count` 上限、`stop_before_ts` 日期截断、`anchor` 断点续挖、
协议端不认锚点不失速、`on_progress` 实时回调递增。

---

 八、注意

- `llm_max_batches=0` 时，一个 QQ 号在几十个群都有发言的话，批次会很多，
  LLM 调用成本随之上升。想控成本就把 `llm_max_batches` 设成 10~20。
- 采集走的是机器人自己的权限，只能采到机器人所在群的消息。
- 拉历史的页间隔别调太低（默认 260ms），容易被 OneBot 实现限流。
- 挖最老历史很重：一个 5000 条的群按 100 条/页、260ms 间隔要翻 50 页 ≈ 13 秒；
  十几个群一次性挖老建议先跑一个群看效果，别一上来就把 `oldest_fetch_count` 拉到 5 万。
  协议端（NapCat / LLOneBot）对翻页有频控，翻到一半报错是正常的，用 `anchor` 续挖即可。
