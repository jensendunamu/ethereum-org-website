---
name: new-page
description: Interview-then-scaffold guide for adding a new page to ethereum.org. Establishes whether the page is markdown-content or an App Router page, pulls Figma designs when available, maps the design to existing components and a layout, and builds it reuse-first. Invoke with /new-page.
disable-model-invocation: true
---

# New Page

A new ethereum.org page is **reuse-first**: every region maps to a primitive, variant, or layout that already exists — you *configure* what's there, you don't build a parallel version of it. This skill interviews you for the missing context, then scaffolds the page against the design system.

Work the steps in order. Each ends on a checkable condition — don't start the next until the current one is met.

## Step 0 — Load the design system

Invoke the `design-system` skill and read its `SKILL.md` fully before asking anything else. It is the source of truth for component choices, tokens, layouts, RTL/i18n, and the server/client boundary. Every step below assumes its rules and its cheatsheet. **Do not restate its catalog here** — when you need the layout inventory, read `design-system/references/layouts.md`; for imports, its "Where Do I Import From?" table.

**Done when:** `design-system` SKILL.md has been read this session.

## Step 1 — Interview

Use `AskUserQuestion`. Resolve every branch before writing code. Cover:

1. **Page type — confirm, never infer.** The highest-stakes branch; everything downstream forks on it. A prose-heavy, hero-topped design is *not* evidence either way — the App Router content pages (`/what-is-ethereum/`, `/learn/`) carry the same hero, breadcrumbs, date/read-time/contributors row, and a TOC, exactly like a markdown page. **The TOC is the tell:**
   - *Markdown-content page* — `public/content/<route>/index.md` via `[...slug]`. Its TOC is **hard-locked to the left rail** (`ContentLayout` → `<TableOfContents variant="left">`); the body is plain stacked prose sections.
   - *App Router page* — a hand-composed `app/[locale]/<route>/page.tsx`. Its signature is anything `ContentLayout` can't produce: a **right-hand card TOC** (`<TableOfContents variant="card">`, like `/what-is-ethereum/`), a bespoke multi-column grid, or interactive/data-backed regions.

   With a Figma design you can't classify this reliably until you've read the frame — take a *provisional* answer here and **bind it at the end of Step 2**, not before.
2. **Route** — the URL path (e.g. `/stablecoins/`). The folder name *is* the route.
3. **Design source** — a Figma link/selection, a written description, or none.
4. **Hero shape** — what the top of the page looks like, because it constrains the layout more than the nav does (see Step 3):
   - *Full-width hero with a side image* (title + body + CTAs on one side, illustration on the other — like `/what-is-ethereum/`)
   - *Plain title + breadcrumbs* (no hero band — like a `static` page)
   - *Docs sidebar* / *tutorial metadata* (author, date, skill)
5. **Content** (markdown pages only) — does the user have the body ready, or should you draft it?

Then **check the route is free**: confirm there's no existing `public/content/<route>/index.md` (markdown) or `app/[locale]/<route>/page.tsx` (App Router). If something is already there, you're *editing*, not creating — surface it and confirm intent before any write; never silently overwrite a page you didn't author.

**Done when:** route and design source are known and the route is confirmed free (or editing is okayed); page type is confirmed — or, when a Figma design is provided, provisional pending the Step 2 binding.

## Step 2 — Pull the design (only if Figma was provided)

Read the design **fresh** with the Figma MCP (designs change between runs — never rely on a read from an earlier turn): `get_metadata` for the frame tree, then `get_design_context` / `get_screenshot` on each region; `search_design_system` / `get_code_connect_map` for existing code mappings. Load the `figma-use` skill before any `use_figma` call.

Produce a **component map**: every design region → the existing primitive + variant that renders it (`PageHero`, `Card`, `BigNumber`, `YouTube`, `Alert`, …), citing the design-system cheatsheet. The map must be **exhaustive** — walk the frame top to bottom and account for every region. A region you don't map is a region that silently won't ship.

**Node names lie.** A node called `Screenshot 2026-…` or `Frame 1789` is routinely real content — a stat band (`BigNumber`), a video embed (`YouTube`), a callout (`Alert`) — not a throwaway. Never omit a region because its name *looks* like a draft or screenshot; `get_screenshot` it and see what it actually is before deciding.

**Icons are lucide in JSX, emoji in markdown.** In App Router / `.tsx` code, map every icon region to a `lucide-react` component. The markdown `<Card>` shortcode (`MarkdownCard`) takes only an `emoji` prop — no lucide — so a design whose cards use lucide icons is itself a weak nudge toward an App Router page, or accept emoji on the markdown path.

**Missing data → ask, don't guess.** When a region needs a value the design doesn't carry — a YouTube ID behind a video thumbnail, a real destination URL, a live data source — you can't extract it from the frame. Flag it and ask the user; a guessed embed or invented link is worse than an open question.

**Match, don't mimic.** Reproduce the design using existing components and their variants. Do **not** layer custom Tailwind/CSS on a primitive to chase pixel parity. If a region has no fitting primitive or variant, the design-system answer is *add a variant* (see `references/variant-vs-new.md`) — never a new one-off component or bespoke styles. Flag any such gap and confirm before inventing.

**Bind the page type.** With the whole frame now in view, settle markdown vs App Router against the tells from Step 1 — a **right-hand card TOC**, a bespoke multi-column grid, or interactive/data-backed regions all mean App Router, because no markdown template can produce them. Confirm the binding decision with the user before scaffolding; flipping it later means rewriting the page.

**Done when:** every region is accounted for — each mapped to a component+variant, flagged for an approved variant addition, or raised with the user as missing data — and the page type is bound and confirmed. Nothing left unexamined.

## Step 3 — Choose the layout (markdown pages only)

The layout is selected by the `template:` frontmatter value (default `static` via `getLayoutFromSlug`). Read `design-system/references/layouts.md` for the current, authoritative inventory.

**Decide on two independent axes, not one.** The common mistake is to pick the layout by its navigation chrome alone and let the hero fall out by accident. It won't: **`static` renders no hero at all** — just breadcrumbs + `<h1>` + the action row. The full-width `PageHero` (side image + title + CTAs, driven by frontmatter `image:` / `buttons:` / `summary:`) is rendered **only by `TopicLayout`** in the markdown world. So decide the hero (Step 1) *and* the nav separately:

| Hero shape (Step 1) | Sub-nav across siblings? | `template:` | Notes / exemplar |
|---|---|---|---|
| Full-width side-image hero | Yes | a topic template (`use-cases`, `staking`, …) | `TopicLayout` with its own topic config |
| **Full-width side-image hero** | **No** | **`use-cases` + `showDropdown: false`** | **`PageHero` without the sub-nav. Exemplar: `public/content/what-are-apps/index.md`.** This is the right call for a standalone page that still needs a hero. |
| Plain title + breadcrumbs (no hero) | — | `static` | `StaticLayout`. Exemplar: most one-off pages. |
| Docs sidebar | — | `docs` | `DocsLayout` |
| Tutorial metadata (author/date/skill) | — | `tutorial` | `TutorialLayout` |

**Disqualifier — do not skip:** if the design shows a full-width hero band with a side image, `static` is **wrong**, no matter how prose-heavy the page is or how sibling-less it is. Reach for `use-cases` + `showDropdown: false` (markdown) or an App Router page composing `PageHero` directly (exemplar: `app/[locale]/what-is-ethereum/page.tsx`).

Present the matched row(s) to the user and confirm. If a Figma design was provided, **auto-detect in this order**: (1) is there a full-width hero band with a side image/title/CTA? → a `PageHero`-capable option above, *not* `static`; (2) only then read the nav chrome — sub-nav across siblings → topic template (dropdown on) vs `showDropdown: false`; docs sidebar → `docs`; author/date/skill → `tutorial`; (3) plain title, no hero → `static`.

A new layout is essentially never the answer. A new topic hub is a `src/data/topics/<key>.ts` config plus `layoutMapping` / `componentsMapping` entries plus a translation namespace — not a new layout file. Borrowing an existing topic template via `showDropdown: false` (as `what-are-apps` does) needs no new config at all. (`layouts.md` has the full rule and the worked example.)

**Done when:** the `template:` value (and `showDropdown` if relevant) is chosen and confirmed, and the hero shape is accounted for.

## Step 4 — Scaffold

**Markdown page:**
- Create `public/content/<route>/index.md` with frontmatter (`title`, `description`, `lang: en`, plus `template:` unless static). English only — never hand-write translated copies; the intl-pipeline propagates them.
- **Inline images co-locate**: drop them next to `index.md` in the content folder and reference them relatively (`![alt](./hero.png)`). A `/images/...` path on an inline image 500s the MDX render. (The frontmatter `image:` hero used by `TopicLayout` is the one exception — that one takes a `/images/...` public path.)
- Every h1–h4 needs a `{#kebab-id}`; run `pnpm lint:md:fix`.
- If using `TopicLayout`: add `src/data/topics/<key>.ts`, wire `layoutMapping`/`componentsMapping` in `src/layouts/index.ts`, and add `src/intl/en/page-<key>.json`.

**App Router page:**
- Create `app/[locale]/<route>/page.tsx`. Server Component unless it needs state/effects/handlers. Pick the shape by TOC side (the Step 2 binding):
  - *Left-rail TOC* — compose `ContentLayout` (mirror `/learn/`); it owns the hero slot, the `variant="left"` TOC, contributors, and feedback.
  - *Right-hand card TOC* (the `/what-is-ethereum/` shape) — hand-compose `PageHero` + a `MainArticle` with `grid grid-cols-1 lg:grid-cols-[1fr_auto]`, dropping `<TableOfContents variant="card">` (both an `isMobile` and a desktop instance) in the right column and `<Section id>` prose in the left. Build the `tocItems` array by hand.
- All user-facing strings via `getTranslations`; add keys to `src/intl/en/`.

**Both:** build from the Step 2 component map. Reuse-first — no raw `<a>`/`<button>`, no hex colors, logical CSS props (`ms-`/`me-`/…), locale-aware `numberFormat()`/`dateTimeFormat()`.

**Done when:** the page renders through its layout with content/strings in place and no inlined reinventions of existing primitives.

## Step 5 — Verify (static checks)

- `pnpm type-check` (catches invalid chain names and type errors).
- Walk the design-system **Pre-Merge Smoke Test** checklist.
- Add a `.stories.tsx` for any genuinely new UI primitive.

**Done when:** type-check passes and the smoke checklist is clean.

## Step 6 — QA in the browser

Static checks pass on pages that are broken in the browser — a wrong image path, a failed MDX compile, or a section that silently didn't render. So run the page and look at it.

1. **Run it.** Start `pnpm dev`, wait for "Ready", and load the page. Routes are locale-stripped for the default `en` (`/<route>/`, e.g. `http://localhost:3000/privacy/`). Confirm the page returns **HTTP 200** — a 500 is almost always an MDX or asset error, so read the dev log for the cause (a non-co-located inline image is the classic one).
2. **Compare to the design.** Screenshot the full page with `playwright-cli`. If a Figma design was provided, set the screenshot beside the design frames and hunt for **obvious** gaps — a missing section, a component that rendered as raw text, a broken or oversized image, content that didn't appear. Fix and reload. Repeat until the page faithfully reflects the design, minus the intentional layout deviations you flagged in Step 3.
   - **Check the hero first.** If the design has a full-width side-image hero and the page renders a plain title instead, that's a **wrong-template bug, not a CSS gap** — go back to Step 3 (you likely need `use-cases` + `showDropdown: false`, not `static`). Never try to rebuild a `PageHero` with custom markup on a `static` page.
   - **Check the TOC side.** Design shows a right-hand card TOC but the page rendered a left rail? That's a **page-type miss, not a CSS gap** — markdown is locked to the left rail, so matching the design means an App Router page (Step 1's binding). Never force a card TOC onto a markdown page.
3. **Hand off.** Leave the dev server running and give the user the local URL the server printed (e.g. `http://localhost:3000/<route>/`) so they can see the result.

**Done when:** the page serves 200, shows no obvious omissions against the design, and the user has the live URL.
