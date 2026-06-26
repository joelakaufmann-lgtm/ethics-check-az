# Arizona Ethics Check

An Agent Skills-compatible skill for Arizona lawyer ethics, professional responsibility, State Bar discipline-adjacent issues, Arizona ethics opinions, trust-account questions, UPL, multijurisdictional practice, and Alternative Business Structure / Legal Paraprofessional issues.

The repository root is the skill folder: it contains `SKILL.md` plus the bundled `references/` files. That layout is intentionally portable across Codex and Claude skill runtimes that support the common Agent Skills pattern.

## Scope

This skill is calibrated to Arizona authority, including:

- Arizona Rules of Professional Conduct, Ariz. R. Sup. Ct. 42
- Arizona Supreme Court Rules 31-80, including admission, discipline, pro hac vice, trust accounts/IOLTA, and MCLE references
- Arizona lawyer-discipline procedure before the Presiding Disciplinary Judge
- State Bar of Arizona Formal Ethics Opinions from 1985-2016, with a complete opinion index and selected full-text seed opinions
- Arizona UPL advisory-opinion pointers

It is not an ABA Model Rules skill. ABA, Restatement, and other-state authority should be treated only as background unless Arizona authority makes it relevant.

## Companion skill: azbar-ethics-navigator

The [`skills/azbar-ethics-navigator/`](skills/azbar-ethics-navigator/) folder contains a companion **retrieval/navigation skill** for the State Bar of Arizona website. Where `ethics-check-az` does the *analysis*, `azbar-ethics-navigator` does the *fetching*: it pulls clean, verbatim text of Arizona ethics opinions and Rules of Professional Conduct (the "ER" Ethical Rules) from the State Bar's server-rendered endpoints, and documents the working URLs, the ER-to-RuleId and opinion-number-to-id maps, and the browser-freeze / result-truncation / privacy-filter workarounds that otherwise make `azbar.org` hard to scrape.

- Skill instructions: [`skills/azbar-ethics-navigator/SKILL.md`](skills/azbar-ethics-navigator/SKILL.md)
- Reference files: [`skills/azbar-ethics-navigator/references/`](skills/azbar-ethics-navigator/references/) (`rule-id-map.md`, `harvest-playbook.md`)
- Packaged archive: `skills/azbar-ethics-navigator/azbar-ethics-navigator.skill`

Use `ethics-check-az` to reason about an Arizona ethics question; use `azbar-ethics-navigator` when you need the underlying opinion or rule text.

## Install

For Codex:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/joelakaufmann-lgtm/ethics-check-az.git ~/.codex/skills/ethics-check-az
```

For Claude Code or another Claude skill runtime that reads local skills:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/joelakaufmann-lgtm/ethics-check-az.git ~/.claude/skills/ethics-check-az
```

To package the skill as a `.skill` archive from the parent directory:

```bash
zip -r ethics-check-az.skill ethics-check-az \
-x "ethics-check-az/.git/*" \
-x "ethics-check-az/.DS_Store"
```

## Use

Invoke the skill by name:

```text
Use $ethics-check-az to analyze whether an Arizona lawyer may share fees with a nonlawyer-owned Alternative Business Structure.
```

For substantive questions, the skill is designed to quote operative Arizona rule text when available, identify relevant Arizona ethics opinions, flag rule drift, and surface corpus gaps that require current primary-source verification.

## Important Limits

This is not legal advice and does not create an attorney-client relationship. Do not use it as a substitute for current primary-source research, conflict checking, client-specific legal advice, firm policy review, or consultation with qualified ethics counsel.

The bundled corpus is a snapshot. Several reference files are seeded rather than complete full-text corpora. Verify current law before relying on any output, especially for active matters, disciplinary issues, trust-account questions, conflicts, candor-to-tribunal issues, confidentiality exceptions, UPL/MJP, ABS/LP issues, or questions involving imminent action.

Do not enter personal, client, privileged, confidential, or matter-identifying information unless your deployment environment satisfies your professional-responsibility and data-protection obligations.

## License

MIT License. See [LICENSE](LICENSE).
