# AI-News / AI Daily Board — HANDOVER

> 给接手的 agent：读完本文件 + `AGENTS.md` 就能接着干，不需要翻聊天记录。
> 首版 2026-10-01，**本次更新 2026-10-30（东京时间）**。数字都是 10-30 当日实测，会过期，用「§9 自检命令」重新量。
> ★读文档前先看 §1.2：有一个**正在漏报的生产问题**还没修。
> 本文件是公网脱敏版；完整版在内部仓库，需要的向负责人索取。

---

## 0. 一句话

每天自动刷新一个 **AI Daily Board**（单文件 HTML 看板）：AI 新闻、AI 危险平衡杆、AGI 时间预测、
**61 位 KOL 观点卡（年度时间线 + 中文解读 + 全文翻译）**、LMArena 模型榜、能力领域榜、年度里程碑。
发布到内部看板托管（正常）与公网 Cloudflare Pages（**目前停更，见 §1.2-0**），并每天早上往 Telegram 推一条中文早报。

## 1. 当前状态（2026-10-30 实测）

| 项 | 值 |
|---|---|
| KOL 名册 | **61 位**，`year_timeline` **1081 条**，其中 **1064 条已有解读/译文**（17 条无：多为纯链接/付费墙截断） |
| 危险平衡杆 | **13.6**（10-01 为 12.6；立场分布 +2:16 / +1:15 / 0:8 / −1:13 / −2:9；轴分布 existential 18 / power-race 13 / acceleration 7 / skeptic 23） |
| AGI 预测点 | 15 个 |
| 公网看板 | ★**已停更 41 天**：线上 Cloudflare Pages 的 `generated_at_jst=2026-09-19`、39 位 KOL、危险指数 36.0（本地为 61 位 / 13.6）。原因见 §1.2-0。GitHub Pages 已被账号级封锁（404）。公网仓库每天仍会提交，但**没人把它发到 Cloudflare** |
| 每日 cron | 8 个，见 §4；**近 30 天全部 `last_status=ok`** |
| 建卡脚本 | `scripts/` 共 93 个文件，其中 30 个 `add_*.py`（一人一脚本，改名册必走脚本，见 §6.3） |
| 磁盘 | 项目 1.9G（`scratch/` 里有多个 `.mp3/.m4a` 转写音频，可清；`data/` 约 167M） |

### 1.1 10-01 之后发生的事（时间线）
- **10-01**：一天入册 11 人（49→60 位）+ 马库斯补两期访谈 + 「芯片制程格局」section + 三个 agent cron 降本换 `claude-haiku-4-5`。
- **10-02**：莫斯塔克入册（第 61 位）；Gawdat 补一期（2025-05 Digital Disruption 访谈，**旧料作立场沿革**）。
- **10-04**：负责人 指出《纽约时报》Anthropic 密会宗教领袖这条**漏了五天**——起因与修法见 §2.1。早报上线方案 2（A 加权 + B 兜底两通道），当天已推送。
- **10-05 至 10-30**：无人工改动，全靠 cron 自跑；名册仍 61 人，**没有新点名**。

### 1.2 ★未完成事项（接手第一件事看这里）
0. **【公网看板停更 41 天】** 线上 Cloudflare 卡在 2026-09-19（39 位 KOL / 危险指数 36.0），本地已是 61 位 / 13.6。
   - **根因**：09-18 因 GitHub 账号被滥用检测标记，公网通道改走 Cloudflare Pages，发布器是 `~/.hermes/scripts/publish_cf_pages.sh`
     （通用发布器，指纹戳 `public-site/.cf_last_sha256_ai-daily-board`）。但 `scripts/daily_refresh_runner.sh` / `scripts/publish_public.sh`
     **从来没接上它**：`publish_public.sh` 仍只 `git push` 到个人 GitHub（已 404）。context-log 在 09-19 和之后两次归档里都记了这条
     「Cloudflare 落后、runner 未接 publish_cf_pages、至今未修」，**但没人修，我 10-01 写的交接文档还把它写成「主入口」——那是错的。**
   - **为什么自检没发现**：我 10-30 写的 `handover_selfcheck.sh` 第一版只测「`data.json` 能读」，没比日期，所以显示 PASS。已改成比对 `generated_at_jst` 与本地。
   - **待 负责人 决定**：是否把 `publish_cf_pages.sh` 接进 `daily_refresh_runner.sh`（改生产发布链路，公网内容会一次性跳到最新版）。
     **在他回复前不要改**；也不要手动执行发布器——那会让公网一次性换成 61 位版本，他可能想先看。
   - 补发前提醒：公网内容必须先过发布脚本里的「内网痕迹」终检（`publish_public.sh` 内已有，命中即中止），不要绕过。
1. **【正在漏报】早报会整份静默（`[SILENT]`），30 天里 3 次**：10-08、10-23、10-26。
   - 我逐日核对过：三天脚本数据都正常、`stale_warning=null`；10-08/10-23 当日新闻分别有 7/10 条，10-26 新闻段为空但 KOL 日报有 17 位合格表态。
   - **根因**：`[SILENT]` 是 cron 框架注入 prompt 的通用段（「没有新东西就回 `[SILENT]`」），项目自己的 prompt 里**没有**规则禁止它；`claude-haiku-4-5` 自行判断后选了静默。换 Haiku 前（9 月）0 份静默，换后 3/26。
   - **负责人 2026-10-30 的决定：「只记录」——不改早报 cron，prompt 保持现状。接手方没有他的新指令前不要改。**
   - **备选修法（他没批，仅记录）**：在 早报 job 的 prompt 里加一条硬规则——「只要 NEWS 与 KOL_REPORT 任一段 `ok:true` 且有条目，就**必须发**；`[SILENT]` 仅在所有段均 `ok:false` 或 empty 时才允许」。改完用 `hermes cron edit`（见 §4）并读回逐字比对。
   - 兜底方案：退回 `claude-sonnet-5-5`（贵约 3 倍，但 9 月 0 次静默）。
2. **Chris Olah 是否入册**：他是 10-04 那条宗教新闻的主角，**不在名册**。负责人 问过后一直没回，**不要再追问**，等他自己提。
3. **待 负责人 定（从 9 月拖下来的）**：杜罗夫是否加入名册；看板改版做一版还是三版。
4. **早报方案 2 的噪音**：高分兜底偶尔混入原分 70 的技术帖，教皇 Leo XIV 批评 AI 艺术那条（原分 54）因摘要引 NYT 被判媒体直通。负责人 没要求收紧；嫌吵就把 `WATCH_MIN_SCORE` 从 70 提到 75。
5. 内部看板需要登录，agent 读不回线上页面，每次发布后请负责人自己打开确认。
6. **没有后台排队进程**了（10-01 的 `wait_then_explain.sh` 早已跑完，解读覆盖 1064/1081）。

## 2. 架构与数据流

```
┌ 数据层（每天 07:00，no_agent 纯脚本）
│  ai_kol_daily.sh : fetch_kol_daily → retry_kol_free → finalize_kol_daily → push_kol_daily_notion
│                    产物 data/kol/daily/<日期>.json
├ 看板层（ai-daily-board，12:30，agent 只写简报；真正干活的是 scripts/daily_refresh_runner.sh）
│  fetch_aihot_leaderboard → fetch_domain_benchmarks
│  → merge_daily_to_roster(--days 3) → refetch_fulltext → explain_and_translate → merge_explain
│  → scrub_secrets(--apply) → generate_board_data → archive_and_summarize → push_board_notion
│  → build_single_html → build_hub → 内部看板打包上传 → publish_public.sh
├ LMArena（08:40，no_agent，wrapper 秒退并后台 spawn runner，绕 cron 120s 硬限）
├ 早报（08:00，agent）
│  ai_news_morning_fetch.sh 三段：NEWS(aihot 24h) / KOL_REPORT / KOL_LEGACY_FALLBACK
└ 周报（周六 07:30）/ 月报（1 号 07:30）：rollup_kol.py，汇报并入 08:00 早报，不单独发
```

SSOT 与产物（**搞混会白干**）：
- **SSOT**：`data/kol_backfill.json`（`kol_cards`）、`data/timeline.json`（里程碑，只增不删）、
  `data/kol/{explain,fulltext}.jsonl`（解读/全文，按 url 键）。
- **构建产物**（每天被整体覆盖）：`dashboard/ai-daily-board/data.json`、`dist/{board,hub}.html`、`public-site/*`。
  成果写进产物会被下次重建冲掉——结果要写回 SSOT。

### 2.1 ★早报选稿设计：A 加权 + B 兜底（负责人 2026-10-04 批「方案 2」，脚本 `scripts/fetch_ai_news.py`）

**起因**：2026-09-29《纽约时报》报道 Anthropic 向宗教学者密会、签 NDA、讨论 Claude 是否有意识。AIHOT 对这件事**从头到尾没给过高分**
（10-01 首报 56，10-02 60，10-03 最高 75，10-04 65），原规则「原分 ≥80 全给，不足取最高 3 条」把它挡在线外，**五份早报都没提**，
是 负责人 自己在社交平台上看到才发现。**评分只衡量内容本身，不衡量「本项目关心的公司」**。

**为什么不能只调阈值**（7 天 1400 条回放，`scratch/aihot_7d_all.json`）：点名五大实验室的条目 457 条，非精选里原分 ≥60 的有 117 条，
同一事件常有 4–5 条转述（Gemini 4 Argon 单事件占好几个名额）。阈值设 60 会刷屏，所以必须**先聚事件再取**。

| 通道 | 规则 | 每天上限 |
|---|---|---|
| 主榜 `items` | 有效分 = 原分 + A 加权；≥80 全给（上限 6），不足取最高 3 条 | 6 |
| **A 加权** | 标题/摘要点名五大实验室**且**命中「信源句式」的七家媒体 → +15。输出里 `score` 仍是原分，另给 `boosted_score`、`boost_reason` | — |
| **B 媒体直通** `watch_media` | 七家媒体报道 + 点名五大实验室，**不看分数**，事件去重，排除已在主榜的事件 | 2 |
| **B 高分兜底** `watch_score` | 过去 24h `mode=all`，原分 ≥70，点名五大实验室，事件去重，排除主榜 | 2 |

- 五大实验室词表 `LABS`：Anthropic/Claude、OpenAI/ChatGPT/GPT-n/奥尔特曼、Google DeepMind/Gemini/哈萨比斯、xAI/Grok/马斯克、Meta/Llama/扎克伯格，另含 Dario/Amodei/Olah。
- 七家媒体 `MAJOR_MEDIA`：NYT / WSJ / Reuters / Bloomberg / FT / Guardian / WaPo。**必须是信源句式**：「据《纽约时报》报道」「援引彭博社」「Bloomberg：…」「路透社称」；
  单纯出现媒体词不算（回放发现「绕过 NYT 付费墙」「路透泄露招股书」会误中，已排除）。
- 事件去重 `cluster()`：共享实体词且共享 ≥2 个中文词，或共享 ≥4 个中文词，归为一簇，簇内取最高原分。
- `fetch_pool()` 用 cursor 翻页拉 `mode=all`（`limit` 最大 100，超了返回 400；`window` 只认 `24h`/`7d`）。
- **回放验证**（10-01~10-04 每天 08:00 窗口）：宗教事件 10-01（原分 56）与 10-03（原分 75）都会进早报；4 天兜底合计 15 条，日均 3.8、上限 4。
- **cron 提示词配套**：早报 prompt 里让 Haiku 把 `watch_media`+`watch_score` 合成一节「值得留意（过去 24 小时的实验室动态）」，每条一句话 + 链接，
  媒体直通标「（媒体直通）」、加权标「（加权）」；两数组皆空整节省略。**脚本新增字段而 prompt 不改，agent 看不到——两处必须一起改。**
- 备份：`fetch_ai_news.py.bak_pre_AB2`（改前版）；v1「原分≥60」试验版在 `scratch/fetch_ai_news.AB_v1_60.py`（会刷屏，仅备查）。

### 2.2 早报 30 天运行实况（10-05 至 10-30，共 26 份，来源 `~/.hermes/cron/output/<早报job-id>/*.md`）
- **静默 3 份**（见 §1.2-1）。**有「值得留意」小节 21 份，有「媒体直通」标记 16 份**。10-30 当天（最新）两者都有。
- Haiku 的输出稳定性：其余日子无编造迹象，我抽读过 10-05、10-30；`stale_warning` 从未触发（数据层全绿）。
- 看板简报 cron（同样 Haiku）近 30 天**没有静默**。

## 3. 目录

```
AGENTS.md                 项目规则与历史踩坑（长，必读）
HANDOVER.md               本文件
dashboard/ai-daily-board/ 看板模板 index.html + 产物 data.json + dist/
data/                     SSOT 与每日归档（~167MB；kol_backfill.json 每次建卡前自动备份在 scratch/backups/）
docs/context-log.md       逐日上下文日志（最新在文末，最详细的「当天做了什么」）
docs/SESSION_HANDOVER_20260824.md   上一轮交接（中文化/莫兰迪改版/三层展开/静默失败三连）
docs/schema.md            数据结构
public-site/              公网发布副本（自带 .git，指向个人 GitHub 仓库 ai-daily-board；该仓库每天被提交，但 Pages 已 404，真正对外的是 Cloudflare，见 §1.2-0）
scripts/                  93 个文件（含 30 个 add_*.py）（add_kol_*.py = 一人一脚本的建卡脚本，见 §6）
scratch/                  临时与备份，gitignore，不进仓库（备份在 scratch/backups/，丢了没法恢复）
```

## 4. cron 清单（本机 `~/.hermes/cron/jobs.json` 实测；机器可读版见 `ops/cron-jobs.json`）

| job id | 名称 | 时间(JST) | 类型 | 模型 | 作用 |
|---|---|---|---|---|---|
| `<job-id>` | ai-kol-daily | 07:00 | no_agent 脚本 | — | KOL 每日抓取链路 |
| `<job-id>` | ai-kol-weekly | 周六 07:30 | no_agent | — | 周报汇总 |
| `<job-id>` | ai-kol-monthly | 每月 1 号 07:30 | no_agent | — | 上月月报 |
| `<job-id>` | ai-news-morning-brief | 08:00 | agent | **claude-haiku-4-5** | 早报（新闻 + KOL 日/周/月） |
| `<job-id>` | ai-lmarena-fetch | 08:40 | no_agent | — | LMArena 抓取 |
| `<job-id>` | ai-daily-board | 12:30 | agent | **claude-haiku-4-5** | 跑刷新 + 写 Telegram 简报 |
| `<job-id>` | ai-timeline-monthly-review | 每月 1 号 10:00 | agent | claude-sonnet-5-5 | 只出「建议追加清单」，**绝不自动写 timeline.json** |
| `<job-id>` | ai-news-context-distill | 01:30 | agent | claude-sonnet-5-5 | 每日上下文归档，只存本地 |

近 30 天 8 个 job 的 `last_status` 全为 `ok`；早报 prompt 已含方案 2 的「值得留意」小节（公网版不附 prompt 原文，需要的向负责人索取内部版）。

要点：
- ★`ai-daily-board` 排 12:30 是为避开 11:00 eco-vol（全量宏观扫描，持工作目录锁十几分钟）与 09:20 Forecast-Checker。
  带 `workdir` 的 cron 独占目录锁，别的 job 等满 660s 就 TimeoutError，**排班避开长任务窗口，报错≠job 有毛病**。
- ★`ai-kol-*` 是纯脚本，不调模型；agent 型三个才有模型费用。
- ★**改 cron 模型用 `hermes cron edit <id> --model X --provider custom`**。
  `cronjob_manage update` 的 `model` 字段不生效，且至少要传一个字段——**传占位值会把 prompt 整个覆盖**
  （10-01 晚我踩过，已从 `jobs.json.bak-*` 取回原文逐字比对恢复）。改前先备份 `~/.hermes/cron/jobs.json`。
- 可用的便宜模型：`claude-haiku-4-5`（代理实测通）。`claude-haiku-5` / `5-5` 返回 404，不存在。
- 日更 cron 的 prompt 铁律：不编造；`stale_warning` 必须原样转达；某段 `ok:false` 或 empty 要如实写「该段今日无数据」。
- 自愈：另有全局 job `Cron Retry Self-Heal`（每小时）会催失败的 job 重跑，且要等 `last_run_at` 真推进才算恢复。

## 6. 日常操作

### 6.1 手动刷新并发布
```bash
cd ~/Projects/AI-News && bash scripts/daily_refresh_runner.sh      # 全流程，版本号自动 +1
```
单独：内部看板发布脚本（向负责人索取内部版）、`bash scripts/publish_public.sh`（公网）。
**发布命令和含 `&` 的 heredoc 不要放同一条命令**，否则整条被拒；长命令一律走脚本。

### 6.3 ★加一位 KOL（本项目最高频操作，负责人 点名人物时）
固定流程，每一步都有门禁：
1. **核实身份**：负责人 的描述当「待核实清单」。10-01 一天 8 次点名，描述有误的占多数
   （All-In 实为固定三人 + 常驻第五位、一位已卸任白宫头衔；「AWS 机器学习全球负责人」实为面向初创与风投的子业务；
   Ryan Kick 实为 Kiick；「两人均有 15 年经验」指主持人和嘉宾）。**交付第一段点破，卡片写正确值。**
2. **已在名册就给原卡补言论，不重复建卡**（先 grep `kol_backfill.json`）。
3. **抓一手来源**：官方逐字稿 > 本人 Substack/官网 > 播客免费音频本机转写。
   - 不用二手摘要（中文 YouTube 摘要频道）；付费内容不收；免费音频被付费墙截断，截断点后不收。
   - YouTube 字幕在本机被封。替代：播客 RSS/iTunes lookup（`itunes.apple.com/lookup?id=…&entity=podcastEpisode`）拿免费音频 →
     `faster_whisper`（small、int8、cpu、`cpu_threads=12`）转写。
   - 无说话人标注的转写：按内容与主持人点名切段，**只收本人段落，拿不准的整句不收**（宁少勿错归）。
     多人共用同一期链接时，条目 url 加 `#who` 锚点，否则解读/译文按 url 存会互相覆盖。
4. **备份**：`cp data/kol_backfill.json scratch/backups/kol_backfill.pre_<name>.json`。
5. **写建卡脚本** `scripts/add_kol_<name>.py`（照 `add_kol_kiick.py`，最小样板；`add_kol_allin.py` 是一脚本多人样板）：
   - `danger_stance` ∈ −2..+2；**没有 AI 风险表态就记 0 且不计入危险杆**。
   - AGI 年份：没有明确年份就留空，**拒绝预测者不编年份**。
   - 评级用星级（名册下限三星）。
   - 数字必须标注是本人说法还是测算；本人口误要核实并标注（马库斯把 AAAI 调查的 76% 说成 85%，卡上并列两个数及出处）。
   - ★**利益相关写在卡上**（格斯特纳的基金投资了被讨论公司）。
   - 立场标签越极端，卡片正文里的保留意见越不能省。
   - 脚本里 `grep -c ANONYMIZED` 必须为 0（redactor 会把人名/术语替换成占位符污染写入的源码，用 `chr()` 拼接或「前 OpenAI CTO」这类写法绕开）。
   - 卡里每个数字对原文做 `n in text` 回查，缺失项人工看一眼（多是转写把数字拼成英文单词）。
6. **全文入库**：追加到 `data/kol/fulltext.jsonl`（`url,kol,old_len,ts,full_text,len,status`）。
7. **解读 + 翻译**：`python3 scripts/explain_and_translate.py --days N` → `merge_explain.py`。
   **有配额门禁**（见 §1.2）。启动命令里不能出现 `build_layers.py` 字样，否则门禁会把自己误判为冲突进程；排队统一用 `scratch/ab/wait_then_explain.sh`。
   自动生成的 gist 偏题时手工追加修正到 `data/kol/explain.jsonl`。
8. **重建**：`generate_board_data.py` → `build_single_html.py` → `build_hub.py`。
9. **浏览器验证**：`python3 -m http.server 8899`（目录 `dashboard/ai-daily-board/dist`）+ `.venv/bin/python` 的 playwright 打开 `hub.html`，数卡片数、查新人名出现次数、页面无 `pageerror`。
10. **发布**内部看板，请负责人自己打开线上确认。
11. 追加 `docs/context-log.md`（用 `execute_code` 追加；含 `&` 的 heredoc 会触发检查）。

## 7. 铁律与踩坑（精选；完整版在 `AGENTS.md` 与 `docs/context-log.md`）

1. **「成功」必须由真实 exit code / 线上实测驱动**。发布脚本曾因 `&& echo pushed || echo failed` 链的位置让失败报喜，公网静默停更两天；
   Cloudflare 上 HTTP 200 不携带信息量，要比对线上内容的日期/特征。
2. **采集端点换版后「还能 200」≠「字段还在」**。aihot 旧端点返回 400 被 `except: return None` 吞成「今天没新闻」；
   v1 里 `permalink` 恒为 null，真值在 `links.aihot`，`source` 是对象不是字符串。
3. **名册只读快照**：看板只读名册 `year_timeline`，而 07:00 的 daily 把言论写到 `data/kol/daily/*.json`——
   曾经没有一步把快照写回名册，停更 11 天两头都是绿的。现由 `merge_daily_to_roster.py --days 3` 负责，且 `_board_summary.py` 有 `ROSTER_STALE` 预警，简报必须原样转达。
4. **批处理「已处理」判据要覆盖所有终态**（成功 / insufficient / 失败），否则 `points_cn=[]` 这类 falsy 终态会被无限重跑。
5. **LLM 批量翻译失败分两类**：429 限流（调间隔有用）与模型静默返空（调间隔无用）。先手打一条失败样本看原始响应。
6. **回源重爬前先做探针**（`probe_refetch.py`）；429 集中在少数域名＝自己打出来的，散落在几十个域名＝出口 IP 被整体判定，只能全局降速。**长任务未跑完不下结论**（曾据中途「只救回 2 条」断言到头，跑完 +18）。
7. **验收必须用真浏览器**：改版时 `class="section-title"` 被改后侧栏目录 10 条全空且零报错，静态 grep 全绿。运行时翻译覆盖率也要在浏览器 DOM 里量，不是磁盘 HTML。
8. **安全研究者的言论天然含真实凭据样本**，翻译会忠实照译并带上公网（曾被 GitHub Push Protection 拦）。`scrub_secrets.py --apply` 必须在建看板前、作用在数据层。**禁用 unblock URL**。
9. **`pgrep -f '<关键字>'` 会匹配到自己的命令行**——等待脚本永远等不完（10-01 踩了两次）。用 `pgrep -f '^绝对路径 …'` 锚定，或直接串行跑。
10. **redactor 会污染写入磁盘的源码**（人名、`KEY=` 字面量）。敏感字面量拼接；人名从磁盘真实原文取，别手打。
11. 公网通道现状：个人 GitHub 账号被 GitHub 滥用检测标记，Pages 全线 404，**唯一的公网通道是 Cloudflare Pages，但发布没接进每日 runner，线上停在 09-19（§1.2-0）**。公网发布已改成「拆分产物 + 仅变更时提交」（`split_public_bundle.py`，每天只变 `data.json`）。别再把整个 hub.html 每天覆盖提交。
12. 带 `workdir` 的 cron 独占目录锁；`cron prompt` 里别写死 KOL 数量。

## 8. 与其他项目的边界
- **War-KOL / Politics-Checker / Forecast-Checker / Science-KOL / UFO-Checker** 是各自独立的 KOL 类项目，**名册不互通**，不要往本项目写它们的数据。
- **共用同一个 LLM 配额**的只有 War-KOL 和 Politics-Checker 的 `build_layers.py`——它们在跑本项目的解读就拒绝启动。
- 同前缀的 cron 可能属于不同项目（`ai-*` 只有 §4 那 8 个属于本项目）；**暂停/改动前核对 workdir 和 script，别按名字批量操作**。

## 9. 自检命令（接手后先跑一遍；期望值是 2026-10-30 的实测，会漂）

**一条命令跑完下面全部**：`bash scripts/handover_selfcheck.sh`（只读；`PASS/WARN/FAIL` 逐条，退出码=FAIL 数）。
2026-10-30 实测基线：**唯一的 FAIL 是公网落后 41 天（§1.2-0），唯一的 WARN 是早报近 7 份里有静默（§1.2-1）——这两条是已知问题，不是你弄坏的**；其余应全 PASS。
下面是它在做的事，想单条跑或在新机上排查时用：
```bash
cd ~/Projects/AI-News
# 1 名册与解读覆盖：期望 61 / ≈1081 / ≈1064
python3 - <<'EOF'
import json
d=json.load(open('data/kol_backfill.json'));c=d['kol_cards']
ex={}
for l in open('data/kol/explain.jsonl'):
    try: r=json.loads(l); ex[r['url']]=r
    except: pass
n=sum(len(x['year_timeline']) for x in c)
h=sum(1 for x in c for t in x['year_timeline'] if t['url'] in ex and (ex[t['url']].get('explain') or ex[t['url']].get('translation')))
print(len(c),n,h)
EOF
# 2 污染占位：已知 4 处是旧卡正文占位（祖宾·加拉马尼、雅各布·考克森），非新引入
grep -c ANONYMIZED data/kol_backfill.json
# 3 8 个 cron 都在、enabled、最近一次 ok
python3 - <<'EOF'
import json
d=json.load(open('/home/user/.hermes/cron/jobs.json'));j=d['jobs'] if isinstance(d,dict) else d
for x in j:
    if x['name'].startswith('ai-'): print(x['id'],x['name'][:28],x.get('model'),x.get('enabled'),x.get('last_status'),str(x.get('last_run_at'))[:16])
EOF
# 4 早报最近 7 份有没有静默（见 §1.2-1）
python3 - <<'EOF'
import glob,os
fs=sorted(glob.glob(os.path.expanduser('~/.hermes/cron/output/<早报job-id>/*.md')))[-7:]
for f in fs:
    r=open(f,encoding='utf8').read().split('## Response')[-1].strip()
    print(os.path.basename(f)[:10],'SILENT' if r.startswith('[SILENT]') else len(r),'值得留意' in r)
EOF
# 5 早报数据层今天能跑：看 watch_media / watch_score 字段在不在
bash ~/.hermes/scripts/ai_news_morning_fetch.sh 2>/dev/null | python3 -c "
import sys,re,json
t=sys.stdin.read();n=t.split('===== SECTION: NEWS')[1].split('===== SECTION: KOL_REPORT')[0];d=json.loads(n[n.index('{'):])
print(sorted(d.keys()));print('主榜',len(d['items']),'媒体直通',len(d.get('watch_media',[])),'高分兜底',len(d.get('watch_score',[])))"
# 6 公网是否最新（★只测「能读」会漏掉停更，必须比对日期；现在应报落后，见 §1.2-0）
curl -s https://ai-daily-board.pages.dev/data.json | python3 -c "import sys,json;d=json.load(sys.stdin);print(d.get('generated_at_jst'),d.get('danger_gauge',{}).get('gauge'))"
# 7 解读门禁：有输出＝门禁会拦解读
pgrep -af '^python3 -u scripts/build_layers.py' | grep -v pgrep
```

## 11. 接手第一小时（按顺序做，每步都有判据）

1. 读本文件 §1.2 + `AGENTS.md`（长，**分段读完**，中段才有 Notion/数据契约）+ `docs/context-log.md` 最后 80 行。
2. 跑 §9 的 8 条自检，逐条对照期望值；任何一条对不上先查原因，不要先动手。
3. **先看 §1.2-1（早报静默）**：用 §9 第 4 条看最近 7 份。若又有静默，向 负责人 报告；**他已决定只记录不修，别擅自改 prompt**，除非他重新授权。
4. 确认你机器上的 cron **没有双跑**：`hermes cron list | grep 'ai-'`。本机（原 agent）的 8 个 job 仍在运行；**接手方起来之前不要重复建，否则重复抓取、双倍烧额度**。
   接手流程：先让 负责人 明确说「原机暂停」，**逐个** `hermes cron pause <id>` 并读回，再在新机用 `ops/cron-jobs.json` 重建；回滚=原机 `resume`。
5. 配好环境变量与笔记库 token（向负责人索取内部版）；没有它们 KOL 日报的笔记库推送会失败，但看板仍能出。
6. 用 §6.1 手动刷新一次（`daily_refresh_runner.sh`），再让 负责人 打开线上确认。
7. 才开始接活：加 KOL 走 §6.3；任何「改名册」都先备份。

## 13. 保持交接包新鲜
`ops/cron-jobs.json` 是**导出快照**，不是 cron 的真身。改了 cron（尤其 prompt、模型）之后必须重导：
读 `~/.hermes/cron/jobs.json`，按 `id` 取本项目 8 个，prompt 里把绝对路径换 `<PROJECT_ROOT>`，写回 `ops/cron-jobs.json`，
再 `rsync` + push。**10-04 方案 2 改了早报 prompt，10-01 导出的 `ops/` 当时已过期**，本次（10-30）已重导。
