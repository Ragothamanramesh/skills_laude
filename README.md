# skills_laude

A personal Claude Code plugin marketplace: 8 skills picked from the
[GitHub "Claude" Top 100](https://github.com/yuxiaopeng/Github-Ranking-AI/blob/main/Top100/Claude.md),
each security-reviewed and pinned to the exact commit that was reviewed.

Claude picks these up automatically by context. Each skill's description is
always visible to Claude, and the full skill loads only when a request matches.

| Plugin | What it's for | Skills | Hooks | Always-on cost | Pinned to |
|---|---|---|---|---|---|
| andrej-karpathy-skills | Coding guidelines against common LLM mistakes | 1 | 0 | ~105 tok | multica-ai/andrej-karpathy-skills@2c60614 |
| planning-with-files | Markdown plans that survive `/clear` and compaction | 14 | 6 | ~1,129 tok | OthmanAdi/planning-with-files@0d840dc |
| hallmark | Anti-"AI slop" UI design | 1 | 0 | ~101 tok | Nutlope/hallmark@04977e1 |
| frontend-slides | HTML slide decks | 1 | 0 | ~125 tok | zarazhangrui/frontend-slides@9906a34 |
| last30days | Research a topic across Reddit, X, YouTube, HN, web | 1 | 1 | ~105 tok | mvanhorn/last30days-skill@e63f8e8 |
| humanizer | Strip AI-writing tells from text | 1 | 0 | ~124 tok | blader/humanizer@a32f244 |
| marketing-skills | Copywriting, CRO, SEO, email, ads, A/B tests (vendored, trimmed 53 → 16) | 16 | 0 | ~2,313 tok | coreyhaines31/marketingskills@5b2b400 |
| skill-inspector | Scan a skill for security risks before installing (vendored) | 1 | 0 | ~99 tok | NVIDIA/SkillSpector@48e0f94 |
| **Total** | | **36** | **7** | **~4,100 tok** | |

## Install

On your own machine:

```bash
claude plugin marketplace add Ragothamanramesh/skills_laude
```

Then install all eight:

```bash
for p in andrej-karpathy-skills planning-with-files hallmark frontend-slides last30days humanizer marketing-skills skill-inspector; do claude plugin install "$p@curated-skills"; done
```

## Auto-enable in another repo (cloud sessions included)

Add this to that repo's `.claude/settings.json`. Cloud sessions on that repo
will then prompt to install the marketplace and these plugins.

See [`settings-snippet.json`](settings-snippet.json).

## Security review (Nov 2026)

Each plugin was scanned with NVIDIA SkillSpector (static, `--no-llm`) and every
HIGH/CRITICAL finding was read in source.

| Plugin | Scanner verdict | After manual review |
|---|---|---|
| andrej-karpathy-skills | SAFE (0) | Approve. One markdown file. |
| skill-inspector | CAUTION (37) | Approve. Instruction file only; flags are its own descriptions of attacks. |
| humanizer | CAUTION (49) | **Approve.** Both HIGH hits are the phrase "announced without warning" in a before/after writing example. |
| marketing-skills (16) | 12 SAFE, 1 CAUTION, 2 DO_NOT_INSTALL | **Approve.** ad-creative: "arsenic" is lab-result copy in a supplement-ad example. ads: "swap rules" are ad-rotation rules, "don't judge" means don't judge a campaign too early. cold-email: prospects "almost always respond". |
| hallmark | DO_NOT_INSTALL (100) | **Approve.** `169.254.169.254` sits in an SSRF *blocklist*; the `~/.codex/config.toml` line says *never* edit it; "keyring" hits are inside a photo catalog JSON; "return rules" is a lint script. Calls Together AI only if `TOGETHER_API_KEY` is set and you ask for generated images. |
| frontend-slides | DO_NOT_INSTALL (100) | **Approve.** "Never use Caveat" refers to the Caveat *font*; `&& rm -` is temp-file cleanup in an optional deploy script. |
| planning-with-files | DO_NOT_INSTALL (100) | **Approve with caution.** No network code. Hooks run local shell/Python on every prompt and tool call; "extract conversation" reads your local session log for the explicit catch-up command only. Pause with `PLANNING_DISABLED=1`. |
| last30days | DO_NOT_INSTALL (100; 247 HIGH) | **Approve with caution.** Makes network calls to Reddit, X, YouTube, HN and others by design. Browser-cookie reading is off unless you set `BROWSER_CONSENT`/`AGENTCOOKIE`, and then reads only named X/Reddit cookies. Decline it. The 218 ".env" hits read only its own `~/.config/last30days/.env` and a trusted `.claude/last30days.env`, never your project `.env`; env-var loops build a redaction list and a subprocess allowlist. |

Notes:
- **last30days** asks before reading browser cookies for X, and will offer a
  paid ScrapeCreators signup. Both are optional; it works keyless with fewer
  sources. Its Stop hook is stdlib-only and saves briefs locally.
- **marketing-skills** kept skills sometimes point to dropped ones
  (customer-research, product-marketing, conversion-tracking). founderos covers
  those topics.
- **skill-inspector** needs the CLI:
  `uv tool install --python 3.13 git+https://github.com/NVIDIA/skillspector.git`

## Updating

Pins are deliberate. To update a plugin, re-run the scan on the new commit,
then bump its `sha` in `.claude-plugin/marketplace.json`.
