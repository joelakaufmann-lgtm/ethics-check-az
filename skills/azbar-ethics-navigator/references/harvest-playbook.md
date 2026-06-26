# Harvest Playbook — azbar.org ethics opinions & rules

Concrete, tested procedures. Prefer `web_fetch` against `tools.azbar.org` (server-rendered) for **content**; use **Claude in Chrome** only for **discovery** (building number→id maps, keyword/year search) because those live functions only run on the JS site.

Browser tools referenced: `mcp__Claude_in_Chrome__navigate`, `…__get_page_text`, `…__javascript_tool`, `…__find`, `…__browser_batch`. Load via ToolSearch if deferred.

---

## 1. Opinion full text (primary path — no browser)

```
web_fetch https://tools.azbar.org/RulesofProfessionalConduct/ViewEthicsOpinion.aspx?id=<OpinionId>
```
- `<OpinionId>` = `Src` column in `ethics-check-az/references/opinions-index.md`.
- Returns clean verbatim text (synopsis, FACTS, QUESTIONS PRESENTED, APPLICABLE RULES, RELEVANT OPINIONS, OPINION, CONCLUSION, footnotes, "© State Bar of Arizona 20XX").
- **Bulk harvest:** parallel batches of ~6 are reliable. Opinions can be long (some >10 KB); each consumes context, so harvest in passes, write to disk incrementally, and record the last id done.

## 2. Rule full text (primary path — no browser)

```
web_fetch https://tools.azbar.org/RulesofProfessionalConduct/ViewRule.aspx?id=<RuleId>
```
- `<RuleId>` from `rule-id-map.md`. Returns the rule + official comments.
- **If empty:** retry once (transient under concurrency). Still empty ⇒ that rule is JS-only; use §5 (Chrome rule fallback) or `research-assistant`/Westlaw Arizona Court Rules (Rule 42). **Never reconstruct rule text from memory.**

---

## 3. Build / refresh the opinion number→id map (Chrome, per-year — freeze-safe)

The opinion search lives only on the JS page. **Do NOT select "All Years"** — it renders all ~297 results at once and freezes the tab.

Setup: `navigate` a tab to `https://www.azbar.org/for-legal-professionals/ethics/ethics-opinions/`.

Then run this `javascript_tool` (≤ ~7 years per call to stay under the 45 s evaluate timeout; accumulate into a page global so partial progress survives a freeze):

```js
window.__AZ = window.__AZ || [];
const seen = new Set(window.__AZ.map(x=>x.id));
const ys = document.querySelector('#lstYears');
async function doYear(y){
  ys.value = String(y); ys.dispatchEvent(new Event('change',{bubbles:true}));
  Search('Year', null);                       // site function; renders into #divSearchResults
  await new Promise(r=>setTimeout(r,1500));   // CORSLoad latency; 1.3–1.8s
  const d = document.querySelector('#divSearchResults'); if(!d) return;
  d.querySelectorAll('a').forEach(a=>{
    const oc = a.getAttribute('onclick')||'';        // e.g. ShowOpinion(726)
    const id = (oc.match(/(\d{1,6})/)||[null])[0];
    if(!id || seen.has(id)) return;
    const t = a.textContent.trim();                  // e.g. "16-01: Of Counsel; Fee Splitting"
    const num = (t.match(/^([0-9]{2}-[0-9]{2,3})/)||[null])[0];
    const title = t.replace(/^[0-9]{2}-[0-9]{2,3}:?\s*/,'').replace(/\s+/g,' ').trim();
    seen.add(id); window.__AZ.push({y, num, id, title});
  });
}
for (let y=1985; y<=1991; y++){ await doYear(y); }   // do the next chunk in the next call
window.__AZ.length;                                   // return a COUNT only (privacy-safe)
```
Repeat for the remaining year chunks (…1992–1998, 1999–2004, 2005–2010, 2011–2016). State Bar opinions span **1985–2016** (no opinions in 2014); 2017+ returns nothing here (those are EAC opinions — §6).

**Get the accumulated data out** (the result is too big for one `javascript_tool` return, which truncates ~1.1 KB) — inject into the DOM and read with `get_page_text` (no truncation):
```js
// build rows however you like, then:
document.body.innerHTML = '<pre>' +
  window.__AZ.map(x=>`${x.id}|${x.num}|${x.title}`).join('\n')
    .replace(/&/g,'&amp;').replace(/</g,'&lt;') + '</pre>';
'ok';
```
…then call `get_page_text` on that tab and parse the rows. (Alternative: many parallel `javascript_tool` calls each returning `window.__AZ.slice(a,b)` in <1 KB chunks.)

## 4. Build / refresh the ER→RuleId map (Chrome, per-section — freeze-safe)

`navigate` to `https://www.azbar.org/for-legal-professionals/ethics/rules-of-professional-conduct/`. **Do NOT call `GetRules()` / `ShowRules()`** (renders all rules → freeze). Sweep section by section instead:

```js
window.__RULES = {};
function grab(){
  document.querySelectorAll('[onclick]').forEach(el=>{
    const oc = el.getAttribute('onclick')||'';
    if(!/RuleDetails|OpenRule/i.test(oc)) return;     // ShowRuleDetails(<RuleId>)
    const id = (oc.match(/(\d{1,4})/)||[null])[0];
    const rm = el.textContent.match(/(\d\.\d+[A-Za-z]?)/);  // "1.6"
    if(id && rm) window.__RULES[rm[1]] = id;
  });
}
for (let s=1; s<=8; s++){ try{ ShowRulesBySection(s); }catch(e){} await new Promise(r=>setTimeout(r,1800)); grab(); }
Object.keys(window.__RULES).length;   // count only
```
Sections: 1 Client-Lawyer Relationship · 2 Counselor · 3 Advocate · 4 Transactions w/ Non-Clients · 5 Law Firms & Associations · 6 Public Service · 7 Information About Legal Services · 8 Integrity of the Profession (+ "Preamble & Terminology"). Retrieve `window.__RULES` via the DOM-injection trick above. (The current map is already in `rule-id-map.md`; only re-run if the Bar renumbers.)

## 5. Chrome rule fallback (when `ViewRule.aspx` is empty)

`ShowOpinion(id)` and the rule detail views also open clean deep links on the JS site that render after JavaScript runs. Two options:
- Open `https://www.azbar.org/for-legal-professionals/ethics/rules-of-professional-conduct/?V=Rules`, run `ShowRulesBySection(<section of the rule>)`, then invoke the specific `ShowRuleDetails(<RuleId>)`, wait, and read with `get_page_text`.
- Or, for opinions, `ShowOpinion(<id>)` opens `?V=Opinions&OpinionId=<id>`; `get_page_text` on that rendered tab returns the full opinion (plus boilerplate you can ignore). Prefer the `tools.azbar.org` fetch over this whenever possible.

To strip the boilerplate (the ~200-language Translate list + nav) before `get_page_text`, delete those nodes first, or just slice the returned text between the opinion/rule heading and "© State Bar of Arizona".

## 6. Post-2016 opinions — Supreme Court Ethics Advisory Committee (EAC)

Not on azbar.org. Current Arizona formal opinions come from the **Arizona Supreme Court Ethics Advisory Committee**: `https://www.azcourts.gov/cld/Ethics-Advisory-Committee` (and `…/Ethics-Opinion-Requests`). Opinions are numbered like `EO-19-0009`, `EO-20-0003` and published as **PDFs on azcourts.gov**, which are generally `web_fetch`-able directly. Search the azcourts.gov ethics pages or `WebSearch` `site:azcourts.gov EO-` for the list.

## 7. UPL opinions

Separate State Bar series at `https://www.azbar.org/for-legal-professionals/ethics/upl-opinions/` (JS-rendered, same search mechanics as ethics opinions). Individual opinions are likely served from `tools.azbar.org/RulesofProfessionalConduct/ViewUPLOpinion.aspx?id=<id>` **(path inferred from the ethics-opinion analogue — confirm before relying).** Enumerate via the §3 year-search procedure on the UPL page.

---

## Gotcha reference card
- **`web_fetch` on `www.azbar.org` content pages → useless** (JS-rendered; returns nav + language list). Use `tools.azbar.org`.
- **Freeze triggers:** "All Years" opinion search; `GetRules()`/`ShowRules()`. → go per-year / per-section.
- **`javascript_tool` truncates ~1.1 KB.** → DOM-inject + `get_page_text`, or parallel slices.
- **`[BLOCKED: Cookie/query string data]`** = your return value contained URL/query-like text. → return ids/plain text only; strip hrefs and `onclick` strings (extract the numeric id).
- **Frozen tab:** the JS global you were filling (`window.__AZ`, `window.__RULES`) usually survives; re-read it with a light call, or open a fresh tab and continue.
- **Concurrency empties on `ViewRule.aspx`:** retry singly.
- **Don't fabricate.** Empty/blocked ⇒ fall back or flag; never invent opinion or rule text.
