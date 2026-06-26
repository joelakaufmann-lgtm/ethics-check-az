---
name: azbar-ethics-navigator
description: >
  Retrieval/navigation layer for State Bar of Arizona ethics materials on
  azbar.org. Gets clean verbatim text of Arizona ethics opinions and the Rules
  of Professional Conduct (the "ER" Ethical Rules) even though the public site is
  JavaScript-rendered. Knows the server-rendered tools.azbar.org endpoints
  (ViewEthicsOpinion.aspx, ViewRule.aspx), the ER→RuleId and opinion-number→id
  maps, and how to avoid its browser-freeze, truncation, and privacy-filter
  traps. Use when you need the text of an Arizona ethics opinion (e.g., Op.
  97-09) or an Ethical Rule (e.g., ER 1.6), to search opinions by year/topic, or
  to (re)build the opinion/rule index. Pairs with ethics-check-az (analysis vs.
  fetching). TRIGGERS: azbar, azbar.org, Arizona Bar website, fetch Arizona
  ethics opinion, Arizona ethics opinion text, pull ER text, Rule 42 text,
  ViewEthicsOpinion, ViewRule, harvest azbar opinions, Arizona ethics opinion by
  number or year, "the bar site won't load."
---

# Arizona State Bar Ethics — Site Navigator / Fetcher (azbar-ethics-navigator)

This skill is the **data-retrieval layer** for Arizona professional-responsibility materials published by the State Bar of Arizona. It encodes a reverse-engineered, tested playbook for getting **clean verbatim text** of (1) Arizona **ethics opinions** and (2) the **Rules of Professional Conduct** (ER 1.0–8.5, housed in Ariz. R. Sup. Ct. 42), plus the unauthorized-practice (UPL) opinions and the post-2016 Supreme Court Ethics Advisory Committee opinions.

It exists because `www.azbar.org` is **client-side (JavaScript) rendered**: a plain `web_fetch` of any opinion or rule page returns only navigation chrome and a ~200-language Google-Translate list — never the content. The fix is to use the State Bar's **server-rendered** companion host and a small set of freeze-safe browser procedures, documented below.

> **Pairing:** Use `ethics-check-az` for the *analysis* of an Arizona ethics question. Use **this** skill whenever you actually need to pull the *text* of an opinion or rule (because `ethics-check-az`'s `opinions-full.md` is a curated subset, and some ER text in its `rpc.md` is by-reference).

## The one thing to remember

**Opinions and most rules have clean, server-rendered URLs on `tools.azbar.org`. Fetch those — do NOT fetch the `www.azbar.org` opinion/rule pages.**

| Need | URL (server-rendered; `web_fetch` works) | Reliability |
|---|---|---|
| Full text of an **ethics opinion** | `https://tools.azbar.org/RulesofProfessionalConduct/ViewEthicsOpinion.aspx?id=<OpinionId>` | **High** — clean FACTS/QUESTIONS/APPLICABLE RULES/OPINION/CONCLUSION + footnotes |
| Full text of an **Ethical Rule** | `https://tools.azbar.org/RulesofProfessionalConduct/ViewRule.aspx?id=<RuleId>` | **Mixed** — returns full rule + comments for many ERs, but **empty** for some (see below); retry, then fall back to Chrome |
| UPL opinion (inferred, confirm) | `https://tools.azbar.org/RulesofProfessionalConduct/ViewUPLOpinion.aspx?id=<id>` | **Unconfirmed** — verify the exact path before relying |

The `<OpinionId>` and `<RuleId>` are scattered internal ids (not sequential, not the opinion/rule number). The maps are:
- **ER → RuleId:** `references/rule-id-map.md` (complete, all 59 entries).
- **Opinion number → OpinionId:** the **`Src` column of `ethics-check-az/references/opinions-index.md`** (all 297 State Bar opinions, 1985–2016). If `ethics-check-az` is not available, rebuild the map via the year-search procedure in `references/harvest-playbook.md`.

## Quick recipes

### A. "Get me the text of Arizona Ethics Op. NN-NN"
1. Look up its `OpinionId` (`Src`) in `ethics-check-az/references/opinions-index.md`.
2. `web_fetch https://tools.azbar.org/RulesofProfessionalConduct/ViewEthicsOpinion.aspx?id=<Src>`.
3. You get the full verbatim opinion. (Check the index for "withdrawn/superseded" flags; opinions apply the ERs in effect on their date.)

### B. "Get me the text of ER X.Y"
1. Look up the `RuleId` in `references/rule-id-map.md`.
2. `web_fetch https://tools.azbar.org/RulesofProfessionalConduct/ViewRule.aspx?id=<RuleId>`.
3. If it returns **empty** (some rules do — see the map's "JS-only" flags), **retry once** (occasional transient empties), then fall back to Chrome (`references/harvest-playbook.md` § Rule fallback) or to `research-assistant` / Westlaw Arizona Court Rules (Rule 42).

### C. "Find opinions on <topic>" or "list opinions from <year>"
- Fastest: skim the topic cross-reference and table in `ethics-check-az/references/opinions-index.md`, then fetch the chosen opinions via recipe A.
- Live search (if the index is unavailable or you want the keyword search): use the Chrome year/keyword procedure in `references/harvest-playbook.md`.

### D. "Harvest many opinions / (re)build the index"
- Use `web_fetch` on `ViewEthicsOpinion.aspx?id=<Src>` in **parallel batches of ~6**. The opinion endpoint tolerates concurrency well.
- These opinions can be long; each one consumes context. Harvest in passes, write to disk incrementally, and track the last id done so a later pass resumes cleanly.

### E. Post-2016 opinions (Supreme Court Ethics Advisory Committee)
The State Bar stopped issuing formal opinions after 2016. Current opinions come from the **Arizona Supreme Court Ethics Advisory Committee** at `azcourts.gov/cld` (e.g., `EO-19-0009`, `EO-20-0003`). These are typically **PDFs on azcourts.gov** and are usually `web_fetch`-able directly. Treat 1985–2016 State Bar opinions as persuasive historical guidance subject to rule drift.

## Traps that make this site hard (and how to avoid them)

1. **Don't `web_fetch` the `www.azbar.org` opinion/rule pages.** They are JS-rendered; you'll get nav + the language list, not content. Use `tools.azbar.org` instead.
2. **Browser freeze on "render everything."** In Chrome, calling the opinion search with **"All Years"** selected, or calling **`GetRules()` / `ShowRules()`** on the rules page, makes the tab render hundreds of items and the CDP evaluate call **times out (45s) / the renderer freezes.** Never do those. Go **per year** (opinions) and **per section** (rules). See the playbook.
3. **`javascript_tool` truncates returns at ~1.1KB.** To pull bulk data out of a page, either (a) inject it into a single DOM element and read with **`get_page_text`** (which does **not** truncate), or (b) retrieve in **parallel small slices** (`arr.slice(a,b)` per call). The playbook shows both.
4. **Privacy filter: `[BLOCKED: Cookie/query string data]`.** `javascript_tool` blocks any return value containing URL/query-string/cookie-like text. **Return plain text, integers, and sanitized tokens only** — strip `href`s and `onclick` strings (extract just the numeric id), and don't echo full URLs back.
5. **Concurrency empties on `ViewRule.aspx`.** Firing many rule fetches at once raises the empty-response rate. If a rule comes back empty, retry it singly before concluding it's JS-only.

## Hard rules
- **Never fabricate opinion or rule text.** If the endpoint won't return it, say so and fall back (Chrome → `research-assistant`/Westlaw → flag the gap). Quote only text you actually retrieved.
- **Preserve verbatim text and citation markers** (opinion numbers, years, "© State Bar of Arizona 20XX", footnotes, "withdrawn/superseded" notes).
- **Confirm currency.** Rule text on `ViewRule.aspx` is current; opinions apply the rules in effect on their publication date — note rule drift (esp. the Dec. 1, 2003 ER framework and the 2021 ER 5.4 repeal / ABS / ER 7.x rewrite).

## Reference files
| File | Contents |
|---|---|
| `references/rule-id-map.md` | Complete ER → `RuleId` table for `ViewRule.aspx`, with per-rule reliability ("clean" vs. "JS-only/empty"). |
| `references/harvest-playbook.md` | Step-by-step Chrome procedures (per-year opinion search; per-section rule sweep), the DOM-injection bulk-retrieval trick, privacy-filter-safe scraping, and the EAC/UPL notes. |
