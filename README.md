# Financing Resilience Toward 2040 — Multi-stakeholder foresight simulation

## 1. Overview

A facilitated, turn-based exercise based on the foresight paper *Financing Resilience Toward 2040: Alternate Futures for SIDS Disaster-Risk Financing and South-South & Triangular Cooperation* (source PDF: `uploads/Nov. 18th Agenda-Practicum.pdf`). Up to nine stakeholder roles each make one decision per round across three periods (2026–2029, 2030–2034, 2035–2040). Their choices move the paper's two critical uncertainties, international cooperation (axis A) and SIDS agency (axis B), plus seven outcome indicators. The final position on the two axes sets which 2040 scenario arrives. The exercise is meant to show how decisions by different actors, especially joint ones, push the system toward Cooperative Resilience, Fragmented Patchwork or Retrenchment and Exposure. A second tab holds a written gap analysis of the paper.

**All decision options, effect sizes, starting values and role interests are placeholders written for the prototype.** None come from data. See section 5.

## 2. Quick start

The project is one HTML file plus a runtime script. There is no `index.html` and no build step.

```bash
# from the project folder
npx serve .
# or
python3 -m http.server 8000
```

Then open `http://localhost:3000/Resilience%202040%20Simulation.dc.html` (or port 8000 for Python). In VS Code you can also right-click the file → **Open with Live Server**.

Opening the file directly (double-click, `file://`) has not been tested; a local server is the safer option.

**External dependencies (internet connection required; it does not work offline):**

| Dependency | Loaded by | Purpose |
|---|---|---|
| `support.js` (local, generated) | `<script src="./support.js">` in `<head>` | Runtime that reads the template and logic in the HTML file and renders them with React. Header says "GENERATED … do not edit". |
| React 18.3.1 + ReactDOM 18.3.1 (unpkg CDN) | `support.js` | Rendering |
| Babel standalone 7.29.0 (unpkg CDN) | `support.js` | Transpiles code at runtime |
| IBM Plex Sans + IBM Plex Mono (Google Fonts) | `<helmet>` in the HTML file | Typography; falls back to Helvetica / monospace |

To run offline you would need to download those CDN files and fonts and point `support.js` and the `<link>` at local copies. That has not been done.

## 3. Project structure

```
.
├── Resilience 2040 Simulation.dc.html   ← the whole simulation (511 lines)
├── support.js                            ← generated runtime (do not edit)
├── uploads/
│   └── Nov. 18th Agenda-Practicum.pdf    ← source foresight paper
└── README.md
```

### Inside `Resilience 2040 Simulation.dc.html`

The file is a "Design Component": the page layout sits inside `<x-dc>…</x-dc>`, uses `{{ name }}` placeholders for live values, and every style is written inline (there is no stylesheet). The logic sits in one `<script data-dc-script>` block at the bottom.

| Lines (approx.) | Section |
|---|---|
| 1–9 | Document head; loads `support.js` |
| 10–14 | `<helmet>`: Google Fonts link and the only CSS rules (body reset, link colours) |
| 16–25 | Header: title and the two tabs (Simulation / Paper gap analysis) |
| 27–77 | **Gap analysis tab** — hard-coded text in three columns: Missing (10), Needs improvement (8), To add (8) |
| 79–115 | **Setup screen** — research question, "Each round" steps, role toggles, Begin button |
| 117–314 | **Game board** (shown during play and debrief) |
| 120–127 | Round progress bar (3 rounds + Debrief) |
| 129–142 | Wildcard panel |
| 144–224 | Play view: round briefing (146–168), round result panel (170–196), stakeholder decision cards (197–216), Resolve button (217–222) |
| 226–263 | Debrief view: 2040 outcome, no-regret moves, debrief questions, decision log, Run again |
| 266–312 | Sidebar: scenario-space plot, indicator bars, facilitator wildcard buttons |
| 317 | `<script data-dc-script data-props="…">` — the 4 settings (section 4) are declared in the `data-props` attribute |
| 318–324 | Model constants: `R`, `TH`, `KEYS`, `SHORT`, `NAMES`, `INV`, `START`, colours |
| 325–335 | `ROLES` — the nine roles |
| 336 | `o()` helper for writing options |
| 337–395 | `ROUNDS` — briefings, background trends, signposts, linked decisions and all 81 options (round 1: 337–356, round 2: 357–375, round 3: 376–395) |
| 396–400 | `WILD` — three wildcards |
| 401–405 | `SCEN` — the three end states from the paper |
| 408 | `NOREG` — the five no-regret move categories |
| 409–412 | Helpers: `roleName`, `fxText`, `classify`, `pct` |
| 414–509 | `class Component` — state, the round engine, and `renderVals()`, which prepares every value the layout shows |

### Recommended split

The current format only works with `support.js`. Two options:

**A. Keep the format, move the content out (low effort, recommended first).** Put the content tables in a separate file so researchers can edit scenarios without touching the logic:

```
data/roles.js        ROLES
data/rounds.js       ROUNDS (briefings, options, linked decisions)
data/wildcards.js    WILD
data/scenarios.js    SCEN, NOREG
data/model.js        R, TH, START, KEYS, NAMES, SHORT, INV
```

Load them from the logic class with dynamic `import()` (or switch to JSON plus `fetch`). This has not been tried in this project, so test it first.

**B. Port to a standard web app.** `index.html` (markup), `styles.css` (move the repeated inline colours and fonts into CSS variables), `data.js` (as above), `sim.js` (pure functions: `apply`, `resolveRound`, `classify` — no UI code), `ui.js` (rendering). This makes the model testable on its own but means rewriting the template.

## 4. Model & logic

### Core entities

| Name | What it is |
|---|---|
| `ROLES` | 9 roles: `car` Caribbean SIDS, `pac` Pacific SIDS, `ais` AIS governments, `mdb` development banks & donors, `ins` insurers & reinsurers, `reg` regional institutions, `lm` last-mile actors, `unossc` UNOSSC, `un` UN system (other offices). Each has `name`, `desc`, `interest`. |
| `ROUNDS[i]` | One per period: `years`, `title`, `brief`, `base` (background trend applied every round), `baseNote`, `signposts` (3 labels), `coalitions` (3 linked decisions), `options` (3 per role). |
| Option `{t, d, fx, tag}` | Title, description, effects object (e.g. `{a:1, pre:6, debt:-3}`), optional no-regret tag (`cap`, `data`, `pool`, `priv`, `match`). |
| Coalition `{n, req, fx}` | Linked decision. `req` maps role id → list of option **indices** (0-based) that count. If every required role picked one of those options, `fx` is applied. |
| `WILD` | 3 wildcards: `habit` island habitability, `unfin` UN financing withdrawn, `partners` partnerships reconfigure. Each has `fx`, `text`, `prompts`. |
| `SCEN` | 3 end states: `s1` Cooperative Resilience, `s2` Fragmented Patchwork, `s3` Retrenchment and Exposure. Each has name, colour, description and debrief questions. |

### State (what changes during a session)

| Field | Meaning |
|---|---|
| `tab` | `'sim'` or `'gap'` |
| `screen` | `'setup'`, `'play'` or `'debrief'` |
| `active` | Which roles are in play (`{car:true, …}`) |
| `round` | 0, 1 or 2 |
| `choices` | `[{roleId: optionIndex}, …]`, one object per round |
| `resolved` | Whether the current round has been resolved |
| `vals` | Current values of the 2 axes and 7 indicators |
| `trail` | Earlier axis positions (grey dots on the plot) |
| `result` | Snapshot for the result panel: `before`, `after`, coalition outcomes |
| `wild` / `wildUsed` | Wildcard currently shown / which have fired |

### Tracked variables

| Key | Name | Start | Range | Better when |
|---|---|---|---|---|
| `a` | Cooperation (A) | −2 | −25 to 25 | higher |
| `b` | SIDS agency (B) | −2 | −25 to 25 | higher |
| `gap` | Protection gap | 60 | 0–100 | lower |
| `reach` | Last-mile reach | 30 | 0–100 | higher |
| `debt` | Debt stress | 58 | 0–100 | lower |
| `data` | SIDS data ownership | 25 | 0–100 | higher |
| `pre` | Pre-arranged finance share | 20 | 0–100 | higher |
| `pool` | Regional pooling | 35 | 0–100 | higher |
| `trust` | Coalition trust | 50 | 0–100 | higher |

The indicators are unitless 0–100 scores, not percentages or dollars, and the starting values are invented.

### Parameters

The four settings are declared in `data-props` (line 317). They can be changed in the Tweaks panel of the tool this was built in. **When run on its own (e.g. from VS Code) there is no panel, so the defaults apply**; to change one, edit the fallback in the code (e.g. `this.props.sensitivity ?? 1`).

| Name | Default | Range | Effect |
|---|---|---|---|
| `showEffects` | `false` | true/false | Shows each option's numeric effects on its button |
| `showCoalitionHints` | `false` | true/false | Lists this round's linked decisions and the roles they need in the briefing |
| `sensitivity` | `1` | 0.5–2, step 0.25 | Multiplies every effect before rounding |
| `autoWildcard` | `'Off'` | Off / Round 2 / Round 3 | Fires the island-habitability wildcard automatically when that round starts |
| `R` (constant) | `25` | — | Axis limit (±25) |
| `TH` (constant) | `7` | — | Distance from centre on both axes needed to reach a corner scenario |

### Step function — order of operations (`resolve()`)

1. Stop if any active role has not chosen.
2. Apply the round's background trend (`base`).
3. Apply each active role's chosen option, in `ROLES` order.
4. Check each linked decision in order; apply its effects if all required roles chose a qualifying option.
5. Save the result snapshot, add the previous position to the trail, and lock the round.

`next()` moves to the next round (and fires the auto-wildcard if set) or, after round 3, to the debrief.

### Formulas and assumptions, in plain language

- **Additive model.** Every effect is added to the current value; nothing interacts. For each effect: `new = old + round(effect × sensitivity)`, then clipped to its range (±25 for axes, 0–100 for indicators).
- **Background trends** are fixed and the same in every session: round 1 cooperation −2, gap +2; round 2 cooperation −1, agency −1, gap +3, debt +3; round 3 cooperation −1, gap +3. With no action, the system drifts toward Retrenchment.
- **Scenario classification** (`classify`): A ≥ 7 and B ≥ 7 → Cooperative Resilience; A ≤ −7 and B ≤ −7 → Retrenchment and Exposure; **anything else** → Fragmented Patchwork. So Fragmented Patchwork covers the two off-diagonal corners (high A with low B, and low A with high B) as well as the cross-shaped middle. The paper's “mixed” middle is treated as one scenario.
- **Indicators do not affect the scenario.** Only A and B decide the outcome. Debt, gap and so on are reported but have no feedback effect.
- **No-regret count**: each chosen option with a tag adds 1 to that category. Which options got which tag was my judgement, not validated.
- **Wildcards** apply their effects once, immediately, whenever the facilitator clicks.

### Randomness

None. The model is fully deterministic: the same choices always give the same result. No random numbers or seeds are used.

## 5. Data

There are **no datasets** in the code. All numbers (starting values, effects, thresholds) are illustrative placeholders, and there are no comments citing sources.

The text references real frameworks and institutions, which should be checked against the paper and primary sources before use:

- Compromiso de Sevilla, paras 17 and 32(e)–(f) (round 1 briefing)
- COP31 outcomes (round 1)
- Antigua and Barbuda Agenda for SIDS, period ending 2034 (round 2)
- Loss and Damage Fund (round 2)
- CCRIF, PCRIC, Pacific Resilience Facility, SIDS Centre of Excellence (roles and options)
- Multidimensional Vulnerability Index (MVI) — the model assumes it shapes concessional terms
- "Four ESB recommendations" (debrief question) — taken from the paper, not defined in the code

Scenario names, the two axes, the three wildcards and the no-regret categories follow the paper. The two off-diagonal corners are folded into Fragmented Patchwork in the prototype; whether they deserve their own scenario is an open decision for the team.

## 6. UI & interaction

There is no URL routing; screens switch based on state (`tab`, `screen`). Refreshing the page resets everything; nothing is saved.

| View | Controls | Bound to |
|---|---|---|
| Header tabs | Simulation / Paper gap analysis | `tab` |
| Setup | Role cards (click toggles in/out); **Begin round 1** | `active`; `screen → 'play'` |
| Play: briefing | Read-only: brief, background trend, signposts, optional linked-decision hints | `ROUNDS[round]` |
| Play: role cards | Click one option per role (locked after resolve) | `choices[round]` |
| Play: Resolve | Enabled-looking once all roles decide; does nothing until then | `resolve()` |
| Play: result panel | Change in all 9 variables this round; linked decisions formed (●) or not (○); Next button | `result` |
| Sidebar plot | 2×2 scenario space; black dot = current position, grey dots = history; shaded corners = Cooperative Resilience (top right) and Retrenchment (bottom left); everything else is Fragmented Patchwork (64% = threshold 7) | `vals.a`, `vals.b`, `trail` |
| Sidebar indicators | Bar width = value (0–100); coloured change since start (teal good, coral bad) | `vals`, `START` |
| Sidebar wildcards | Inject buttons (marked USED after firing; clicking again only reopens the panel); Reset simulation | `fire()`, `restart()` |
| Wildcard panel | Description, effects, discussion prompts; Back to the round | `wild` |
| Debrief | Outcome scenario, no-regret counts, debrief questions (2 scenario-specific + 3 general), decision log; Run again | `classify(vals)`, `choices` |

Charts are plain positioned boxes, not a charting library.

## 7. Key functions reference

All in the `<script data-dc-script>` block.

| Function | Purpose | Inputs → outputs | Line |
|---|---|---|---|
| `o(t,d,fx,tag)` | Shorthand to write an option | strings, effects, tag → option object | 336 |
| `roleName(id)` | Look up a role's display name | role id → name | 409 |
| `fxText(fx)` | Format effects for display | `{a:1,debt:-3}` → `"Cooperation +1 · Debt −3"` | 410 |
| `classify(a,b)` | Decide the scenario | axis values → `'s1' \| 's2' \| 's3'` | 411 |
| `pct(v)` | Axis value to plot position | −25…25 → 0…100 (%) | 412 |
| `fresh()` | Initial state | — → state object | 415 |
| `apply(v,fx)` | Add effects with sensitivity and clipping | values, effects → new values | 417 |
| `autoWild(round,st)` | Auto-fire habitability wildcard | round index, state → partial state (or `{}`) | 422 |
| `resolve()` | Resolve the current round (section 4) | state → updates `vals`, `result`, `trail`, `resolved` | 428 |
| `next()` | Next round or debrief | state → updates `round` / `screen` | 442 |
| `fire(w)` | Inject a wildcard | wildcard → updates `vals`, `wild`, `wildUsed`, `trail` | 448 |
| `renderVals()` | Compute every value and click handler the layout shows | state + settings → object of display values | 453 |
| `start`, `toggle`, `pick`, `restart`, `closeWild`, `toSim`, `toGap` | Click handlers defined inside `renderVals()` | click → `setState` | 453–508 |

## 8. Design system

There are **no CSS variables**. All colours and fonts are typed directly into inline styles and repeated.

| Colour | Use |
|---|---|
| `#f5f4ef` | Page background |
| `#fbfaf7` | Cards / panels |
| `#1c1d1a` | Text, primary buttons, selected states, dark result panel |
| `#5b5d57` | Secondary text |
| `#dcdad2`, `#e7e5de`, `#c9c7be` | Borders and dividers |
| `oklch(0.52 0.09 195)` teal (`TEAL`) | Cooperative Resilience; "improved" changes |
| `oklch(0.55 0.09 75)` amber (`AMBER`) | Fragmented Patchwork |
| `oklch(0.52 0.09 30)` coral (`CORAL`) | Retrenchment; "worsened" changes; wildcard panel border |
| `#8d8f88` | Trail dots; bars for "lower is better" indicators (gap, debt) |

Fonts: **IBM Plex Sans** (body, 400/500/600) and **IBM Plex Mono** (labels, numbers, 400/500).

## 9. Known limitations & bugs

**Content and validation**
- All options, effects, starting values and role interests are invented placeholders. The model has not been calibrated or playtested.
- Indicators have no units, data sources or baselines.
- No-regret tags are a judgement call and look questionable in places (e.g. MDB contingent credit lines tagged as private-sector involvement; Caribbean "expand CCRIF cover" tagged as capacity building).
- Indicators never feed back into the axes or the outcome (e.g. high debt does not limit later choices).
- "Fragmented Patchwork" is the catch-all for everything not in a corner, which is a modelling choice, not something the paper defines.

**Mechanics**
- `Math.round` is not symmetric for negative halves: with sensitivity 1.25, +2 becomes +3 but −2 becomes −2 (JavaScript rounds −2.5 up to −2). Sensitivity values other than 1 and 2 therefore slightly favour positive effects.
- Clipping happens after each effect, so the order effects are applied in matters when a value hits 0, 100 or ±25.
- Linked decisions check option **positions** (0, 1, 2). Reordering options in `ROUNDS` silently breaks them.
- A wildcard fired before Resolve is not included in that round's result panel (the "before" snapshot is taken at resolve time). A wildcard can also be fired after Resolve, which changes values the result panel does not show.
- Only the habitability wildcard can auto-fire. If it was already fired by hand, auto-fire is skipped silently.
- The Resolve button looks greyed until everyone decides but is still clickable; it just does nothing.
- Roles can only be changed on the setup screen. If a role required for a linked decision is out of play, that decision can never form ("not possible, role not in play").
- The setup label "of 9 selected" is hard-coded.

**Hard-coded to three rounds** (adding a round needs all of these changed): `choices:[{},{},{}]` in `fresh()`; `Math.min(st.round+1,2)` in `nextLabel`; `repeat(3,…)` in the decision log grid (line ~249).

**Other**
- No saving: a page refresh mid-session loses everything.
- Requires internet (CDN scripts and fonts).
- The gap analysis tab is static text; it will not update if the model changes.
- One shared screen only; there is no secret voting or per-player device.
- Accessibility not reviewed: selected states rely on colour and border; buttons have no `aria-pressed`.
- No automated tests.

## 10. Suggested next steps

1. **Replace placeholder content with the team's own.** Write role briefs and decision options with the paper's authors; get baselines and data sources for each indicator.
2. **Move content out of the code** (section 3, option A) so non-developers can edit roles, options and scenarios in one place. Refer to linked-decision options by an id instead of position.
3. **Publish an effects annex.** Generate a table of every option's effects from `ROUNDS` so reviewers can check the model.
4. **Fix the mechanics bugs** in section 9 (rounding, wildcard timing in the result panel, Resolve button state, 3-round hard-coding).
5. **Add tests** for `apply`, `classify` and round resolution, e.g. "all positive options → Cooperative Resilience", "all negative → Retrenchment".
6. **Validate the model.** Playtest with two or three interviewees; check that outcomes feel plausible and that no single dominant strategy exists.
7. **Add feedback loops** where the paper implies them (e.g. debt stress reducing SIDS agency; trust affecting whether linked decisions form).
8. **Save sessions** (localStorage) and export the decision log (CSV/PDF) for the debrief record.
9. **Facilitation features:** negotiation timer, printable role cards, pre/post participant survey.
10. **Make it work offline** by bundling scripts and fonts locally.

## 11. Conventions for contributors

- **Style:** plain JavaScript (no TypeScript, no imports in the logic class). The class must stay named `Component`. Keep logic in the script block; the layout should only read named values from `renderVals()` — the `{{ }}` placeholders cannot contain expressions.
- **Styling:** inline `style="…"` only, reusing the colours in section 8. Do not edit `support.js`.
- **Naming:** short lowercase ids for roles (`car`, `pac`…) and variables (`a`, `b`, `gap`…). User-facing labels go in `NAMES` / `SHORT`.
- **Add a new indicator:** add its key to `KEYS`, `SHORT`, `NAMES`, `START`; add it to `INV` if lower is better; use it in option `fx` objects.
- **Add a role:** add an entry to `ROLES` and three options under its id in **every** round's `options` (otherwise the round crashes); update the "of 9" label.
- **Edit options:** change the `o(title, description, effects, tag)` entries in `ROUNDS`. If you reorder options, update any `coalitions[].req` indices that point at them.
- **Add a linked decision:** add `{n, req:{roleId:[optionIndex,…]}, fx}` to a round's `coalitions`.
- **Add a wildcard:** add `{id, n, fx, text, prompts}` to `WILD`.
- **Change scenarios:** edit `SCEN` text and questions; change boundaries via `TH` and `classify`, and update the plot shading (the `64%` values around line 276) to match. Only three scenarios are defined; `classify` returns `s1`, `s3` or `s2`.
- **Add a round:** add an entry to `ROUNDS` and fix the three hard-coded places listed in section 9.
- Keep the gap analysis tab in sync manually if the model changes what it describes (e.g. "81 options").

## 12. Drivers and pathways (added)

- **Source of the content:** the driver states, pathways, effects and stakeholder reactions are drafted in the Claude Doc "Driver pathways 2026-2040 (draft)": https://claude.ai/artifact/EsWPh7PQKVitQi9pGPSLUy. Edit the doc first, then mirror changes here.
- **Constants:** `DRIVERS` (five drivers from section 3 of the paper, with a description per state), `PATHWAYS` (Positive, Medium and Negative pathway; one state per driver per round). Hazard stays Medium until 2035–2040, when the Negative pathway sets it to Negative; the Positive pathway has no hazard change, and `DFX` (effects per driver state, added to the round trend). Medium adds nothing.
- **State:** `pathway` (set on the setup screen, default `Medium`).
- **Resolve:** driver effects are applied after the background trend and before the role options.
- **Briefing:** the play view shows each driver's state for the current round.
- **Not yet built:** stakeholder reactions in the options (the doc lists them; the simulator does not use them), and driver states in the debrief.
- **Numbers are placeholders.** The effect sizes in `DFX` are not calibrated.
