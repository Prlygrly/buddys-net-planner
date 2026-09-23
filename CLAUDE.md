# Buddy's Net Planner

A single-page tool for FarmRPG players. They paste their Mastery Progress page, and it tells them how many Grand Masteries (GM, 100,000 caught) and Mega Masteries (MM, 1,000,000 caught) they can finish with a given number of Large Nets, and where to fish.

Live site: https://prlygrly.github.io/buddys-net-planner/ (GitHub Pages, deployed from `main`, root folder).
Everything lives in `index.html`: HTML, CSS and JS in one self-contained file. No build step, no dependencies beyond Google Fonts.

## Audience and writing standard
The main user is a newer player with a learning disability. **Every piece of on-screen text should be understandable without prior knowledge.** Point at what's on screen ("that column's nets at that row's fishing spot"), use plain verbs and sentence case, and explain terms like GM/MM where they appear. Keep it friendly but concise.

## Owner preferences
- Always keep the dark mode toggle (the header button; theme tokens on `:root`, `@media (prefers-color-scheme: dark)` guarded by `:root:not([data-theme="light"])`, plus `:root[data-theme="dark"]`).
- Never use the phrase "load-bearing".
- Commit and push changes to `main` when asked; Pages redeploys automatically.

## How it works
**Parsing** (`parse()`): the player selects all on farmrpg.com's mastery page and pastes it.
- Pass 1 is line-based: an item name on its own line, followed by a `X / Y Progress` line.
- Pass 2 handles one-line pastes from some phone browsers. The name must follow `Stop`, `Track`, `Complete!` or `chevron_*`, which avoids matching "Catfish" inside "Frozen Catfish".
- Only names in the fish data are kept. Duplicates keep the **highest** count, since mastery only goes up.
- Collapsed tiers copy as `Tier ... chevron_right`. A tier is flagged only if it never appears as `chevron_down`/`chevron_up` anywhere in the paste.
- Mega Mastered items show `N / ∞ Progress` + `Complete!`.

**Net math:** fish per Large Net is 250 base, +150 for Reinforced Netting (default on) and +100 for Fishing Trawl (default off), times (1 + event bonus %). These constants come from buddy.farm's source (`src/utils/format.tsx`).
Nets needed = ceil((target − current) × fishesPerDrop ÷ fishPerNet).

**Data** (`LOCS`): fishes-per-drop for each item at 11 locations, snapshotted from buddy.farm in September 2026. Apple Bobbing (event) and Old Boot (no mastery) are left out. Items rarer than 200 fishes/drop everywhere count as "rare" and are hidden unless found in the paste or "Show rare drops" is ticked.

## Page behaviour
- Only step 1 shows until a paste yields mastery data, or the player clicks "Type your numbers in by hand".
- When every expected fish is found and no tier is collapsed, the paste area folds up (CSS grid-rows animation) into a "Loaded N items" bar with an **Update masteries** button that reopens it with the old text selected. A **Done pasting** button forces the collapse.
- Fishing spots switch off automatically unless `spotUnlocked()` passes: at least half of the spot's own (non-rare, found only at that spot) fish have a count above 0. One stray catch, like a Stone Jelly from elsewhere, doesn't unlock Glacier Lake. The missing-fish warning only covers unlocked spots.
- Step 3: a TL;DR row of best-spot cards per budget (1k/3k/5k/10k plus an optional custom one), shown only for budgets where something finishes. If nothing finishes, it shows the three closest milestones instead. The heat grid drops empty rows and columns and hides entirely when empty. The recommendation cards include a greedy "split your nets" plan when it beats one spot.
- Step 4: a table of every item, sortable, with a location filter. "Done" sorts as the highest value in the nets columns. The Caught column is editable, and edits reset on a new paste.
- Respect `prefers-reduced-motion`.

## Testing
Playwright with Chromium works well: load `index.html` via `file://`, fill `#paste` with sample text, and check `#status`, `#tldr`, `#heat` and `#list`. `window.__netPlanner.parse(text)` is exposed for parser tests. Test both multi-line and single-line pastes, duplicates, and a collapsed tier.

## Ideas not done yet
- A plain-language pass over remaining labels: the "split it" line, the GM/MM tags, the goal toggle, "Show rare drops", and "spot switched off".
- Refreshing drop rates from buddy.farm (it also has a GraphQL API at api.buddy.farm).
