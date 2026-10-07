# Handoff — ETHIndia Institutions site

Start a new chat with: "Read HANDOFF.md and PRODUCT.md, then continue from 'Next'."

## What this is

A **reskin of the old portal at https://institutions.ethindia.co**. The client wants the portal's data exactly, in this repo's styling.

- **Data is the portal's, verbatim:** copy, nav, sections, page structure, titles, meta descriptions, module headings and questions, ledger, reconciliation, the copy-for-an-LLM texts. Never reword, add claims, or add sections the portal doesn't have. Last two-way text diff of every portal page against `dist` (8 Oct): nothing missing; the only additions are UI controls.
- **Styling is ours:** navy + stone palette, frame hairlines, dotted-India hero map, fonts, motion, buttons.
- Allowed re-presentation of portal text (user-approved): small uppercase kickers became running heads or plain facts (see "Labels" below); the hero eyebrow is visually hidden.
- `ETHINDIA_INSTITUTIONAL_BUILD.md` is the original build brief. It is **superseded** wherever it adds things the portal lacks (ticker, FAQ, team, supporters, newsletter, `/dinner`, `/privacy`, palettes); all of those were removed. Still useful for accessibility rules and tone.
- `PRODUCT.md` (audience, personality, anti-references) still holds. Its `## Register` section is obsolete for the current `/impeccable` skill.

## Stack, commands, deploy

Astro 7 (static), plain CSS custom properties, vanilla JS. One dependency (`astro`). Windows, Node 24.

```bash
npm run dev                     # http://localhost:4321 (.claude/launch.json "eii")
npm run build                   # → dist/
npx astro preview --port 4331   # .claude/launch.json "eii-review"; build first
npx vercel deploy --prod --yes  # manual deploy; look for the "Aliased" line
```

- Git: `main`, remote `origin` = https://github.com/superark21/ethindia-institutional (**public** since 7 Oct; history scanned, no secrets).
- Vercel project `superark21s-projects/ethindia-institutional`, linked locally (`.vercel/`, gitignored). **No Git integration**: pushes don't deploy; deploy from the CLI.
- Live: **https://ethindia-institutional.vercel.app**. Last deploy 8 Oct 2026 (commit `c944155`).
- **`institutions.ethindia.co` still serves the old portal.** `vercel.json` 301s every old `.html` URL (including `ledger.html#fig-N`) to the new paths, so the domain can be attached to the Vercel project whenever the user says so (needs a DNS change on ethindia.co).

## Page structure (mirrors the portal's three page families)

`<Base chrome=…>` picks the frame:

| chrome | Pages | Frame |
|---|---|---|
| `landing` | `/` | Sticky nav: kit logo + Why India, Ethereum, Briefing, Updates (anchors). Footer "Tokenised settlement in India" + "An ETHIndia initiative…" |
| `narrative` | `/briefing` | No nav; hero with "← ETHIndia Institutions". Footer "The full briefing" |
| `portal` | `/briefing/a`–`g`, `/briefing/ledger`, `/briefing/reconciliation` | `PortalShell.astro`: header "Ethereum/India Institutional Briefing / <tab>" + sidebar (overview, modules A–G, reference). Checkbox-driven menu at ≤960px |

URLs: `/briefing/<letter>` (not the portal slugs; those redirect). `/dinner` and `/privacy` no longer exist (404).

## Where the content lives

- `src/content/home.json`: every home string, **generated verbatim from the portal's HTML** (hero, the four Why India points with their figure-chip HTML, Where Ethereum fits, On this site, Briefing, dinner, updates, footer).
- `src/content/site.json`: name ("ETHIndia Institutions"), domain, default title and description.
- `src/content/briefing/`:
  - `<letter>.md`: title, heading (portal H1, e.g. "Module A — What Actually Shipped"), question, description (portal meta), legacy slug, summary bullets.
  - `<letter>.html`: the three reading tiers.
  - `_overview.html`: the entire overview page body, verbatim from the portal (hero, scenes 01–05, findings, LLM prompts, split-sort quiz).
  - `_reconciliation.html`: the reconciliation body including its H1.
  - Portal `.html` links were rewritten to the new URLs. **Edit these HTML files directly to change text.**
- `public/data/figures.json`: ledger data. `public/llm/module-<letter>.md`: each module's "Full report — copy for an LLM" text (verbatim). `public/llms.txt`: the portal's file, verbatim (relative `.html` links resolve via redirects).
- If the portal changes, regenerate from its HTML rather than retyping. The 8 Oct generator was a one-off script in the session scratchpad: fetch each page, take `<main>`, rewrite links, write the files above.

## Behaviour carried over from the portal

- **Ledger** (`ledger.astro`): full static table (works without JS); JS adds module and tier toggles, weak-tier/stale/conflict filters, search, sortable ID/Module/As of/Tier, and per-row "Copy" citation. Staleness = older than six months before `meta.generated`, unless HISTORICAL or CURRENT. Rows keep `id="fig-N"`.
- **Module pages:** H1 + question, reading-depth tabs (30 seconds default, remembers last choice, `#tier-30s|5min|full`), "Full report — copy for an LLM" (`CopyForLlm.astro`, loads `public/llm/…` on open).
- **Overview:** personalised LLM prompt (twice, as on the portal), split-sort quiz ("Show me the answers"), both in the page script of `briefing/index.astro`.
- Not ported: the portal's per-section "Copy" buttons on module h3s (subsets of the full report).

## Visual system

- **Palette:** navy + stone only. Zones: page (stone), `.deep` (nav, heroes, footer, portal header), `.band` (dinner). Components use only `--paper --panel --surface --ink --ink-2 --border --rule --accent --hairline`. All pairs pass AA.
- **Background:** a frame (`body::before`): two hairlines just outside the content edges (`--hair-pad` out), nothing inside the reading column. Chosen 8 Oct over 4-column rules (inner lines crossed headings and rows), margin graph paper, full graph paper, dots and none. At 375px the lines sit 4px from the screen edge, text at 20px; checked on home, overview, a module, ledger and reconciliation with no text crossing and no sideways scroll. `body` has no background on purpose; `html` carries the colour.
  - Leftover: desktop two-column rows (`.point p`, `.split > :last-child`, `.cols > li`, `.dinner__card`) still carry a `--hair-pad` indent that only existed to dodge the old 50% line. Harmless; remove if the user wants flush columns.
- **Fonts:** Hanken Grotesk (variable) stands in for Neue Montreal, and Pixelify Sans for Matrix Sans. Both are self-hosted in `public/fonts` (OFL) and declared in `tokens.css`. The licensed faces stay first in the stacks: drop their files in and uncomment the block in `tokens.css`. The pixel face is only for the hero's glyph-swapped letters, module letters A–G, chapter numerals and the dinner date. Data (tier chips, flags, ledger IDs) uses the sans.
- **Logo:** `public/logo/ethindia-institutional.svg` (from the user's kit), drawn as a CSS mask (`.logo`, `--logo-h`) so it takes the zone's ink. Nav only. `favicon.svg` is the logomark. OG image `public/og/default.png` was rendered once from a throwaway HTML page (navy, bone logo, hairlines).
- **Labels:** no uppercase tracked pixel kickers anywhere (user found them generic).
  - Home section names sit as running heads (`.runhead`) at each section's top-right edge.
  - Overview chapter markers (`.scene-eyebrow`) get the same treatment.
  - "During Devcon 8, Mumbai" and "By invitation" are facts in the dinner card. The dinner date breaks after the weekday on narrow screens, never inside "4 November 2026" (spans in `index.astro`).
  - Update types (Event, Report) sit under the date.
  - "What this could not establish" is a plain bold heading.
- **Buttons:** square-cornered (2px), 48px tall. Hover lightens the fill and draws a brass rule along the bottom.
  - `.btn--secondary` is a rule-coloured outline.
  - The disabled state is a dashed outline ("Invite requests open soon").
  - Arrows are drawn: any `aria-hidden` span inside `.btn`, `.rlink`, `.deeper a` or `.scene-deeper a` is replaced by the `--arrow` mask and nudges on hover.
  - Ruled links (`.rlink`, `.deeper a`, `.scene-deeper a`, `.ref-list a`) sit on a grey rule that redraws in ink on hover, with a 44px hit area.
- **Hero map:** canvas, dotted India (official boundary incl. all of J&K, DataMeet CC BY 2.5 IN), 3D ETH mark, Mumbai ripple. Pauses offscreen; static under reduced motion. Regenerate dots with `node scripts/india-dots.mjs india.geojson src/components/hero/india-dots.json 0.36` after downloading `india-composite.geojson` from datameet/maps (10 MB, not in repo). Don't swap in Natural Earth (different boundary).

## Motion

One grammar, "the ledger being ruled". CSS lives in `base.css` and `Hero.astro`; the script is in `Base.astro`.

- **Load:** the frame lines draw down; the hero title rises out of its baseline; the sentence settles; the map surfaces.
- **Leaving the hero:** copy and map drift apart, on a scroll-driven `view()` timeline.
- **Each below-the-fold section** (`main > section`, `main > .narr > section`):
  1. The seam line draws across.
  2. The running head slides in where the line ends.
  3. The h2 rises word by word (`.w`, `--wi`).
  4. Content settles in reading order with a slight blur (`data-step`, `--step`).
  5. The rule under each row draws (`data-row` on `.points .updates .cols .findings` items).
- **Dinner band:** opens from the content edge to full bleed on a `view()` timeline.
- Sections already on screen at load never animate. Everything is visible without JS, static under reduced motion, and visible in print.

## Verifying visually

- The in-app browser pane works with `resize_window` (mobile preset or 1280×800); screenshots are fine. Sections below the fold may be caught mid-reveal: set `data-reveal="in"` on them before capturing. For multi-page overflow checks at 375px, load each page in a 375px iframe and measure text rects against `body::before`'s left/right.
- **`node scripts/capture.mjs <url> <out> <selector> [delays] [w] [h]`** drives headless Chrome over CDP: it scrolls like a reader and captures frames mid- and post-transition. Set `HOVER=<selector>` for hover states. Run it against the preview on 4331.
- Plain `chrome --headless --screenshot` doesn't scroll to `#hash` reliably and won't lay out narrower than ~500px.
- Design detector: `C:/Users/orok/.claude/skills/impeccable/scripts/impeccable detect --json src/pages src/components src/styles`. It was clean on 8 Oct.
- Scoped Astro styles don't reach `set:html` content or JS-created elements (sort buttons, ledger header buttons). Use `:global()` or `prose.css`.
- Large files: use the Write tool, not Bash heredocs (quoting breaks).

## Client handover (in progress, 8 Oct)

The client (ETHIndia) approved the site and asked for three things:
1. **Their final wording pass on the briefing** for sponsor sensitivities. Must land **before** the domain switch.
2. **The new site on the live URL** `institutions.ethindia.co`.
3. **Self-serve updates:** the briefing facts change a few times a quarter; they need to deploy those without us.

Agreed plan (least work for the user): the client gets the repo (currently just the public link; transfer via GitHub Settings → Transfer ownership when they send a username), imports it into their own Vercel at vercel.com/new (Astro auto-detected, `vercel.json` redirects come along), adds the domain under Settings → Domains after their wording pass, and edits `src/content/briefing/*` in GitHub's web editor; every commit to `main` deploys. Offered but not done: an `EDITING.md` mapping each briefing section to its file.

## Open decisions for the user

- Domain and Git-connected deploys move to the client's Vercel under the handover plan above; until then deploys stay manual from this machine.
- Dinner card copy repeats "Mumbai" ("During Devcon 8, Mumbai" then "Mumbai. The venue is shared with confirmed guests."). Suggested "Venue shared with confirmed guests."; awaiting the user (copy is portal-verbatim, so it's their call).
- The ledger toggles (A–G, T1–T5) and module reading-depth tabs are still rounded pills, which no longer match the square buttons. User was offered the change; no answer yet.
- The hero's pixel-letter swap reads crowded in Pixelify Sans; could go plain sans until Matrix Sans arrives.

## Next

1. The client handover and open decisions above.
2. Licensed Neue Montreal + Matrix Sans files when the client sends them (see Fonts).
3. Re-diff against the portal before attaching the domain, in case the portal changed.
4. Lighthouse on the preview (last full run was 3 Oct, before the reskin):
   ```bash
   CHROME_PATH="C:\Program Files\Google\Chrome\Application\chrome.exe" npx -y lighthouse http://localhost:4331/ --preset=desktop --only-categories=accessibility,performance,best-practices,seo --chrome-flags="--headless=new"
   ```

## File map

```
src/
  layouts/Base.astro            head/meta/OG, font preload, chrome switch, section-transition script
  components/
    Nav.astro                   landing nav: kit logo + 4 anchors, active-section highlight
    Footer.astro                landing / narrative footer variants
    PortalShell.astro           portal header + sidebar (modules, ledger, reconciliation)
    CopyForLlm.astro            "copy for an LLM" disclosure (inline text or fetched file)
    hero/Hero.astro             home hero copy (glyph swap), entrance + scroll-out motion
    hero/IndiaMap.astro         canvas map; hero/india-dots.json is its generated dot grid
  pages/
    index.astro                 home sections + styles
    briefing/index.astro        overview (renders _overview.html) + LLM prompt and sort-quiz script
    briefing/[module].astro     module reader
    briefing/ledger.astro       Figure Ledger + filter/sort/copy script
    briefing/reconciliation.astro
    404.astro, sitemap.xml.ts
  scripts/clipboard.ts          copyText + flash, shared by the copy widgets
  content/                      see "Where the content lives"
  content.config.ts             briefing collection schema
  styles/tokens.css             layout tokens, palette, zones, @font-face, --arrow
  styles/base.css               reset, hairlines, type, runhead, buttons, links, motion
  styles/prose.css              long-form, tables, figure chips, overview (.narr) scenes, findings, sort quiz
scripts/india-dots.mjs, scripts/capture.mjs
public/                         fonts/, logo/, og/, llm/, data/figures.json, llms.txt, robots.txt, favicon.svg
vercel.json                     301s from the portal's .html URLs
```
