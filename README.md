# The Missing 50

An interactive, evidence-based simulation of **UK charity non-donors** — the
roughly half of UK adults who gave nothing to charity in the last 12 months.

It turns a statistic into 50 named people with individual barriers, then lets
you test interventions drawn from published research and see what actually
moves them.

**Live:** _(add the deployed URL here)_

---

## What it does

Four phases, roughly 10–15 minutes end to end:

| Phase | Name | What happens |
| --- | --- | --- |
| 0 | **Hook** | Intro sequence framing the 50% gap |
| 1 | **The Investigation** | Explore 50 personas, their segments and barriers, with sources attached |
| 2 | **Play** | Card game — spend budget on interventions across several turns |
| 3 | **Outcomes** | Results, scaled impact estimate, full card deck, key takeaways |

Built for fundraisers, charity strategists and workshop facilitators. Free to
use and share.

## Architecture

The whole thing is **one self-contained `index.html`** — no build step, no
dependencies, no server, no CDN. Open the file and it runs. The only external
request is Google Analytics.

Inside, the file is organised as clearly delimited sections:

- `DATA` — sources, barriers, segments and intervention cards
- `UNIQUE_PEOPLE` — the 50 personas
- `RNG` — seeded Mulberry32 PRNG, so runs are reproducible
- `MoneyCalc` — scaling from 50 simulated people to UK-wide impact
- `GameState`, `CardSystem`, `HELPERS`
- `Phase0`–`Phase3` modules, then a small router

### Data model

Everything is referentially linked by id:

- a **person** belongs to one `segment` and carries weighted `barriers`
- a **barrier** cites one or more `sources`
- a **card** targets `barriers` and `segments`, and cites `sources`

Current dataset: **43 sources, 11 barriers, 7 segments, 24 cards, 50 people.**

Counts shown in the UI are derived from `DATA` at boot (elements with class
`js-count-sources` / `js-count-cards`), so the copy cannot drift from the data.

## Running locally

```bash
# just open it
open index.html

# or serve it, if you want to test analytics / social previews
npx http-server . -p 8080
```

## Editing the data

Add or change entries inside the `DATA` object near the top of the `<script>`
block. Keep ids stable — they are the only thing linking people, barriers,
cards and sources together.

A quick referential-integrity check:

```js
// paste into the browser console on the running page
const ids = a => new Set(a.map(x => x.id));
const S = ids(DATA.sources), B = ids(DATA.barriers), G = ids(DATA.segments);
const bad = [];
DATA.cards.forEach(c => {
  (c.citations || []).forEach(x => S.has(x) || bad.push(`card ${c.id}: bad source ${x}`));
  (c.target_segments || []).forEach(x => G.has(x) || bad.push(`card ${c.id}: bad segment ${x}`));
  (c.targets || []).forEach(t => B.has(t.barrierTypeId) || bad.push(`card ${c.id}: bad barrier ${t.barrierTypeId}`));
});
UNIQUE_PEOPLE.forEach(p => {
  G.has(p.segment_id) || bad.push(`${p.id}: bad segment ${p.segment_id}`);
  p.barriers.forEach(b => B.has(b.barrierTypeId) || bad.push(`${p.id}: bad barrier ${b.barrierTypeId}`));
});
console.log(bad.length ? bad : 'OK');
```

## Accessibility

- Person avatars are keyboard-operable (Tab to reach, Enter/Space to open) and
  labelled for screen readers
- Visible `:focus-visible` outlines throughout
- `prefers-reduced-motion` is honoured, including the Phase 0 physics loop
- `Escape` closes any open dialog
- `<noscript>` fallback explains the premise when JavaScript is off

Still outstanding — see *Known gaps* below.

## Known gaps

Things a future pass should address:

- **Analytics consent.** Google Analytics loads unconditionally. UK PECR
  normally requires consent before setting analytics cookies — either add a
  consent banner with GA4 Consent Mode, or switch to a cookieless analytics
  provider.
- **Social preview image.** `og:image`, `og:url` and the canonical link need
  absolute URLs once the final domain is fixed; until then link previews are
  text-only.
- **No custom analytics events.** Only pageviews are tracked, so there is no
  visibility into where people drop out of the four-phase funnel.
- **No progress persistence.** A refresh restarts the simulation.
- **No shareable result.** Workshop participants cannot share or export their
  outcome as a link.
- **Licensing is undecided** — see below.

## Sources and licensing

The simulation draws on published UK government statistics, sector reports and
peer-reviewed research. Each source carries its own `licence_note` in the data,
and the mix includes Crown Copyright / OGL v3.0 material, academic copyright,
and sector publications summarised under fair dealing.

**This repository has no licence file yet.** Because the bundled data
summarises third-party research under a mix of terms, the code licence and the
data/content terms should be decided separately before the repo is made public
or reused. Until a licence is added, default copyright applies.

Barrier data was synthesised with AI assistance from published reports, then
reviewed and structured by hand; individual figures may not match primary
sources exactly. The impact figures in Phase 3 are illustrative, not forecasts.

## Credits

Built by **Steve Law** — fundraising strategist.
[LinkedIn](https://www.linkedin.com/in/stevejohnlaw/) ·
[Buy me a coffee](https://buymeacoffee.com/stevelaw)
