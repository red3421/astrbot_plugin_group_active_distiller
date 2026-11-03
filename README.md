# astrbot_plugin_group_active_distiller
此项目灵感与参考来于https://github.com/Shawlei/astrbot_plugin_group_distiller 但其并未拥有主动拉旧 蒸馏自己 蒸馏私聊 一键删库等 只能干等所以出现本项目

astrbot蒸馏群友+蒸馏自己插件
此readme并非最新！

蒸？蒸！还在蒸！想化身赛博生命？来试试这个 推荐接入各大llm api 本地效果也凑合
 # 群友主动蒸馏器 · AstrBot 插件

> **主动**扫描机器人所在的 QQ 群，抓取历史消息，**分批次**喂给 LLM，蒸馏出详细人设 /
> 说话习惯 / 表情包习惯画像。自带可视化蒸馏台（AstrBot 插件拓展页）。
>
> 与被动蒸馏插件的区别：不靠等群里慢慢攒数据，而是**你给一个 QQ 号，它主动跨群把所有
> 能采到的记录拉回来，全量分批蒸馏**。

- 插件 ID：`astrbot_plugin_group_active_distiller`
- 版本：1.1.0 · 作者：red3421
- 依赖：**零第三方依赖**（纯标准库 `sqlite3` / `asyncio` / `json` / `re`）

---

## 一、安装

把整个 `astrbot_plugin_group_active_distiller` 文件夹放进：

```
AstrBot/data/plugins/astrbot_plugin_group_active_distiller/
```

重启 AstrBot。后台左侧「插件拓展页」会出现 **群友主动蒸馏器 / 蒸馏台**。

前提：机器人账号已加入目标群，且 OneBot 实现支持 `get_group_msg_history`
（NapCat / Lagrange / go-cqhttp 均可）。

---

## 二、核心用法：只给一个 QQ 号

### 界面操作（推荐）

1. 打开「🔍 按QQ号蒸馏」页签
2. 输入 QQ 号 → 点「查覆盖」（列出这个 QQ 在哪些群说过话）
3. 勾选要用的群（支持**一键全选**）
4. 点「开始蒸馏」
   - 勾「先跨群采集」= 先强制重拉全部群的历史（`full` 全量）
   - 不勾 = 只用库里已有的数据，速度快
5. 任务卡会**实时**显示：进度条 / 当前阶段 / 已分析批次 / 预计剩余
6. 完成后画像**直接显示在同一页**，可切历史版本、复制 system_prompt

### 群聊指令

| 指令 | 说明 |
|---|---|
| `/蒸馏 追踪 <QQ号>` | 跨群采集 + 蒸馏（最常用） |
| `/蒸馏 追踪 <QQ号> --full` | 强制全量重拉后再蒸 |
| `/蒸馏 追踪 <QQ号> --no-collect` | 只用本地已有数据直接蒸 |
| `/蒸馏 搜qq <QQ号>` | 只查这个 QQ 出现在哪些群，并采集 |
| `/蒸馏 搜群 <关键词>` | 搜索机器人所在的群 |
| `/蒸馏 采集 [群号]` | 采集指定群 / 全部群 |
| `/蒸馏 群列表` | 列出已登记的群 |
| `/蒸馏 对象 <群号>` | 列出该群发言排行 |
| `/蒸馏 蒸馏 <群号> <QQ号>` | 群内单人蒸馏 |
| `/蒸馏 批量 <群号> [数量]` | 群内 Top N 批量蒸馏 |
| `/蒸馏 画像 <群号> <QQ号>` | 查看画像 |
| `/蒸馏 投放 <QQ号>` | 生成可直接用的 system_prompt |
| `/蒸馏 状态` | 运行状态 |
| `/蒸馏 后台` | 后台地址 |

---

## 三、可视化后台（6 个页签）

| 页签 | 作用 |
|---|---|
| **🔍 按QQ号蒸馏** | 主入口。查覆盖 → 勾群 → 蒸馏 → **实时进度** → 画像内嵌显示 |
| **📋 群管理** | 群列表、搜索群、单个/批量开关「自动采集」「自动蒸馏」、一键全选 |
| **⚙️ 蒸馏任务** | 任务列表 + 点开看阶段明细、每群命中、耗时、失败原因、可取消 |
| **🎛 参数配置** | 33 个配置项，按组展示，改完即时生效 |
| **💾 数据库** | 文件大小 / 各表行数 / 可回收空间 / 一键清理 / VACUUM 压缩 |
| **📜 运行日志** | 实时运行日志 |

---

## 四、三个关键参数

### 1. 每次喂 LLM 多少条（你关心的核心）

| 配置项 | 默认 | 说明 |
|---|---|---|
| `llm_batch_size` | **150** | 每批塞多少条群消息给 LLM。越大单批越准、调用次数越少 |
| `llm_max_batches` | **0 = 不限** | **0 表示不限制批数，所有素材全部喂进去** |
| `llm_sample_char_limit` | **0 = 不截断** | 0 表示不按字数截断 |
| `llm_batch_overlap` | 8 | 相邻批次重叠条数，避免上下文被切断 |
| `llm_concurrency` | 2 | 同时跑几批（1 = 串行，稳妥） |

**「全部分批喂」就是靠 `llm_max_batches=0` + `llm_sample_char_limit=0` 实现的**：
素材有多少就切多少批，一批不漏，尾部也一定覆盖。

### 2. 收录多少条

| 配置项 | 默认 | 说明 |
|---|---|---|
| `default_fetch_count` | **3000** | 首次采集每群拉多少条历史 |
| `incremental_fetch_count` | 300 | 之后增量补多少条 |
| `fetch_page_size` | 100 | 单页大小（OneBot 一次 100 最稳） |
| `fetch_page_interval_ms` | 260 | 页间隔，防触发风控 |
| `recent_sample_window_days` | **0 = 不限** | 只取最近 N 天，0 = 全部历史 |
| `auto_distill_min_messages` | 30 | 群里某人至少说多少条才值得蒸 |

### 3. 调度

| 配置项 | 默认 |
|---|---|
| `enable_auto_collect` | true |
| `collect_interval_minutes` | 30 |
| `discover_interval_minutes` | 120 |
| `max_groups_per_round` | 8 |
| `auto_distill_after_collect` | true |
| `auto_distill_top_n` | 3 |

---

## 五、产出的画像长什么样

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

## 六、实现要点

- **分页游标不做 `-1` 位移**：NapCat 的 `message_seq` 不连续，位移会报"消息不存在"。
  锚点优先级 `message_seq > real_id > seq > message_id`。
- **全量分批**：`sample_lines(..., max_batches=0, char_limit=0)` 按 `batch_size`
  逐步切窗，并额外补一段尾部窗口，保证最后的消息也能进 LLM。
- **JSON 三级兜底**：直接 `json.loads` → 剥 ```` ```json ```` 围栏 → 括号配对截取。
- **跨群聚合**：`fetch_context_global` 按群取上下文窗再合并，**保留他人发言**作语境。
- **实时进度**：后台每阶段回调 `store.update_job()` + `store.add_step()`，
  前端 2s 轮询 `/jobs/detail`，ETA = `已耗时 × (100-进度) / 进度`。
- **版本管理**：每次蒸馏 `version = MAX+1`，可回看任意历史版本。
- **去重键**：`platform:group:message_id`，跨轮采集不重复入库。
- **自己的数据库**：`data/plugin_data/astrbot_plugin_group_active_distiller/distiller.db`（SQLite，WAL 模式），8 张表，支持一键清理 + VACUUM。
- **数据持久化位置**：所有用户数据统一落在 AstrBot 的 `data/plugin_data/<插件名>/`
  （本插件为 `data/plugin_data/astrbot_plugin_group_active_distiller/`），
  不写插件代码目录，随数据目录一起迁移与审计。老版本写在插件目录 `data/distiller.db`
  的库会在首次启动时自动复制到新位置（原文件保留，可自行删除）。
  目录解析顺序：环境变量 `ASTRBOT_DATA_DIR` → `astrbot.core.utils.astrbot_path` →
  astrbot 安装位置向上推导 → 插件目录兜底。

---

## 七、自测

```bash
cd astrbot_plugin_group_active_distiller
python selftest.py
```

31 组 / 120+ 项断言，覆盖：JSON 兜底、人设归一化、搜群、全量/增量采集、表情包识别、
跨群查询、全量分批、进度与 ETA、批量开关、勾选群蒸馏、画像可见性、数据库清理/VACUUM、
页面元素与接口一致性。测试数据写在 `data/plugin_data/astrbot_plugin_group_active_distiller/`。
**不需要真 QQ、不需要真 LLM。**

---

## 八、注意

- `llm_max_batches=0` 时，一个 QQ 号在几十个群都有发言的话，批次会很多，
  LLM 调用成本随之上升。想控成本就把 `llm_max_batches` 设成 10~20。
- 采集走的是机器人自己的权限，**只能采到机器人所在群**的消息。
- 拉历史的页间隔别调太低（默认 260ms），容易被 OneBot 实现限流。
