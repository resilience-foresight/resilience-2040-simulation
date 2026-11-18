# Financing Resilience Toward 2040: foresight simulation

A facilitated, turn-based exercise for the foresight paper *Financing Resilience Toward 2040: Alternate Futures for SIDS Disaster-Risk Financing and South-South & Triangular Cooperation*. A facilitator runs it on one shared screen with up to nine stakeholder roles. The exercise has three periods (2026–2029, 2030–2034, 2035–2040), a briefing per round, and a 2040 debrief.

**Status: prototype.** Options, effect sizes and starting values are placeholders. Nothing in the model has been calibrated or validated with the team.

## 1. What the simulator does

- **Setup.** The facilitator picks the roles in play, then presses Next to the drivers screen and sets one state per driver of change (Likely, Neutral or Unlikely). Begin round 1 starts the game.
- **Each round.** The briefing shows each driver's state, the background trend, the effect each driver adds this round, the reactions in force, the collective signal from the previous round, and the milestones of the period. Each role card shows its interest, its resources and red line, its decision point with the trade-off, the driver that shapes its options, a decision question, and three options. On Site B, each card also shows the role's track record and its behaviour under pressure.
- **Resolve.** The round's effects are applied: background trend, then drivers, then each role's option, then linked decisions. The result panel splits the change into background trend, drivers and decisions.
- **2040 debrief.** The final position on the two axes sets the scenario. The debrief shows the scenario reached, the no-regret moves, the decision log, the timeline of the run, the ESB recommendations tested against the scenario, a statement for each role, the limits of the run, and a narrative that can be opened with a button.

The two axes are international cooperation (A) and SIDS agency (B). Seven indicators are tracked: protection gap, last-mile reach, debt stress, SIDS data ownership, pre-arranged finance share, regional pooling and coalition trust.

## 2. Two sites from one code base

The two sites share all code. They differ only in one constant, `const SITE`.

| File | `SITE` | What changes |
|---|---|---|
| `Resilience 2040 Simulation.dc.html` | `'A'` | Fixed options for every round and driver state. Linked decisions match by option position. No continuations, follow-up questions, reactions, track records or effects annex. |
| `Resilience 2040 Simulation - Site B.dc.html` | `'B'` | Options depend on each driver's state. Continuations, follow-up questions, direct links, reactions, track records, the effects annex. |

Whenever the shared code changes, regenerate Site B from Site A by replacing `const SITE = 'A';` with `const SITE = 'B';`. Do this as the last step of every change, so the two files stay identical except for that line.

## 3. Quick start

Serve the folder with a local server. Opening the HTML file directly is not supported.

```bash
# from the project folder
python3 -m http.server 8000
```

Then open:

- Site A: `http://localhost:8000/Resilience%202040%20Simulation.dc.html`
- Site B: `http://localhost:8000/Resilience%202040%20Simulation%20-%20Site%20B.dc.html`

**External dependencies (internet connection required):**

| Dependency | Loaded by | Purpose |
|---|---|---|
| `support.js` (local, generated) | `<script src="./support.js">` | Runtime that renders the template and logic in the HTML file. Do not edit. |
| React 18.3.1 and ReactDOM 18.3.1 (unpkg) | `support.js` | Rendering |
| Babel standalone 7.29.0 (unpkg) | `support.js` | Transpiles code at runtime |
| IBM Plex Sans and IBM Plex Mono | `fonts/` (local, `woff2`) | Typography |

## 4. Files

```
.
├── Resilience 2040 Simulation.dc.html          ← Site A (fixed decisions)
├── Resilience 2040 Simulation - Site B.dc.html ← Site B (decisions by driver state)
├── support.js                                  ← generated runtime (do not edit)
├── fonts/                                      ← 18 IBM Plex woff2 files
└── README.md
```

The source paper (`Nov. 18th Agenda-Practicum.docx`) and the evidence brief (`SSTrC_Evidence & Solutions Brief_revised.docx`) are not in this folder.

## 5. Parameters

The four settings are declared in the `data-props` attribute of the script block. In the original tool they could be changed in a Tweaks panel. When the file runs on its own there is no panel, so the defaults apply. To change one, edit the fallback in the code (for example `this.props.sensitivity ?? 1`).

| Name | Default | Range | Effect |
|---|---|---|---|
| `showEffects` | `false` | true/false | Shows each option's numeric effects on its button |
| `showCoalitionHints` | `false` | true/false | Lists this round's linked decisions and the roles they need in the briefing |
| `sensitivity` | `1` | 0.5–2, step 0.25 | Multiplies every effect before rounding |
| `autoWildcard` | `'Off'` | Off / Round 2 / Round 3 | Fires the island-habitability wildcard when that round starts |
| `R` (constant) | `25` | — | Axis limit (±25) |
| `TH` (constant) | `7` | — | Distance from centre on both axes needed to reach a corner scenario |

## 6. Data and evidence

The numbers in the model are placeholders. The text refers to real frameworks and institutions, and each should be checked against the paper and primary sources before use:

- Compromiso de Sevilla, paragraphs 17 and 32(e)–(f) (round 1 briefing)
- COP31 outcomes (round 1)
- Antigua and Barbuda Agenda for SIDS, period ending 2034 (round 2)
- Loss and Damage Fund (round 2)
- CCRIF, PCRIC, the Pacific Resilience Facility and the SIDS Centre of Excellence (roles and options)
- The Multidimensional Vulnerability Index (MVI): the model assumes it shapes concessional terms, which the paper leaves open
- The four ESB recommendations (a debrief question): taken from the brief, not defined in the code

Scenario names, the two axes, the three wildcards and the no-regret categories follow the paper. The two off-diagonal corners are a prototype choice.

## 7. Model

### Drivers and states

Five drivers come from section 3 of the paper: climate hazard and loss; SSTrC and international cooperation; finance architecture and private capital; regional institutions and SIDS capacity; technology and data. Each has three states, named by likelihood of the favourable trajectory for SIDS:

- **Likely:** the favourable trajectory.
- **Neutral:** today's trend.
- **Unlikely:** the adverse trajectory.

At setup, each driver gets one choice. The choice sets its state for each period (`DPAT` in the code):

| Choice | 2026–2029 | 2030–2034 | 2035–2040 |
|---|---|---|---|
| Likely (cooperation, finance, institutions, data) | Neutral | Likely | Likely |
| Neutral | Neutral | Neutral | Neutral |
| Unlikely (cooperation, finance, institutions, data) | Neutral | Unlikely | Unlikely |
| Hazard, Likely | Neutral | Neutral | Neutral |
| Hazard, Unlikely | Neutral | Neutral | Unlikely |

Hazard has no Likely effect in the model because the paper treats sea-level rise as locked in. The evidence for these choices is in the Stakeholder behaviour document (section 16).

Each state adds fixed placeholder effects to the round (`DFX`). Neutral adds nothing.

### Scenarios

The final position on the two axes sets the scenario:

- **Cooperative Resilience:** both axes at or above +7.
- **Retrenchment and Exposure:** both axes at or below −7.
- **Fragmented Patchwork:** everything else. This includes the two off-diagonal corners, which the paper does not describe. Whether they deserve their own scenario is an open decision for the team.

### Reactions (Site B only)

- **Trust below 40:** linked decisions that need several roles are blocked.
- **Debt stress 70 or more:** options that add debt are locked.

These are checked against the values at the start of each round.

### Linked decisions

A linked decision forms when every required role chose a qualifying option. On Site A, qualifying means the option's position. On Site B, it means the option's coalition key, so the decision still forms when an option changes with the driver state. Linked decisions give a fixed effect when they form.

### Site B: decisions

- **Options by driver state.** Each role has three options for each state of its primary driver. The Neutral state uses the Neutral set.
- **Continuations (rounds 2 and 3).** The first option of a role follows its own previous action (for example, a pool move leads to a pool continuation). The continuation replaces the first state option. The other two options still depend on the driver state.
- **Follow-up questions (rounds 2 and 3).** Each role's question depends on its own previous action.
- **Direct links.** A small set of role pairs react to each other: the role on the target side sees one extra line only when the source role took the relevant action. Other roles do not react.
- **Hidden choices.** None. The facilitator sees each choice as it is made, and the effects are applied at Resolve.

## 8. Key functions

All in the `<script data-dc-script>` block. The table covers the functions of the original and the ones added for the two sites (the last six rows).

| Function | Purpose |
|---|---|
| `o(t, d, fx, tag)` | Shorthand to write an option |
| `roleName(id)` | Looks up a role's display name |
| `fxText(fx)` | Formats effects for display, for example `Cooperation +1 · Debt −3` |
| `classify(a, b)` | Decides the scenario: `'s1'`, `'s2'` or `'s3'` |
| `pct(v)` | Converts an axis value (−25…25) to a plot position (0–100%) |
| `fresh()` | Initial state (also used by Reset) |
| `apply(v, fx)` | Adds effects with sensitivity and clipping |
| `autoWild(round, st)` | Fires the habitability wildcard automatically, if set |
| `resolve()` | Resolves the current round: background, drivers, options, linked decisions |
| `next()` | Moves to the next round, or to the debrief |
| `fire(w)` | Injects a wildcard. A used wildcard is undone when clicked again (`unfire`) |
| `stateOf(st, id, i)` | Returns a driver's state in a given round |
| `opts(roleId, i)` | Returns a role's three options for a round (Site B: state, continuation) |
| `prevAct(roleId, i)` | Returns the action type of a role's previous round |
| `coalsOf(i)` | Returns the linked decisions for a round |
| `matches(roleId, i, idx, want)` | Checks whether a chosen option matches a linked decision's key |
| `renderVals()` | Computes every value the layout shows, and the click handlers |

## 9. Screens and interaction

| Screen | Controls | Bound to |
|---|---|---|
| Header tabs | Simulation, Drivers, Effects annex (Site B only), Paper gap analysis | `tab` |
| Setup (roles) | Role cards (click to toggle in or out); Next · choose drivers (needs one role) | `active`; `screen → 'drivers'` |
| Drivers of change | Five driver cards, each with three states and their descriptions; Back to roles; Begin round 1 | `drivers`; `screen → 'play'` |
| Play: briefing | Read-only: title, brief, drivers and their effect this round, background trend, reactions, collective signal, milestones, signposts | `ROUNDS[round]`, `drivers` |
| Play: role cards | Click one option per role; decision question and decision point on each card | `choices[round]` |
| Play: Resolve | Enabled once every role has decided | `resolve()` |
| Play: result panel | Change split into background trend, drivers and decisions; linked decisions formed or not | `result` |
| Sidebar: plot | 2×2 scenario space; dot = current position, grey dots = history | `vals.a`, `vals.b`, `trail` |
| Sidebar: indicators | Bar = value; colour = change since 2026 (teal good, orange bad) | `vals` |
| Sidebar: wildcards | Inject, USED (click again to undo), Reset entire simulation | `fire()`, `restart()` |
| Debrief | Outcome, no-regret counts, debrief questions, decision log, timeline, ESB test, role statements, limits, Convert-to-text narrative, Run again | `classify(vals)`, `choices`, `hist` |

## 10. Design system

The palette comes from the Evidence & Solutions Brief, adjusted for reading on screen:

| Role | Colour |
|---|---|
| Page background | `#F7F9FC` |
| Cards | `#FFFFFF` |
| Primary text | `#1F2A37` |
| Secondary text | `#4A5A6A` |
| Selected button, current scenario | `#00335B` (navy) |
| Header accent | `#009ED8` |
| Likely, Cooperative Resilience, improved | `#00A19B` (teal) |
| Unlikely, Retrenchment, worsened | `#F58220` (orange) |
| Fragmented Patchwork | `#0072BC` (blue) |
| Borders and dividers | `#C9D6E3`, `#DCE5EF` |

Fonts: IBM Plex Sans (body) and IBM Plex Mono (labels and numbers), both local in `fonts/`. The brief uses a serif for titles; that is not applied yet.

Styling is inline `style="…"` on each element. Colours are typed in place, so changing the palette means changing the values in the file, not in one place.

## 11. Conventions for contributors

- **Class name.** The class must stay named `Component`.
- **Logic and layout.** Logic lives in the script block. The layout only reads named values from `renderVals()`, because the `{{ }}` placeholders cannot contain expressions.
- **Styling.** Inline `style="…"` only, reusing the palette in section 10.
- **Runtime.** Do not edit `support.js`.
- **Naming.** Short lowercase ids for roles (`car`, `pac`, `ais`, `mdb`, `ins`, `reg`, `lm`, `unossc`, `un`) and drivers (`haz`, `coop`, `fin`, `inst`, `tech`). User-facing labels go in the text fields.
- **Adding a role.** Add it to `ROLES`, its options to every round on Site A, its state sets to `B_OPTIONS`, `B_KEYS`, `B_ACTS` and `B_TAGS` on Site B, its follow-up questions (`QF`), continuations (`FO`) and questions (`QUESTIONS`), and its texts in `BEHAV`, `ROLE_LIMITS`, `DECISION_TR`, `NARR` and the ESB table. Then update the "of 9" labels and regenerate Site B.
- **Adding a driver.** Add it to `DRIVERS`, `DRV_ROLE`, `DPAT`, `DFX`, and the Drivers tab. The setup, the briefing and the timeline read from `DRIVERS`. This is the step we decided against for debt; it is still possible.
- **Adding a wildcard.** Add `{id, n, fx, text, prompts}` to `WILD`.
- **Adding a linked decision.** Add `{n, req, fx}` to a round's coalitions on Site A. On Site B, add the entry to `B_COALS` with keys, and give the relevant options keys in `B_KEYS`.
- **Changing scenarios.** Edit `SCEN`, `NARR` and the scenario rules in `classify`, then update the plot shading and the Drivers tab.
- **Adding a round.** Add to `ROUNDS`, and fix the hard-coded three-round places in section 12.
- **Changing content.** Change the Claude Docs documents first (section 16), then mirror the change in the code.

## 12. Known limits and bugs

**Carried from the original prototype**
- **Rounding.** `Math.round` is asymmetric for negative halves, so effects are biased slightly at sensitivities other than 1 or 2.
- **Three rounds are hard-coded** in `fresh()` (`choices`), in the decision log grid, and in the next-round label.
- **Linked decisions match by position on Site A.** Reordering options silently breaks them.
- **A wildcard fired before Resolve** is not in that round's result panel, because the "before" snapshot is taken at Resolve. A wildcard fired after Resolve changes values the result panel does not show.
- **Auto-wildcard is skipped silently** if the habitability wildcard was already fired by hand.
- **The Resolve button** looks greyed until everyone decides, but it still accepts a click and then does nothing.
- **Roles can only change on the setup screen.** If a role needed for a linked decision is out of play, that decision can never form.
- **The setup label "of 9 selected" is hard-coded.**
- **Accessibility is not reviewed.** Selected states rely on colour and border; buttons have no `aria-pressed`.

**Added in this version**
- **Undo of a wildcard is not exact** when an indicator is at a bound (0, 100, or ±25), because clipping happens after each effect.
- **Clipping order matters.** Effects are applied one at a time and clipped after each, so the order in which they apply changes the result when a value reaches 0, 100 or ±25.
- **Indicators feed back only on Site B, and only as reactions.** Trust and debt lock decisions there. On Site A, indicators are reported but never change the options or the scenario. The scenario depends only on the two axes on both sites.
- **The gap analysis tab is static text.** It does not update when the model changes, so it must be edited by hand.
- **Reset returns the drivers to Neutral**, and the roles to unselected.
- **Site A has no follow-ups, continuations, reactions, track records or annex.** Its options and questions are the same in every round.
- **Effects annex** lists every option and continuation from the same data the game uses. It does not list linked decisions.
- **Debrief no-regret counts** on Site B use the tags on options. Options without a tag are not counted.
- **Narrative and analysis** in the debrief are generated from the run's data with fixed sentences. They describe what happened; they do not argue for a conclusion.
- **No save.** A page refresh loses the session.
- **No tests.** Nothing checks `apply`, `classify`, coalition formation or the follow-up logic automatically.
- **Requires internet** for React, Babel and the fonts' CDN loaders.

## 13. Alignment with the paper

Covered: the five drivers, the three scenarios and their rules, the three wildcards, the scenario debrief questions, the no-regret categories, a role for each stakeholder group in the paper's audience, decision points with their period and trade-off, stakeholder resources and red lines, signposts with indicator, source and threshold (Drivers tab), a test of the four ESB recommendations in each scenario, a round calendar with the milestones of each period, a statement for each role in each scenario reached, a timeline of the run, and the limits of the run.

Partial: stakeholder roles are the draft from the paper's gap list and need team confirmation; the signposts' thresholds are placeholders, except where the brief gives a reference; the ESB test is a set of hypotheses from the brief, not checked in play; the 81 options (and 104 continuations on Site B) carry a period and a trade-off through their role card, not on each option.

Not included, by decision: debt as a sixth driver. Debt stress stays an indicator, with the reaction on Site B. Revisit only if the team asks for it.

Missing: a 2040 timeline of turning points as a visual (the timeline is a list); a facilitator guide inside the app (it is a separate document); an exercise evaluation inside the app.

## 14. Changes from the original README

The first README described the prototype as it was when the project started. This section lists what changed in the model and the product, and why.

| Original | Now | Why |
|---|---|---|
| Five scenarios: three from the paper and two unmapped quadrants | Three scenarios; the two off-diagonal corners fold into Fragmented Patchwork | The paper defines three; the team decides whether the quadrants deserve their own scenario |
| Options and states by pathway (Positive, Medium, Negative) | Each driver has its own choice (Likely, Neutral, Unlikely) | Players decide each driver on its own, as the paper's drivers do |
| Decisions the same in every round | Site B: options depend on the driver state, and on the role's previous action | The stakeholders react to context and to their own history |
| One version of the game | Two sites: fixed decisions (A) and decisions by driver state (B) | To compare fixed and driver-dependent decisions in the same game |
| Decision log and debrief | Plus timeline, ESB test, role statements, limits, narrative | The paper's structure needs these to read a run |
| Reactions to outcomes not modelled | Trust and debt reactions on Site B | Stakeholders react to what happened to the system |
| No track records | Track records and behaviour under pressure on Site B cards, from 1986–2026 | The paper asks what each stakeholder can realistically do |

Not carried over, by decision: the split of the content into separate data files (option A of the original). The code stays in one file.

## 15. Next steps

1. Restore the lost sections in the original: done in this version.
2. Add the sources tab for the Site B cards, and confirm the placeholder thresholds with the team.
3. Split the UN system into its agencies, and add the voice of communities and inclusion.
4. Fix the known bugs in section 12, starting with rounding and the hard-coded rounds.
5. Add a small set of automated tests for `apply`, `classify`, coalition formation and the follow-up logic.
6. Validate the model with two or three playtests before the full session.
7. Save sessions, so a page refresh does not lose a run.
8. Add facilitation aids: a negotiation timer, printable role cards (the card content is already in the app), and a short pre and post survey.
9. Make the app work offline by bundling the scripts and fonts locally. Only the fonts are bundled now; React and Babel still load from unpkg.

## 16. Related documents

- **Facilitator guide and evaluation (draft):** https://claude.ai/artifact/NZNHzU4JX8CzPWcapUvE2d
- **Driver pathways 2026–2040 (draft):** https://claude.ai/artifact/EsWPh7PQKVitQi9pGPSLUy
- **Stakeholder behaviour: recent evidence (2025–2026) and forty years of history:** https://claude.ai/artifact/JJk4pC1DAJcd6CYamaAqHj

These documents are the source for the driver states, the stakeholder track records and the reaction rules. Change them first, then mirror the change in the code.
