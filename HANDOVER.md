# AI-News / AI Daily Board — HANDOVER

> 给接手的 agent：读完本文件 + `AGENTS.md` 就能接着干，不需要翻聊天记录。
> 写于 **2026-10-01（东京时间）**。数字都是当日实测，会过期，用「§9 自检命令」重新量。
> 本文件是公网脱敏版：内部发布通道、内部仓库、密钥位置、cron 提示词原文均已省略。

---

## 0. 一句话

每天自动刷新一个 **AI Daily Board**（单文件 HTML 看板）：AI 新闻、AI 危险平衡杆、AGI 时间预测、
**60 位 KOL 观点卡（年度时间线 + 中文解读 + 全文翻译）**、LMArena 模型榜、能力领域榜、年度里程碑。
发布到公网 Cloudflare Pages，并每天早上往聊天工具推一条中文早报。

## 1. 当前状态（2026-10-01 晚）

| 项 | 值 |
|---|---|
| KOL 名册 | **60 位**，`year_timeline` 888 条，其中 516 条已有解读/译文 |
| 危险平衡杆 | **12.6**（今天 25.1→12.6，主因 All-In 四位投资人入册后负分权重大） |
| AGI 预测点 | 14 个 |
| 公网看板 | Cloudflare Pages `ai-daily-board.pages.dev`（主入口）；GitHub Pages 已被账号级封锁，404 |
| 每日 cron | 8 个，见 §4 |

### 1.1 今天（10-01）新增的人（第 49→60 位，全部带建卡脚本）
齐特伦、伯里、马特·怀特、科彭斯、阿尔诺·贝特朗、约翰·舒尔曼、艾莉·米勒、
All-In 四位（萨克斯 / 帕里哈皮蒂亚 / 格斯特纳 / 卡拉卡尼斯）、瑞安·基克；马库斯补了两期访谈；
新增「芯片制程格局」section。

### 1.2 ★未完成事项（接手第一件事看这里）
1. **约 24 条解读/译文还没生成**：马库斯 2026-01-19 那期、舒尔曼 2 条、米勒、All-In 四位、基克等。
   原因：`explain_and_translate.py` 自带门禁——探测到 Politics-Checker 的 `build_layers.py`
   在跑就拒绝启动（两边共用同一个 LLM 配额，并发会互相 429）。Politics-Checker 一年回溯 526 条，
   10-01 晚跑到第 278 条，预计还要十几小时。
   - 后台已挂 `DAYS=900 bash scratch/ab/wait_then_explain.sh`（等它结束自动开跑，`/tmp/js_explain.log`）。
     **该进程不随 git 走**，接手机器要自己重起或等下一次每日 cron（09:00 前后的 `daily_refresh_runner.sh` 同样会被门禁拦，跑完后会自动补）。
   - **不要用 `--force` 绕门禁**；强跑前问 负责人。
   - 完成后：看 gist 是否偏题（偏题的手工追加到 `data/kol/explain.jsonl`），重建、发布、浏览器核对。
2. **待负责人定**：杜罗夫是否加入名册；看板改版做一版还是三版。
3. **Haiku 降本首日观察**：10-02 08:00 的早报是 `claude-haiku-4-5` 第一次写。看有没有漏 `stale_warning`、编造或把旧数据说成今天；有就退回 `claude-sonnet-5-5`（§4）。
4. 发布后请人工打开线上页面确认，agent 无法读回登录后页面。

## 2. 架构与数据流

```
┌ 数据层（每天 07:00，no_agent 纯脚本）
│  ai_kol_daily.sh : fetch_kol_daily → retry_kol_free → finalize_kol_daily → push_kol_daily_notion
│                    产物 data/kol/daily/<日期>.json
├ 看板层（ai-daily-board，12:30，agent 只写简报；真正干活的是 scripts/daily_refresh_runner.sh）
│  fetch_aihot_leaderboard → fetch_domain_benchmarks
│  → merge_daily_to_roster(--days 3) → refetch_fulltext → explain_and_translate → merge_explain
│  → scrub_secrets(--apply) → generate_board_data → archive_and_summarize → push_board_notion（外部笔记库推送）
│  → build_single_html → build_hub → 打包内部发布载荷 → 内部发布 → publish_public.sh
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

## 3. 目录（公网仓库只含 public-site/ 的两个发布文件 + 本文件）

```
AGENTS.md                 项目规则与历史踩坑（长，必读）
HANDOVER.md               本文件
dashboard/ai-daily-board/ 看板模板 index.html + 产物 data.json + dist/
data/                     SSOT 与每日归档（~167MB；kol_backfill.json 每次建卡前自动备份在 scratch/backups/）
docs/context-log.md       逐日上下文日志（最新在文末，最详细的「当天做了什么」）
docs/SESSION_HANDOVER_20260824.md   上一轮交接（中文化/莫兰迪改版/三层展开/静默失败三连）
docs/schema.md            数据结构
public-site/              公网发布副本（自带 .git，指向个人 GitHub 仓库 ai-daily-board）
scripts/                  89 个脚本（add_kol_*.py = 一人一脚本的建卡脚本，见 §6）
scratch/                  临时与备份，gitignore，不进仓库（备份在 scratch/backups/，丢了没法恢复）
```

## 4. cron 清单（本机 `~/.hermes/cron/jobs.json` 实测（公网版不附 cron 提示词原文，需要的向负责人索取内部版））

| job id | 名称 | 时间(JST) | 类型 | 模型 | 作用 |
|---|---|---|---|---|---|
| `ef7c9ae6b24d` | ai-kol-daily | 07:00 | no_agent 脚本 | — | KOL 每日抓取链路 |
| `1bc71ecc6e55` | ai-kol-weekly | 周六 07:30 | no_agent | — | 周报汇总 |
| `9aaac3b6fe5c` | ai-kol-monthly | 每月 1 号 07:30 | no_agent | — | 上月月报 |
| `a13ba24efc37` | ai-news-morning-brief | 08:00 | agent | **claude-haiku-4-5** | 早报（新闻 + KOL 日/周/月） |
| `cfeaf3a6c662` | ai-lmarena-fetch | 08:40 | no_agent | — | LMArena 抓取 |
| `9fa3d2d89eeb` | ai-daily-board | 12:30 | agent | **claude-haiku-4-5** | 跑刷新 + 写 Telegram 简报 |
| `32a56cce7674` | ai-timeline-monthly-review | 每月 1 号 10:00 | agent | claude-sonnet-5-5 | 只出「建议追加清单」，**绝不自动写 timeline.json** |
| `2e3e82ec4025` | ai-news-context-distill | 01:30 | agent | claude-sonnet-5-5 | 每日上下文归档，只存本地 |

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
cd ~/Projects/AI-News && bash scripts/daily_refresh_runner.sh      # 全流程
```
单独发公网：`bash scripts/publish_public.sh`。
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
10. **发布**（公网 `scripts/publish_public.sh`），人工打开线上页面确认。
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
11. 公网通道现状：个人 GitHub 账号被 GitHub 滥用检测标记，Pages 全线 404，**只剩 Cloudflare Pages**。公网发布已改成「拆分产物 + 仅变更时提交」（`split_public_bundle.py`，每天只变 `data.json`）。别再把整个 hub.html 每天覆盖提交。
12. 带 `workdir` 的 cron 独占目录锁；`cron prompt` 里别写死 KOL 数量。

## 8. 与其他项目的边界
- **War-KOL / Politics-Checker / Forecast-Checker / Science-KOL / UFO-Checker** 是各自独立的 KOL 类项目，**名册不互通**，不要往本项目写它们的数据。
- **共用 LLM 配额**的只有 War-KOL 和 Politics-Checker 的 `build_layers.py`——它们在跑本项目的解读就拒绝启动。
- 同前缀的 cron 可能属于不同项目（`ai-*` 只有 §4 那 8 个属于本项目）；**暂停/改动前核对 workdir 和 script，别按名字批量操作**。

## 9. 自检命令（接手后先跑一遍）
```bash
cd ~/Projects/AI-News
python3 -c "import json;d=json.load(open('data/kol_backfill.json'));c=d['kol_cards'];print(len(c),sum(len(x['year_timeline']) for x in c))"   # 期望 60 / 888
grep -c ANONYMIZED data/kol_backfill.json                      # 已知 4 处是旧卡正文占位（祖宾·加拉马尼、雅各布·考克森），非新引入
~/.hermes/hermes-agent/venv/bin/hermes cron list 2>/dev/null | grep -E 'ai-|ai_'   # 8 个 job 都在且 enabled
tail -3 docs/context-log.md
curl -s https://ai-daily-board.pages.dev/data.json | python3 -c "import sys,json;d=json.load(sys.stdin);print(d.get('generated_at_jst'),d.get('danger_gauge'))"
pgrep -af 'build_layers.py' | grep -v pgrep                    # 有输出＝门禁会拦解读
```

