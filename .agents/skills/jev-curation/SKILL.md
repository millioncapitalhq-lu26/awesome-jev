---
name: jev-curation
description: Curate and expand the awesome-jev repository. Use when adding Jev projects or cases, collecting discussion evidence from X/Reddit/HN/blogs, promoting discussion items into main categories, refreshing README counts, or running periodic evidence sweeps.
---

# Jev Curation

Use this skill to maintain `awesome-jev` as a **strict, high-signal list of projects and practices built on Jev**.

## Goal

Keep the repository focused on two questions:

1. Where is Jev actually being used in public?
2. Which decision patterns transfer across domains?

This skill is for **curation**, not general AI news collection. Jev is a typed-decision model: `Choice`, `Score`, `Boolean`, each with confidence. Entries that do not involve a Jev decision do not belong here.

## Source of truth

Read these files before making changes:

- `README.md`
- `CONTRIBUTING.md`
- `tags.json`
- every file under `categories/`

`README.md` is the homepage aggregate, not the primary editing surface.
Update category files first, then refresh `README.md` from the current category files.
Always use `scripts/build-readme.py` instead of hand-editing the aggregate.
`tags.json` holds the entry tag vocabulary; the table in `CONTRIBUTING.md` is
generated from it, so add a tag value there rather than to either document.
An `agent` value also needs an evidence rule in `SUPPORT` in
`scripts/audit-tags.py`: without one, entries carrying it are never checked
against their own text. The audit names any value it has no rule for, so add
both in the same change.

**Removing a value is not symmetric with adding one.** Adding a value only makes
something available later; removing one silently orphans every entry whose text mentions
it, because the audit cannot propose a tag for a value that no longer exists. That is
what happened to `opencode`: it was removed on 2026-09-22 for a good reason — it had
indeed entered from a misread — and `opencode-jev-compaction` then sat in the catalogue
untaggable for nineteen days with nothing reporting it. So before deleting a value, run
the check the audit cannot:

```bash
grep -ril "<value>" categories/ | sort
```

List what the removal would silence, and record both the removal and the orphans in the
value's `$comment`, so the next person to consider deleting it sees the same list.


> **2026-10-18 起：本列表的 sweep 不再自己发帖。** 三个列表的更新每天合成**一条**合并帖，
> 由独立的 `curation-digest-post` 任务发出。所以每轮结束时把结果追加到
> `~/.agent-reach/state/curation-digest.md` 的 `## <本仓库名>` 段（总数 + 每条新增一句话 +
> 一两条最值得说的未采纳原因）；**无新增就什么都不写**。格式与示例见
> /skill:tweet-posting-cdp 的「跨列表合并帖」一节。

## Hard inclusion rules

Only include items that satisfy at least one of these:

- explicitly name `Jev` / `jev`
- explicitly cite TypeSafe AI's System One models
- clearly implement a typed-decision loop (typed question → typed answer + confidence → accept / reject / escalate)

And all of these:

- source is public and citable
- description is concrete
- entry stays **one sentence**
- item is **strictly Jev-relevant**, not a generic classifier, router, or LLM judge

Reject:

- generic classifiers, routers, or decision agents with no Jev involvement
- launch-hype commentary with no artifact
- private or uncitable claims
- things that need a paragraph to justify inclusion

## Category model

Use **main category pages** for stronger evidence such as:

- public repos
- project pages
- substantial write-ups
- benchmark or evaluation results

Use **`categories/related-practices-discussions.md`** for:

- X threads
- Reddit discussions
- Hacker News threads
- interviews
- blog mentions

when they show credible real practice signals but do not yet have a strong standalone repo or case page.

The category list, their order on the homepage, and which files are still being
seeded all live in `CATEGORIES` and `OPEN_TRACKING` in `scripts/build-readme.py` —
read them there rather than from a copy that goes stale.

## Working strategy

### 1. Search broadly, classify narrowly

Preferred evidence channels:

- GitHub
- X / Twitter
- Reddit
- Hacker News
- Vercel AI Gateway / AI SDK ecosystem
- independent blogs / write-ups

### 2. Keep queries simple

Prefer medium-complexity searches such as:

- `jev typesafe`
- `typesafe-ai/jev`
- `jev Choice Score Boolean`
- `Jev System One Model`
- `jev classification routing`
- `jev verification guardrail`
- `Jev Diogo Almeida`

Avoid very long advanced-search expressions when the adapter is unstable.

### 3. Industrial and physical-world channels

The physical side of the ecosystem is invisible to every name search above: `jev ros2`,
`jev embedded`, `jev plc` and `jev industrial` all return zero repositories, and the
first entries in `robotics-physical.md` came from a web search instead. Search the
sources that cover that industry, not the model name:

- `gh search code 'api.typesafe.ai'` inside robotics and embedded repositories, or read
  their tree directly — a factory integration rarely says "Jev" in its description.
- ROS/ROS2 and embedded ecosystems: `gh search repos "ros2 decision model"`,
  `gh search repos "plc ai classifier"`, `gh search repos "edge inference classifier"`.
- Non-English industry writing. The 2–14 Hz decision-band framing — servo control around
  1000 Hz, VLA models 5–50 Hz, task planning under 1 Hz — came from a Japanese robotics
  column, not from GitHub.
- **Search by endpoint, not by name.** `api.typesafe.ai` and `/v1/systemone` appear in
  code long before anyone writes the word Jev in a description. Two collisions make name
  search unreliable here: **System One is also an unrelated company** (systemone.com,
  flooring and defect software), and MES, SCADA and PLC are generic acronyms that return
  other people's products in prose search.
- **Expect the deployments to be private, and say so.** Factories rarely publish their
  in-house classifier. What stays citable is the enabling layer (OPC-UA, Modbus, BACnet
  and IEC 61131-3 tooling, bounded-action runtimes), vendor case studies and conference
  talks. Absence of a public deployment is not evidence of absence of use — record what
  is public and label it as such rather than concluding the field is empty.

### 4. Chinese + English

Search in both languages when useful. Useful Chinese patterns:

- `jev 大模型 决策`
- `TypeSafe Jev 分类`
- `Jev 类型化决策`

Keep Chinese queries narrow to avoid noisy generic matches.

## `last30days`: the multi-channel sweep

`last30days` (mvanhorn, MIT) is installed at `~/.agents/skills/last30days`, a symlink into the vendored clone at `~/.agent-reach/vendor/last30days-skill/skills/last30days`. It resolves for every project, so this repo and its sibling list share one copy; update it with `git -C ~/.agent-reach/vendor/last30days-skill pull`.

```bash
SKILL=~/.agent-reach/vendor/last30days-skill/skills/last30days
python3 "$SKILL/scripts/last30days.py" doctor          # which sources are alive, and how to fix the dead ones
python3 "$SKILL/scripts/last30days.py" "TypeSafe Jev" --days 7 --emit json --output /tmp/sweep.json
```

Then read `/tmp/sweep.json` and pull `url` / `title` per source. Live out of the box on this machine: reddit, youtube (via yt-dlp), hackernews, polymarket, github (via `gh`). X is unconfigured until `setup --allow-browser-cookies` runs, so keep using `twitter search` for X in the meantime.

**What it reaches that the manual sweep cannot.** Two classes show up here and nowhere else:

- **Unmerged integrations.** A pull request or an issue that adds Jev to an existing project is invisible to `gh search repos` and `gh search code`, because the code is not on the default branch and the repository never matches. `open-orcha/orcha#253`, `wso2/api-platform#3517`, `ahstn/oceans-llm#387` and `fakoli/anvil#238` all arrived this way — five third-party projects integrating Jev that the name-based searches had missed across a dozen prior passes.
- **Discussion in places nothing indexes.** Small-subreddit threads, conference talks, personal blogs. The sharpest critique of the calibration claim came from r/ArtificialInteligence, and the design rationale from a Diogo Almeida talk — neither was reachable from a repository search.

Treat it as a complement, not a replacement: it is strong on recency and breadth, while `gh search` remains better for exhaustive repository coverage. Run both, and reconcile.

## Promotion workflow

Use this exact ladder:

1. **Discussion lead found**
   - Add to `categories/related-practices-discussions.md` if it is credible and directly Jev-related.
2. **Evidence chain search**
   - Look for repo, project page, case page, or blog post.
3. **Promotion test**
   - Promote only if public evidence clearly shows a real Jev decision in use.
4. **Promote**
   - Move it into the best-fit main category.
5. **Deduplicate**
   - Remove the weaker discussion-only item if the main case now covers it.
6. **Refresh counts**
   - Run `scripts/build-readme.py` if category totals changed.

## Entry-writing rules

### Main categories

Format:

```md
- [Name](URL) - Industry: one-sentence description of the Jev use case.
```

Rules:

- one sentence only
- must mention the decision + the typed shape (`Choice` / `Score` / `Boolean`) + the gate or escalation
- prefer concrete verbs like `classifies`, `routes`, `gates`, `scores`, `verifies`
- include a number when the source provides one
- avoid hype

### Discussions page

Format:

```md
- [Name or thread title](URL) - Source/platform: one-sentence description of the Jev-related practice or discussion.
```

Rules:

- keep it factual
- describe the practice signal, not your opinion
- if it is mostly about transfer of the pattern, say that clearly

## Periodic maintenance loop

When invoked for a recurring sweep:

1. Read the current category files.
2. Search for 3-10 new public leads.
3. Filter aggressively.
4. Add only high-signal entries.
5. Attempt promotion for the strongest discussion leads.
6. Remove duplicates.
7. Recount category totals.
8. Refresh `README.md` so the homepage aggregate matches the current category files and counts.
9. Publish one short X post covering the round, following the 统一格式 in `/skill:tweet-posting-cdp` (same layout for both awesome lists), then summarize what was added, promoted, and rejected.
   - Verify with `DRY_RUN=1` first; the script prints `内容校验通过：N 字符 / M 段落` and must match before actually posting.

## Suggested commands

Count entries:

```bash
python3 scripts/build-readme.py && sed -n '/## Current coverage/,/Each entry lives/p' README.md
```

Example searches:

```bash
bb-browser site twitter/search 'jev typesafe' --json
bb-browser site google/search 'site:github.com "typesafe-ai/jev" OR "jev" typesafe | sed -n "1,120p"'
bb-browser site google/search 'site:news.ycombinator.com jev typesafe | sed -n "1,120p"'
bb-browser site google/search 'site:reddit.com jev typesafe model | sed -n "1,120p"'
opencli gh api repos/typesafe-ai/jev/readme
```

## Quality bar

**Promote slowly. Add discussions faster.**

If evidence is good but not strong enough for a main case, keep it in discussions.
Precision beats coverage. This model shipped in September 2026, so the corpus is small — resist padding it.

Before adding a repository, check depth (see CONTRIBUTING.md, "AI-assisted work, and bulk submissions"):

- Does the code actually call the Jev API, or is the claim README-only?
- Is there a runnable check (test, example with expected output, public demo)?
- Is the repository mostly prompt documents? Then describe it as such, or skip it.
- Does a batch of same-day repositories share one scaffold? That is a risk signal, not momentum — review each on its own merits, and take at most three entries per author in a single pass, queuing the rest.
- Are the numbers in the entry traceable to the linked page? Strip what you cannot verify.

## Handling pull requests

When a contributor opens a PR:

1. Verify the repository, do not just read the PR body — file tree, source, tests, CI, commit history, and whether the code really calls the Jev API.
2. If the repository is thinner than the entry implies (README-only claims, no runnable check, docs outweighing code), say so in a review comment and ask for the missing evidence rather than merging or silently rejecting it.
3. When an author already has three entries accepted in the current pass, state the rule in the PR comment *before* acting, and queue the extra entries to a later pass instead of rejecting them. Queued entries count against the pass that takes them, not the one that queued them, so honour the queue even when it lands the author a fourth or fifth entry inside the same week. Link the CONTRIBUTING section so the contributor knows it is policy, not a judgement about them. Queue by submission order, never by quality — picking "the best three" turns the rule into a hidden quality verdict, which is exactly what it exists to avoid. If one of the queued entries has more stars than the merged ones, say so explicitly in the comment.
Keep a PR reply short: the rule in one line, what happens next in one line, then stop. No restating the contributor's work, no explaining the queue's arithmetic, no advice they did not ask for. 4. **Bulk AI-generated submissions are a rule, not a preference** (CONTRIBUTING → "Rule: do not submit bulk drops of AI-generated projects"). Before merging a cluster of same-author, same-day repositories, check for a shared scaffold (`AGENTS.md` / `CLAUDE.md` / `STATE.md`), a common README template, or single-commit histories — those mark one project family, which counts as **one** submission, so merge at most one and queue the rest. Disclose-or-pause also applies: if a submission was generated and not disclosed, hold the author's whole pending batch until they say so.
5. Never edit a contributor's wording purely for style; do fix inaccurate claims.
6. README.md is generated — when a PR conflicts there, rebase the branch and regenerate it (`python3 scripts/build-readme.py`) rather than hand-resolving. For conflicts in a category file, keep main's version and append the PR's new line.
8. **A PR that rewrites an existing entry deserves more scrutiny than one that adds a new one**, not less. The contributor usually knows the project better than the catalogue does. Rule 6 in the closing list below came from exactly this: `jev-chat` had long said the Android service reads "WeChat, QQ, X, or Feishu", a contributor changed it to "QQ, X, Feishu/Lark or WhatsApp", and a file count settled it - zero files mentioning wechat or weixin, forty-five mentioning whatsapp. The catalogue had been wrong for an unknown number of rounds, and the sweep posts repeated it.
7. An entry written **only** into `README.md` (no category file) will be silently erased by the next generator run — the merge is clean, CI is green, the count is unchanged, and the entry is gone. Do **not** close the PR and apply it to main by hand: that shows the contributor a closed-unmerged PR and puts their contribution on your commits. Put the line into the right category **on the PR branch** and merge the PR:
   ```bash
   RELOCATE_INTO=infra-sdks-integrations.md scripts/maintainer/merge-prs.sh <PR>
   ```
   The script relocates the line, normalises the separator (`ENTRY_RE` accepts only ` - `, but hand-written README lines use `:` or an em dash), regenerates the README, commits on the branch and pushes it back. It refuses to merge without `RELOCATE_INTO`, because choosing a category is an editorial call, not a guess. When the contributor has not enabled maintainerCanModify the push fails, and the script falls back to a **local** merge whose message is exactly `Merge pull request #N from fork/branch` — GitHub associates that commit with the PR, so the PR still reads as merged and the contributor's commit is really in main. Verify afterwards with `python3 scripts/build-readme.py && git diff --quiet`: no drift means the entry came from the category file and not from README.
   Do this even when the PR's own wording leads with a competitor's model: `Bud-Decision-Studio` led with Laya and Kev, but the repo calls itself "the LM Studio of Jev-like System One models" and runs Jev-Omni, so it was on topic and worth the extra step.
8. Use `scripts/maintainer/merge-prs.sh` to process PRs in bulk. It handles the four traps that make manual merging tedious: fork repository names vary (`awesome-jev`, `awesome-jev-yibie`, `yibie_awesome-jev`, `awesome-jev-1`), so it resolves them via `headRepository.nameWithOwner`; it fast-forwards local `main` before rebasing, because rebasing onto a stale base leaves the PR conflicting after the push; it loops over conflict rounds, because a PR with two commits (add the entry, then reword it) conflicts twice; and it appends the PR's entry with `grep -qF --` so an entry starting with `- [` is not parsed as a grep option.
9. A PR that edits only a category file merges cleanly and leaves the README count stale, so `merge-prs.sh` rebuilds the README after every merge and pushes a follow-up commit when it drifted.
10. Every PR touches the adjacent count lines in README.md, so from the second PR onward conflicts are normal, not a sign of a bad contribution. Expect to rebase and force-push to the contributor's branch (most forks have `maintainerCanModify: true`) rather than asking them to resolve it.

## Deliverable checklist

Before finishing, verify:

- entries are one sentence
- no generic classifiers or judges slipped in
- promoted items have stronger evidence than discussion-only items
- discussions page remains useful as a map of emerging practice
- README homepage aggregate matches the current category files

## Recommended invocation phrases

This skill should be used for prompts like:

- "继续搜集 awesome-jev"
- "做一轮 Jev 证据巡检"
- "把 discussions 里强条目升格"
- "更新 jev awesome list"
- "定期维护这个仓库"

## 两个已固化的操作规则（2026-10-21）

### 1. 农场黑名单：同描述 ≥3 次即整类不收

同一段描述在 7 天内出现 ≥3 次（账号名多为 `FirstnameLastname+数字`）→ 判定为农场，**整类不收**，并记进下表；后续轮次**不再逐个核实**，只在摘要里写一句「农场又出现 N 个」。

| 描述开头 | 首见 | 规模 | 处理 |
|---|---|---|---|
| `Don't trust an autoresearch paper at face value`（Anti-Autoresearch） | 2026-10-14 | 24+ | 只收 1 条（已收），其余不收 |
| `Unofficial skill that teaches coding agents to build with TypeSafe Jev`（building-with-typesafe-jev） | 2026-10-10 | 24+ | 原项目 `aaddrick/building-with-typesafe-jev` 已收，克隆一律不收 |

判断口径：**「同描述」比「同账号」可靠** —— 农场会换账号，但不换文案。

### 2. 数条目只读 README 自己的计数行

三个生成器的输出格式不同，而且有两个**成功时不打印**：

| 仓库 | 读法 |
|---|---|
| awesome-rea | 生成器会打印 `README.md written: N entries across M categories` |
| awesome-jev / awesome-autoresearch | 生成器**静默** → 读 README 的 `## Current coverage` 段里的 `— N entries` |

**不要用统一正则**（已因此出错三次：`/tmp` 缓存被清、rea 的表格格式、jev 静默）。

### 3. 「AI 提及数」的正则陷阱

`\bAI\b` **会匹配 `aim`**（`A` 前是边界、`I` 后接 `m` 也是边界）。统计 AI 参与时用：

```bash
grep -icE '\b(ai|llm|agent|claude|gpt|copilot)\b' README.md | ...
grep -oicE '\bai\b' README.md   # 还要人工看一眼是否全来自 aim/aisle 之类
```

判断「AI 是否参与」不能只靠计数，**要看命中行的上下文** —— 一次把 `aim` 当 AI、一次把「in-game AI opponents」当 AI 参与，都是这条规则要防的。

### 供应链检查（2026-10-23 加，来自 issue #14）

条目指向的仓库，若**源码树里直接提交了可执行文件或压缩包**（`.zip .exe .msi .dll .bat .ps1 .scr`），尤其是这几条同时成立时，极可能是**诱饵仓库**（把恶意二进制塞进同名 fork 或仿冒项目）：

- 项目本身是 Python / JS / Go 之类**不需要二进制**的；
- README 让读者「Run anyway」跳过 SmartScreen 之类的警告；
- 下载链接指向 `raw.githubusercontent.com` 而不是 GitHub Releases；
- 仓库创建时间**晚于**同名项目；
- README 的安装步骤与项目描述对不上（例如一个打分引擎要求你装 Windows 应用）。

**处置**：不收录该仓库，去找原始项目替换，并在 issue/PR 里说明核实过什么。

核查成本很低：

```bash
gh api "repos/OWNER/REPO/git/trees/HEAD?recursive=1" \
  --jq '[.tree[] | select(.type=="blob") | select(.path|test("\\.(zip|exe|msi|dll|bat|ps1|scr)$";"i"))] | length'
```

返回 `0` 才算干净。**新增条目一律跑这一条**；命中的再逐条人看，不要自动判死（有些项目确实提交测试用的小二进制）。

### 解冲突必须按行，不能整份文件取 theirs

2026-10-23 在 awesome-rea 上犯过一次：合并一个只加了一条条目的 PR 时，冲突发生在分类文件上，我用了 `git checkout --theirs <分类文件>` —— 整份文件被换成 PR 分支的旧版本，**PR 分支创建之后新增的条目被静默删掉一条**（Precursor），而条目总数只从 38 变 38，看数字发现不了。

**正确做法**：

1. 分类文件冲突 → **保留 ours**，再把 PR 新增的那一行**按行追加**（先 grep 是否已存在，避免重复）；
2. README.md 冲突 → 一律**重新生成**（`python3 scripts/build-readme.py`），不要手解；
3. 合并后**逐条核对**：`git diff HEAD~1 -- categories/ | grep '^-'` —— 任何消失的条目行都必须解释或恢复；
4. **条目数只增不减**是最快的自检：若 `N` 没变而合并的是一次新增，就是丢东西了。

awesome-jev 有 `scripts/maintainer/merge-prs.sh` 按行解冲突（`resolve_round` + `added_line_for`）；awesome-rea 与 awesome-autoresearch **没有**这个脚本，所以这两个仓库手工合并时更要照上面四条做。

### 同源项目只交代一次（2026-10-30）

同一组织或同一作者的一组项目，若共用同一骨架，**只在最早进目录的那条里交代家族**，其余条目保持单项目视角。

**理由**：三条都写「与 X、Y 同源」等于同一句话出现三次，读者还得自己拼；写一次在最早那条，等于在人物第一次出场时介绍关系。

**实例**：`AgonAlpha`（量化 alpha）、`Agon`（科学循环）、`AutoSR`（符号回归）都来自 AutoResearch-Factory，共用「proposer 写候选 → 独立 auditor 打分 → 留还是弃」的骨架。家族关系写在 `AgonAlpha` 那条里，另两条不重复。

**判断「共用骨架」时别只看调度器**：这三者里两个用 MCTS、一个用 modify→verify 循环；真正共同的是那个**独立审查者**。条目里要写的是后者，因为那才是这家组织认定的核心。

**例外**：若后来那两条之一**升级为不同骨架**（不再是同一套），就把家族从句子里拆掉，各自独立描述。
