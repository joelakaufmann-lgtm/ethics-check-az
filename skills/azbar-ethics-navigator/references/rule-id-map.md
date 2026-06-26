# ER → RuleId map for `ViewRule.aspx`

Construct the clean, server-rendered URL for any Ethical Rule as:

> `https://tools.azbar.org/RulesofProfessionalConduct/ViewRule.aspx?id=<RuleId>`

The `RuleId` is an internal id (scattered, not the ER number). This map was extracted from the State Bar rules page (`?V=Rules`) by sweeping each section with `ShowRulesBySection(n)` and reading the `ShowRuleDetails(<RuleId>)` handlers. Verify against the live source if a rule was recently re-numbered.

## Reliability legend
- **clean** = `web_fetch` of the `ViewRule.aspx` URL returned full rule text + comments (tested).
- **empty** = `web_fetch` returned an empty body (tested) — the page renders the rule only via JavaScript. **Retry once** (some empties are transient under concurrency); if still empty, use the Chrome fallback in `harvest-playbook.md` or `research-assistant`/Westlaw.
- **untested** = not fetched during mapping; try `web_fetch` first (most work), then fall back.

| ER | Title | RuleId | Reliability |
|---|---|---|---|
| 1.0 | Terminology | 21 | clean |
| 1.1 | Competence | 3 | untested |
| 1.2 | Scope of Representation & Allocation of Authority | 4 | untested |
| 1.3 | Diligence | 23 | untested |
| 1.4 | Communication | 24 | untested |
| 1.5 | Fees | 25 | clean |
| 1.6 | Confidentiality of Information | 26 | empty (JS-only) |
| 1.7 | Conflict of Interest: Current Clients | 27 | empty (JS-only) |
| 1.8 | Conflict of Interest: Current Clients: Specific Rules | 28 | untested |
| 1.9 | Duties to Former Clients | 29 | empty (JS-only) |
| 1.10 | Imputation of Conflicts of Interest: General Rule | 22 | empty (JS-only) |
| 1.11 | Special Conflicts: Former/Current Gov't Officers | 30 | untested |
| 1.12 | Former Judge, Arbitrator, Mediator, Third-Party Neutral | 31 | untested |
| 1.13 | Organization as Client | 32 | untested |
| 1.14 | Client with Diminished Capacity | 33 | untested |
| 1.15 | Safekeeping Property | 63 | clean |
| 1.16 | Declining or Terminating Representation | 35 | empty (JS-only) |
| 1.17 | Sale of Law Practice | 36 | untested |
| 1.18 | Duties to Prospective Client | 37 | untested |
| 2.1 | Advisor | 7 | untested |
| 2.2 | (Reserved) | 8 | untested |
| 2.3 | Evaluation for Use by Third Persons | 38 | untested |
| 2.4 | Lawyer Serving as Third-Party Neutral | 39 | untested |
| 3.1 | Meritorious Claims and Contentions | 9 | untested |
| 3.2 | Expediting Litigation | 10 | untested |
| 3.3 | Candor Toward the Tribunal | 40 | empty (JS-only) |
| 3.4 | Fairness to Opposing Party and Counsel | 41 | untested |
| 3.5 | Impartiality and Decorum of the Tribunal | 42 | untested |
| 3.6 | Trial Publicity | 43 | untested |
| 3.7 | Lawyer as Witness | 44 | untested |
| 3.8 | Special Responsibilities of a Prosecutor | 45 | untested |
| 3.9 | Advocate in Nonadjudicative Proceedings | 46 | untested |
| 3.10 | Threatening Criminal/Administrative/Disciplinary Action (AZ-specific) | 64 | untested |
| 4.1 | Truthfulness in Statements to Others | 11 | untested |
| 4.2 | Communication with Person Represented by Counsel | 12 | empty (JS-only) |
| 4.3 | Dealing with Unrepresented Person | 47 | untested |
| 4.4 | Respect for Rights of Third Persons | 48 | untested |
| 5.1 | Responsibilities of Partners/Managers/Supervisors (and owners) | 13 | untested |
| 5.2 | Responsibilities of a Subordinate Lawyer | 14 | untested |
| 5.3 | Responsibilities Regarding Nonlawyers | 49 | untested |
| 5.4 | (Abrogated eff. 1/1/2021 — former "Professional Independence of a Lawyer") | 50 | untested |
| 5.5 | Unauthorized Practice of Law; Multijurisdictional Practice | 51 | untested |
| 5.6 | Restrictions on Right to Practice | 52 | untested |
| 5.7 | Responsibilities Regarding Law-Related Services | 53 | untested |
| 6.1 | Voluntary Pro Bono Publico Service | 15 | untested |
| 6.2 | Accepting Appointments | 16 | untested |
| 6.3 | Membership in Legal Services Organization | 54 | untested |
| 6.4 | Law Reform Activities Affecting Client Interests | 55 | untested |
| 6.5 | Nonprofit & Court-Annexed Limited Legal Services | 56 | untested |
| 7.1 | Communications Concerning a Lawyer's Services | 17 | untested |
| 7.2 | Advertising (amended 2021) | 18 | untested |
| 7.3 | Solicitation of Clients (amended 2021) | 57 | untested |
| 7.4 | Communication of Fields of Practice/Specialization (amended 2021) | 58 | untested |
| 7.5 | Firm Names and Letterheads (amended 2021) | 59 | untested |
| 8.1 | Bar Admission and Disciplinary Matters | 19 | untested |
| 8.2 | Judicial and Legal Officials | 20 | untested |
| 8.3 | Reporting Professional Misconduct | 60 | empty (JS-only) |
| 8.4 | Misconduct | 61 | untested |
| 8.5 | Disciplinary Authority; Choice of Law | 62 | untested |

**Pattern note:** the "empty (JS-only)" rules observed so far (1.6, 1.7, 1.9, 1.10, 1.16, 3.3, 4.2, 8.3) are scattered across sections, so there is no shortcut to predict them — just try `web_fetch`, retry once, then fall back. The "clean" confirmations (1.0, 1.5, 1.15) prove the endpoint and the id map are correct; an empty body means *that page's* server rendering is missing, not a wrong id.

**Currency caveats baked into the titles above:** former ER 5.4 was **abrogated effective Jan. 1, 2021** (Arizona now permits nonlawyer fee-sharing/ownership and **Alternative Business Structures** under Ariz. R. Sup. Ct. 31/31.1); the **ER 7.1–7.5** advertising rules were **rewritten in 2021**; **ER 3.10** is an Arizona-specific rule with no ABA analogue. Always quote the live `ViewRule.aspx` text, not these titles.
