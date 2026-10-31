# FitFindr — Project Walkthrough

> **Who this is for:** someone who has never seen this repo, and doesn't
> necessarily know what "an agent" or "a tool call" means. Every technical term
> is explained the first time it shows up. Numbers come from real runs of real
> commands, not estimates — if something was only tried once, that's said
> plainly, and if a target was missed, that's said plainly too.
>
> **How this file is built:** maintained one milestone at a time, as a diary and
> a manual together. Earlier entries are never rewritten or reordered once
> written — only added to, the same rule this project's own `criteria.md`
> follows for revising a criterion instead of deleting it.
>
> **A note on scope:** the project brief and features text available when this
> file was created cover **Unit 3** only. Unit 4 has no separate formal
> "introduction" document the way Unit 3 does — its content enters this file
> through the Milestones section instead, built from the actual Unit 4
> milestone instructions and the real work already completed in this repo
> (search tool, planning loop, the MCP move, the test run, the diagnosis, the
> fix). Where Unit 4 material appears below, it's labeled as such.

---

## 1. Unit Introduction

### Unit 3 — Building an Agent

The project before this one (Units 1–2) answered questions. It ran the same two
steps in the same order every time: look something up, then write an answer
based on what it found. That's a **fixed pipeline** — useful, but it never has
to decide anything.

An **agent** is different. It decides what to do next based on what just
happened. If a search comes back empty, *now what?* If an item costs more than
the budget, *now what?* Those decisions live in code someone writes, not inside
the model itself — and getting them right is the actual engineering work of
this unit.

**What's being built:** FitFindr, an agent that takes a request like *"a
vintage graphic tee under $30, size M"*, searches a listings file, works out
what the item would go with, and writes a short caption for it. Three separate
functions ("tools"), a loop that picks between them based on what came back,
and a shared place to hold information so the user is never asked for the same
thing twice.

This project runs across **two units that do different jobs**:

| Unit | Job |
|---|---|
| **Unit 3** (this document's starting point) | Build the agent. Write five acceptance criteria — a stated, numeric definition of "working" — *before* knowing whether the agent will actually clear them. |
| **Unit 4** | Test the agent against those criteria, diagnose what's missed, fix one thing, and re-test. Same repository both units — it is never deleted and recreated. |

The criteria are harder to write this time, on purpose. In the project before
this one, most of the acceptance criteria were handed over already written.
Here, two are given and three have to be written from scratch — for a system
whose output genuinely changes between runs, since two of the three tools call
a language model. A criterion like *"returns a good outfit"* can't be checked
by a machine or agreed on by two people, so it's worthless as a criterion; part
of the unit's work is discovering that the hard way and replacing it with
something a specific number can be attached to.

**Why this matters (the brief's own framing, kept intact):** agents are what
most real AI product work looks like right now — a model with tools attached,
deciding what to call. The parts that break in production are almost never the
model itself. They're the loop that didn't handle an empty result, the state
that got silently dropped between two function calls, the tool description
that didn't say what it actually returns. Those are engineering problems, and
they're the ones this unit (and this document) is about.

### Learning objectives

By the end of Unit 3, the goal is to be able to:

1. Write a **tool spec** precise enough that something else (a model, or a
   person who never sees the code) could call the function correctly from the
   description alone.
2. Build a **planning loop** that branches on what the previous step returned,
   instead of calling every step regardless.
3. Carry **state** across tool calls, so information found by one step reaches
   the next one without the user re-typing it.
4. **Stop a loop cleanly** when a step comes back with nothing, rather than
   calling the next step with empty input.
5. Write **acceptance criteria** for a system whose behavior varies from run to
   run.

---

## 2. Project Title, Problem, and Goals

### FitFindr

**The problem it solves:** thrifting (buying secondhand clothes) takes real
effort. A person searches several apps, tries to picture how a found item
would look with clothes they already own, and guesses whether the asking price
is reasonable. Most of that effort is *looking things up* and *comparing
them* — exactly the kind of repetitive decision-making an agent can take over.

**What it does:** someone describes what they want — *"a vintage graphic tee
under $30, size M"* — and FitFindr searches a listings dataset, works out what
outfit the item would complete using the person's own wardrobe, and writes a
short caption ("fit card") someone would actually post about the find.

### Goals

By the end of this unit, the project demonstrates the ability to:

- Write a tool spec that states exactly what goes in and what comes out.
- Build a loop that picks its next step from what the previous step returned.
- Carry information from one tool call to the next without asking the user to
  repeat themselves.
- Write acceptance criteria for a system whose behavior changes from run to
  run (because part of it calls a language model).

---

## 3. Glossary

Terms used throughout this document and the project itself. New terms are
added here as later milestones introduce them — this table grows; it's never
shortened.

| Term | What it means | Where it shows up in FitFindr |
|---|---|---|
| **Agent** | A program that decides which steps to take, instead of running a fixed list in a fixed order every time. | `agent.py::run_agent` — the whole reason this project is "an agent" and not a script. |
| **Tool** | One function the agent can call. It takes named inputs and returns something specific — not "some text," but a documented shape. | The three functions in `tools.py`: `search_listings`, `suggest_outfit`, `create_fit_card`. |
| **Tool call** | One actual run of one tool, with real values passed in — as opposed to the tool's definition. | E.g., calling `search_listings("vintage graphic tee", max_price=30)` once. |
| **Planning loop** | The code that looks at what a tool just returned and decides what to do next. | `agent.py::run_agent`'s body — it reads `session["search_results"]` before deciding whether to call `suggest_outfit`. |
| **Session state** | A shared place to keep what each tool returned, so the next tool can use it without the user repeating themselves. | The `session` dictionary created by `agent.py::new_session`. |
| **Branch** | A point where the loop takes one path or another depending on a condition. Without one, a "loop" is really just a list of calls that always all run. | The `if not session["search_results"]:` check in `run_agent` — the one required branch (empty search stops the loop). |
| **Happy path** | The run where everything works end to end. Easy to build; not the interesting engineering problem — the branches and failure handling are. | A query that matches a real listing, completing all three tools and returning a fit card. |
| **Listing** | One item for sale in the project's dataset — not a live marketplace, a fixed JSON file standing in for one. | A single entry in `data/listings.json`, with fields like `title`, `price`, `size`, `platform`. Introduced in Milestone 1. |
| **Wardrobe item** | One piece of clothing a user already owns, in the shape `suggest_outfit` expects. | An entry in `data/wardrobe_schema.json`, or an empty `{"items": []}` for a new user. Introduced in Milestone 1. |
| **Empty case** | What a tool returns when it has nothing to give — decided and written down *before* the tool is built, not discovered as a crash later. | `search_listings` returns `[]`, never `None`, on no match. Introduced in Milestone 2. |
| **Acceptance criterion** | A stated, numeric definition of "working" — a target plus a reason — written before any test results exist, so it can't be quietly adjusted after the fact. | The five entries in `criteria.md`. Introduced in Milestone 3. |
| **Stub** | A placeholder function that runs and returns something, but doesn't yet do the real work — used to prove the shape of a tool before its logic exists. | How all three tools in `tools.py` started, before Milestone 4. Introduced in Milestone 4. |
| **Cache** (response cache) | Reusing an answer already received for an identical prompt, instead of calling the model again — free while building, turned off for real tests. | `CACHE_ENABLED` in `config.py`. Introduced in Milestone 4. |
| **Temperature** | How much a model's wording varies between calls with the same prompt. `0.0` repeats itself; higher values vary. | `TEMPERATURE` in `config.py`, set to `0.9`. Introduced in Milestone 4. |
| **Stop condition** | A hard limit that ends a loop even if its own branches never do — a safety net, not the main logic. | `config.MAX_ITERATIONS`, checked by `trace.check_iterations()` in `run_agent`. Introduced in Milestone 5. |

---

## 4. How to Start the App

Every command below was run for real, from inside this repository on Windows,
while writing this document — not copied from other documentation unverified.
Full, general setup instructions (Python, Git, the API key, the course-wide
conventions) live in [`ENVIRONMENT_SETUP.md`](ENVIRONMENT_SETUP.md); this
section is the minimum to actually see the app do something.

**1. Confirm the environment is ready:**
```bash
python test.py
```
A real run of this, including the one gap in this particular working copy
(no virtual environment — see `ENVIRONMENT_SETUP.md`), is recorded there in
full.

**2. See what one listing and one wardrobe item actually look like:**
```bash
python app.py fields
```
Real output:
```
A listing has these fields:

  id             str    lst_001
  title          str    Vintage Levi's 501 Jeans — Medium Wash
  description    str    Classic 501s in a perfect medium wash. Some light fading a…
  category       str    bottoms
  style_tags     list   ['vintage', 'classic', 'denim', 'streetwear']
  size           str    W30 L30
  condition      str    good
  price          float  38.0
  colors         list   ['blue', 'indigo']
  brand          str    Levi's
  platform       str    depop

A wardrobe item has these fields:

  id             str    w_001
  name           str    Baggy straight-leg jeans, dark wash
  category       str    bottoms
  colors         list   ['dark blue', 'indigo']
  style_tags     list   ['denim', 'streetwear', 'baggy']
  notes          str    High-waisted, sits above the hip
```

**3. Browse a few of the listings the agent searches over:**
```bash
python app.py listings -n 5
```
Real output:
```
40 listings.

id       price  size                  platform   title
--------------------------------------------------------------------------------------------
lst_001   38.00  W30 L30               depop      Vintage Levi's 501 Jeans — Medium Wash
lst_002   18.00  S/M                   depop      Y2K Baby Tee — Butterfly Print
lst_003   22.00  XL (oversized)        thredUp    Oversized Flannel Shirt — Plaid Red/Bl
lst_004   45.00  M                     poshmark   90s Track Jacket — Navy/White Stripe
lst_005   32.00  W28                   depop      Corduroy Wide-Leg Pants — Rust

… 35 more. Use -n 40 to see them all.
```

**4. Run the actual agent on a real request:**
```bash
python app.py ask 'vintage graphic tee under $30'
```
⚠️ **Single quotes are required, on every operating system** — see the common
failure below.

Real output (trimmed to the parts that matter; a full trace-level version is in
Milestone entries below once added):
```
[2] search_listings (via MCP)
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    branch: results found, continuing to suggest_outfit

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Fit card: Literal peak 2000s energy right here with the prettiest pink and purple butterfly graphic. Snagged this mint-condition baby tee for $18.00 and I'm about to live in it with baggy denim and chunky sneakers all season. Go check my Depop before I change my mind and keep it.

2 model calls this session, 763 prompt + 361 output tokens
```

### The common failure worth knowing about before it happens

An invalid or misconfigured `GEMINI_API_KEY` is the most likely thing to go
wrong when `suggest_outfit` or `create_fit_card` runs — both call the hosted
model, and a bad key surfaces there, not at startup. The full real diagnosis
(what the error looked like, what was actually wrong with `.env`, and the fix)
is recorded under **Milestone 4** below, since that's the step where it
actually happened — not repeated here.

---

## 5. What I Learned

*To be filled in as the project progresses.* This section is meant to
synthesize real lessons across every milestone, not to restate the per-step
findings — so it's intentionally left as a placeholder until there's a body of
completed milestones to draw a synthesis from, rather than written in advance
and padded out to look complete.

---

## 6. Showcasing the Learning

*To be filled in as the project progresses.* This section will hold concrete
evidence — real command output, real run logs, real before/after
comparisons — pulled from the Milestones section once it has entries, rather
than described in the abstract ahead of time.

---

## 7. Milestones

Documented retroactively from this repository's real history — git commits,
`README.md`, `criteria.md` — rather than redone from scratch, per the decision
made for this project. Every claim below about what changed, when, and in
which commit was checked against the actual repository, not recalled from
memory. Where the real history has a gap or an irregularity, it's named rather
than smoothed over.

### Milestone 1: Read the Data and Run the Starter

**The idea.** A tool can't be specified honestly against data nobody has
looked at. Reading real listings and the real wardrobe shape before writing
`search_listings`'s filters — rather than guessing at field names from the
brief alone — avoids building a tool against an imagined schema and
discovering the mismatch as a bug later.

**Where the answers went.** Nowhere, by design — this milestone is read-only
exploration, so it produced no code to commit. Checked against the real git
history: no commit exists that's uniquely attributable to this step. The
nearest commit in time, `7f8e4a4` ("updates," 2026-08-24, touching
`RUNNING.md`, `agent.py`, `generate.py`, `mcp_server.py`, `requirements.txt`,
`serve.py`), reads as course-starter upkeep rather than this milestone's own
output. That's expected, not a gap: a step whose only actions are running
existing commands and reading existing files has nothing to commit.

**Step: read five or six listings, record the field names.**
- What was done: `python app.py fields` and `python app.py listings --full -n 6`
  (both shown with real output in Section 4 above — not repeated here).
- Why this way, not by opening the JSON file directly: the CLI prints each
  field's Python type alongside its value, which the raw JSON doesn't.
- Result: eleven fields recorded — `id`, `title`, `description`, `category`,
  `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, `platform`.

**Step: look at the wardrobe shape, note the empty case.**
- What was done: read `data/wardrobe_schema.json` and
  `utils/data_loader.py::get_empty_wardrobe()`.
- Result: a wardrobe item has `id`, `name`, `category`, `colors`,
  `style_tags`, `notes` — a different shape from a listing (no `price`, no
  `platform`). An empty wardrobe is `{"items": []}`.

**Step: run the starter on one query.**
- Command: `python app.py ask 'vintage graphic tee under $30'`.
- Result at the time: *"The planning loop isn't built yet — see the TODO in
  agent.py."* — the correct starting position per the milestone's own
  framing, not a failure.

**What this means for the next milestone.** The eleven listing fields
recorded here are exactly the dictionary schema Milestone 2's Tool Inventory
promises `search_listings` will return.

**Terms from this milestone:** Listing, Wardrobe item (see Section 3).

**Milestone summary.** True now: the starter runs, and the real shapes of a
listing and a wardrobe item are on record. Still a placeholder, on purpose:
nothing is built yet — `tools.py` is still stubs.

**What this milestone taught:**
1. A tool spec written from the brief alone would have guessed wrong on size
   formatting — real sizes include `"W30 L30"` and `"S/M"`, not just
   `"M"`.
2. The "correct" starting position can look identical to a failure at a
   glance — the TODO message here is a pass, not a bug.
3. A listing and a wardrobe item share some field names (`category`,
   `style_tags`, `colors`) but are not the same shape — conflating them in
   Milestone 2's spec would have broken on the first real wardrobe lookup.

**Before moving on** (the milestone's own checklist, checked truthfully):
- *"The starter runs."* — **True**, reconfirmed by re-running it while writing
  this document.
- *"You can name three fields in the listings file from memory."* — not
  something a repository's contents can verify on its own; treated as true
  since Milestone 2's Tool Inventory later names all eleven fields correctly.
- *"Commit and push to your fork."* — **No commit in the real history is
  uniquely attributable to this milestone** (see above). Documented honestly
  rather than invented, since a read-only step genuinely has nothing to
  commit.

**If stuck:** the milestone's own fallback — post the exact error, move on to
Milestone 2 — wasn't needed; the starter ran on the first attempt.

---

### Milestone 2: Spec the Three Tools

**The idea.** Writing down exactly what each tool takes and returns — before
writing the tool itself — forces a decision a vague return value would
otherwise dodge. A loop can only branch on something definite; "returns a
list" gives it nothing to check.

**Where the answers went.** README.md's **Tool Inventory** section (the
four-line spec per tool, plus the Agent Loop Branching Rule). Real commit:
`a1f8f7e` (2026-09-30), *"README.md Tool Inventory (draft section; needs
cleanup)"* — README +82/−16 lines, `tools.py` +1/−1 (a minor stub comment, not
an implementation). **One commit covers all three tool specs and the branch
rule together**, not one commit per tool — a real departure from "each step
committed on its own," named here rather than smoothed over. The commit's own
message calls it a draft; it was further condensed three days later, folded
into the commit that finished Milestone 3, once writing acceptance criteria
forced the tool contracts to be read again critically.

**Step: four lines per tool.** The real, current spec for each (from README's
Tool Inventory):

| Tool | What it does | Inputs | Returns | Empty case |
|---|---|---|---|---|
| `search_listings` | Searches listings by keyword, with optional size/price filters | `description: str`, `size: str \| None`, `max_price: float \| None` | `list[dict]`, each with 11 named fields, sorted by match score | `[]` — never `None`, never an exception |
| `suggest_outfit` | Pairs a selected item with wardrobe pieces using the model | `new_item: dict`, `wardrobe: dict` | Non-empty `str` naming specific owned pieces | General styling advice (still non-empty) when `wardrobe['items']` is `[]` |
| `create_fit_card` | Writes a short, postable caption for the find | `outfit: str`, `new_item: dict` | `str`, 2–4 sentences, mentions title/price/platform once each | A descriptive fallback string if `outfit` is empty/whitespace |

Decision made explicit for the empty case, since it's the one the milestone
calls out as commonly lost:

| Option (search_listings, no match) | Why not |
|---|---|
| "Returns a list" | Doesn't say what's in it — the exact failure the milestone names |
| Return `None` | Forces the loop's branch to check two things (`is None` and "is it empty") instead of one |
| Return `[]` | **Chosen** — one check (`if not results`), and it matches how Python already treats an empty list as falsy |

**Step: write the one branch, as a rule.** Recorded verbatim in README's
"Agent Loop Branching Rule": *"If `search_listings` returns an empty list,
put a message in the session and stop. Otherwise, take the first result and
go to `suggest_outfit`."* — stated with both the condition and both paths, so
it can be checked against later code mechanically rather than interpreted.

**How it was checked.** Against the milestone's own test: *"could someone
else build this tool from what I wrote, without asking me anything?"* —
recorded as a yes in README's "Specification Audit" subsection, reasoned from
the fact that every contract names exact Python types, full dictionary keys,
and a decided empty-state return.

**What this means for the next milestone.** The branch rule written here is
the literal sentence Milestone 5 later implements in `agent.py::run_agent`,
and the `[]`-not-`None` decision is the exact contract Milestone 4's
implementation of `search_listings` has to honor.

**Terms from this milestone:** Empty case (see Section 3).

**Milestone summary.** True now: three tool specs exist with named, typed
inputs, concrete return shapes, and a decided empty case each; one branch
rule is written as a rule. Still a placeholder: no implementation exists yet.

**What this milestone taught:**
1. "Returns a list" reads like a spec and isn't one — nothing about it lets a
   branch be written against it.
2. Stating the branch rule as a rule (condition *and* both paths), not a
   description, is what makes it checkable against real code later instead
   of just readable now.
3. Specs drift from their own draft quickly — this one needed cleanup three
   days later, once Milestone 3's criteria work forced a second, more
   critical read.

**Before moving on:**
- *"All three tools have inputs with types, a specific return value, and an
  empty-case answer."* — **True**.
- *"Your branch rule is written as a rule."* — **True**.
- *"Commit."* — **True**: `a1f8f7e` — one commit for the whole milestone, not
  one per sub-step.

---

### Milestone 3: Write the Acceptance Criteria

**The idea.** An acceptance criterion names a target specific enough that
someone else could check it — a number or a plainly observable outcome, never
an opinion like "the agent handles errors." Writing these *before* any test
exists is deliberately uncomfortable: it's the only way a later "basically
works" can't quietly redefine what "working" meant.

**Where the answers went.** `criteria.md` — all five criteria and their
reasons. Real commit: `55ee4f7` (2026-10-02), *"citeria.md Complete"* (the
project's own commit message, typo included — left exactly as it is here,
since this document's job is accuracy, not tidying up history).
`criteria.md` +41/−27 lines; README −97/+27 (the Tool Inventory draft from
Milestone 2 condensed as a side effect of this same commit).

**Step: read the two given criteria.**

| # | Criterion | Target | Why this target (as written) |
|---|---|---|---|
| 1 | A matching query completes all three tools and returns a fit card | **4 of 5** | `search_listings` is plain keyword overlap; natural phrasing or synonyms can fail to overlap with dataset terms and cause a false empty result |
| 2 | An impossible query stops before `suggest_outfit`, naming what to change | **5 of 5** | The empty-state branch is a deterministic `if not listings:` check, not an LLM decision — a miss here would be an unambiguous code defect |

**Step: write three criteria from scratch.**

| # | Criterion | Target | Why this target (as written) |
|---|---|---|---|
| 3 | `session["selected_item"]["id"]` matches the `id` on the dict actually passed to both `suggest_outfit` and `create_fit_card` | **5 of 5** | State persistence is a deterministic dict assignment (`session["selected_item"] = results[0]`) — any mismatch would be a critical data-flow bug, not an occasional one |
| 4 | `create_fit_card` returns a 2-to-4 sentence caption explicitly mentioning the item's title, price, and platform | **4 of 5** | The tool calls a model with `TEMPERATURE > 0.0`; prompt engineering enforces this, but occasional stochastic variation could still drop a field |
| 5 | `suggest_outfit` with an empty wardrobe returns non-empty general styling advice, no error | **5 of 5** | Gated by a deterministic `if not wardrobe.get("items"):` check before the prompt is even built |

Criterion 3 is the required "state" criterion (*"how would you know the item
search found is the same item the next tool received?"*), and criterion 4 is
the required "fit card" criterion (*"the same input can give different words
each time — what would make that acceptable?"*). Criterion 5 was the free
choice:

| Candidate third (free-choice) criterion | Why not chosen |
|---|---|
| Price ceiling respected exactly | Already implicit in Milestone 2's spec for `search_listings`; less distinct to test on its own |
| Response speed | Not a concern the brief or the design treats as load-bearing — would be a number picked for its own sake |
| Empty wardrobe path | **Chosen** — exercises a real branch inside `suggest_outfit`, with the same deterministic-code reasoning already used for criterion 2 |

**Step: the self-check.** Rather than asking an AI tool to draft the
criteria — explicitly against the milestone's own instruction — each written
criterion was checked for whether it's testable from its sentence alone, with
no extra context. Criterion 3 is the clearest example of why this step
matters: per the milestone's own framing, *"state failure doesn't look like
state failure — it looks like a tool problem."* That turned out to be exactly
right; Unit 4's later testing could only confirm the claim with a deliberate
spy wrapped around both tools, not by eyeballing the output.

**What this means for the next milestone.** Criterion 2's exact wording (stop
before `suggest_outfit`, name what to change) becomes the literal error
message `agent.py` produces in Milestone 5. Criterion 3 foreshadows exactly
how Milestone 5's "is it really the same item" check has to be done — by
comparison, not by inspection.

**Terms from this milestone:** Acceptance criterion (see Section 3).

**Milestone summary.** True now: five numbered criteria exist, each with a
stated target and a reason tied to the mechanism behind it (deterministic
code vs. model output). Still a placeholder: no results exist against them
yet — that's Unit 4's job, on purpose.

**What this milestone taught:**
1. A criterion that can't be tested from its own sentence isn't a criterion
   yet — this is the literal test the milestone assigns, and it has real
   teeth.
2. The reasoning in "why this target" matters more than the number itself —
   4-of-5 vs. 5-of-5 only makes sense once it's clear whether a model or a
   deterministic branch sits behind the behavior.
3. Writing criteria before any results exist is uncomfortable on purpose —
   it's the only way a later miss can mean something.

**Before moving on:**
- *"criteria.md has five numbered criteria."* — **True**.
- *"Each names a number or something a person could plainly observe."* —
  **True**.
- *"Each has a reason underneath."* — **True**.
- *"Commit."* — **True**: `55ee4f7`.

**If stuck:** the milestone's fallback is a breakout discussion, or — working
alone — asking an AI tool to say exactly how it would test each criterion
without suggesting improvements. Not needed here; the five criteria as
written held up on first review (see the Specification Audit note in
Milestone 2).

---

### Milestone 4: Build the Three Tools

**The idea.** Build and test each tool alone, before any loop joins them.
Three untested tools wired into a loop isn't three problems — it's one
problem wearing six disguises, because a failure anywhere could be any of
four different layers (a tool, the loop, the session, or the model).

**Where the answers went.** `tools.py` (all three implementations);
`config.py` and `generate.py` (two small fixes found while testing); README's
**Sample Run** section (the three per-tool terminal tests). Two real commits:
`739c422` (2026-10-02, *"Defined 3 funcs in tools.py..."*, `tools.py` +110
lines) for the implementations, and `4220e00` (same day, *"Robust fixes to
config.py and generate.py, ran tests on the tools.py's 3 ai agent tools"*)
for the fixes found while actually running the tests this milestone
requires. As with Milestone 2, this is two commits for the whole milestone,
not one per tool.

**Step: `search_listings`.**
- What changed: built against `utils/data_loader.py::load_listings()` (not
  reading the file directly, per the instruction); filters on `max_price`
  (inclusive) and `size`; scores the remaining listings by keyword overlap
  with `description`; returns `[]` — never `None` — when nothing scores
  above zero.
- Decision made on size matching:

  | Option | Why not / chosen |
  |---|---|
  | Plain substring match (`"s" in size.lower()`) | Matches `"S"` against `"US 9"`, and `"L"` against `"XL"` — the exact trap the starter's own comments warn about |
  | Token-based match (split on `/`, compare whole tokens) | **Chosen** — `"M"` correctly matches `"S/M"` without `"S"` ever falsely matching `"US 9"` |

- How checked:
  ```
  $ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
  ```
  Real output (from README's Sample Run): six matching listings returned,
  every one at or under $30, confirming both the keyword match and the price
  filter.

**Step: `suggest_outfit`.**
- What changed: branches on `wardrobe.get("items")` — empty gives general
  styling advice; non-empty formats each owned item into the prompt and asks
  for outfits that name specific pieces.
- How checked:
  ```
  $ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
  ```
  Real output (from README's Sample Run): two outfit options, each naming
  real wardrobe pieces by name (*"White ribbed tank top," "Vintage black
  denim jacket"*).

**The real failure hit at exactly this step.** This command first failed
with `google.genai.errors.ClientError: 400 INVALID_ARGUMENT ... API key not
valid`, even though `.env` looked present and correct at a glance.
- **Diagnosis:** reading the raw contents of `.env` — not skimming it — found
  two separate problems at once: a second key saved under the **misspelled
  variable name** `GEMINI_API_KEY1` instead of `GEMINI_API_KEY` (so the
  correctly-named line was never actually holding the real key), and both
  keys starting with `AQ.Ab8RN6...` rather than the standard Google AI Studio
  prefix `AIzaSy...` — a sign of the wrong *kind* of credential entirely, not
  just an expired one.
- **The fix:** get the genuine key from aistudio.google.com (confirming it
  starts with `AIzaSy...`), and replace `.env`'s contents with one,
  correctly-named `GEMINI_API_KEY=` line.
- **The generalizable lesson:** "the key is present in the file" and "the key
  is readable under the exact name the code looks for, and is the right kind
  of credential" are two different claims. Only the second one matters, and
  the only way to check it is to read the literal file contents, not assume
  them from the filename being right.

**Step: `create_fit_card`.**
- What changed: guards against an empty or whitespace-only `outfit` (returns
  a descriptive fallback message instead of raising); otherwise builds a
  prompt from the item's details plus the outfit text and calls the model.
- The milestone's own specific check — run it three times on the same item,
  read the three outputs: the first attempt came back **word-for-word
  identical** across all three runs, exactly the two-suspect case the
  milestone names.

  | Suspect | Checked in | Result |
  |---|---|---|
  | `CACHE_ENABLED` | `config.py` | `True` by default — an identical prompt reuses a cached answer while building |
  | `TEMPERATURE` | `config.py` | `0.9`, not `0.0` — ruled out |

  Confirmed by forcing the cache off (`AI201_CACHE=0`) and re-running: three
  genuinely different captions came back, proving the cache — not the
  temperature — explained the identical result.
- Two small, pre-existing robustness issues were found and fixed while
  running these tests (not introduced by this milestone, just surfaced by
  it):
  - `generate.py`: a type-checker error on `call_config["system_instruction"]
    = system` (the dict's inferred type didn't allow a `str` value) — fixed
    with an explicit `dict[str, float | str]` annotation; confirmed to clear
    with no change in runtime behavior.
  - `config.py`: a type-checker false positive on `_stream.reconfigure(...)`
    (the `TextIO` type stub doesn't declare that method, though the real
    object has it) — suppressed with a targeted `# type: ignore[attr-defined]`,
    since the surrounding `try/except` already handles the real runtime risk.
- How checked:
  ```
  $ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
  ```
  Real output (from README's Sample Run): *"Found these broken-in vintage
  Levi's 501 jeans on Depop for $38 and my closet has never looked
  better..."*

**What this means for the next milestone.** All three tools now behave
exactly as Milestone 2 specified — Milestone 5's loop can branch on
`search_listings`'s `[]` return without guessing what "nothing found" looks
like in code.

**Terms from this milestone:** Stub, Cache, Temperature (see Section 3).

**Milestone summary.** True now: all three tools run from the terminal and
match their Milestone 2 specs, including their empty cases. Still a
placeholder: they aren't wired into a loop yet, so the agent as a whole still
does nothing end to end.

**What this milestone taught:**
1. Testing each tool alone surfaces and correctly attributes problems a
   wired-up loop would have hidden — the API key failure showed up here as a
   clean, specific error; inside a full loop it could have looked like "the
   agent is broken" with no obvious cause.
2. "It returned the same thing three times" has exactly two likely causes on
   this stack, and checking both takes under a minute, because `config.py`
   puts them both at the top on purpose.
3. A type-checker warning and a runtime risk are not automatically the same
   thing — the `config.py` warning was already covered by a real
   `try/except`; it needed a label, not a rewrite.
4. "The key is in `.env`" and "the key is correct" are different claims — a
   misspelled variable name and a wrong credential format both look like "a
   key is present" at a glance.

**Before moving on:**
- *"Each tool runs from the terminal and returns what your spec said it
  would, including its empty case."* — **True**.
- *"Commit."* — **True**: two commits, `739c422` and `4220e00`, together
  covering the whole milestone.

**If stuck:** the milestone's fallback is a hardcoded stub matching the
spec's shape, for a tool still broken after 40 minutes. Not needed here — all
three tools were completed; the one real obstacle (the API key) was a
configuration problem, fully resolved within this same milestone rather than
carried forward.

---

### Milestone 5: Wire the Loop and the State

**The idea.** Connect the three tools with a loop that reads the result of
the last call before deciding the next one. Without that, calling all three
tools regardless of outcome is a list of function calls wearing a loop's
clothing — not an agent.

**Where the answers went.** `agent.py::run_agent` (and its helper
`agent.py::_parse_query`); README's **Planning Loop** section. Real commit:
`5ec9944` (2026-10-05, *"Defined run_agent() in agent.py, and README
additions"*) — `agent.py` +86 lines, README +341/−30. One commit for the
whole milestone.

**Step: fill in `run_agent()`, following the Milestone 2 branch rule.**
- What changed: `new_session()` builds the session dict (`query`, `parsed`,
  `search_results`, `selected_item`, `wardrobe`, `outfit_suggestion`,
  `fit_card`, `error`); `run_agent()` parses the query, calls
  `search_listings`, and branches exactly on the Milestone 2 rule — empty
  results set `session["error"]` to a message naming what to change and
  return immediately; otherwise take the first result and continue to
  `suggest_outfit`, then `create_fit_card`.
- Decision made on parsing the plain-language query:

  | Option | Why not / chosen |
  |---|---|
  | Ask the model to parse it | Costs an extra model call for something deterministic, and adds a third source of run-to-run variation |
  | Hand-rolled string splitting | Workable, but regex handles the price-cue and size-phrase patterns more precisely with less ad hoc code |
  | Regex | **Chosen** — deterministic, free, and this query shape (a price cue, an optional "size X," leftover keywords) fits a small, fixed set of patterns |

- How checked: `python agent.py` runs both the matching-query and the
  impossible-query paths directly, printing the session at the end of each.

**Step: route every value through the session, not a carried-over local
variable.**
- What changed: each tool call reads its inputs back out of `session[...]`
  at the call site (e.g. `suggest_outfit(session["selected_item"],
  session["wardrobe"])`), rather than from a plain local variable kept from
  the previous line.
- Why this matters concretely: a local variable can't be inspected after
  it's been handed off and overwritten; a session value can be checked at
  any point — which is exactly what the next step depends on.

**Step: run the happy path, confirm the item's identity carries through.**
- Check performed: `session["selected_item"]` compared against the literal
  object `suggest_outfit` and `create_fit_card` actually received, using a
  spy wrapped around both functions — not a glance at matching titles.
- Real result: `item id in session: lst_002`, `item id passed to
  suggest_outfit: lst_002`, and the object-identity check (`is`, not just
  `==`) returned `True`. This is the direct, concrete evidence behind
  criterion 3's claim.

**Step: run an impossible query, confirm the stop is clean.**
- Real result: `fit_card: None`; `error: No listings matched. Try a higher
  price ceiling, a different size, or broader keywords in the description.`
  — satisfying both the milestone's check (`fit_card` stays `None`) and its
  explicit warning that *"'No results' is not that message"* by naming three
  concrete things to try instead.

**What this means for the next milestone.** The session fields populated
here are exactly what Unit 4's later testing inspects and prints in its run
logs — nothing needed to be added to the session's shape for that work.

**Terms from this milestone:** Stop condition (see Section 3).

**Milestone summary.** True now: a full query runs through all three tools
with values moving visibly through the session, and an impossible query
stops early with a message naming what to change. Still a placeholder: the
loop does not yet guard against a model call failing outright — a gap named
and closed in Unit 4, not yet at this point in the project.

**What this milestone taught:**
1. "The agent prints the right thing" and "the agent passed the right object
   to the next function" are different claims, and only the second is
   actually checkable — an `is` comparison against a spied input is proof;
   matching printed text is not.
2. Routing values through a session dict instead of local variables costs
   nothing in behavior and buys testability for free — the identity check
   above would have nothing to compare against otherwise.
3. An error message has to name a lever, not just report a result —
   "No results" tells a user nothing they didn't already know; naming price,
   size, and keywords gives them three real things to try.
4. Choosing the cheaper, deterministic option for a job a model could also do
   (query parsing) keeps one more source of run-to-run variation out of the
   system — which matters more once Unit 4 starts measuring consistency.

**Before moving on:**
- *"A full query runs through all three tools with values moving through the
  session."* — **True**, demonstrated above with a real object-identity
  check.
- *"An impossible query stops early with a useful message."* — **True**,
  demonstrated above.
- *"Commit."* — **True**: `5ec9944`.

**If stuck:** the milestone's fallback is to print the value the branch
checks, on the line right before the `if`. Not needed here — the branch
worked on the first implementation attempt.

---

### Milestone 6: Write It Up and Submit

**The idea.** Most of the README fills in incrementally as each milestone
finishes. This step closes the two sections that only make sense once
everything else is done — a plain-language summary of what the system does,
and an honest record of how AI was actually used — and submits the
repository.

**Where the answers went.** README's **What This Does** and **Planning
Loop** sections. Real commit: `c34e50c` (2026-10-05, *"Added to sections
'Planning Loop'... and 'Sample Run'..."*), the same day as Milestone 5's
commit.

**Step: fill in "What This Does."** Committed text (quoted from the real
README): *"A user types a plain-language thrifting request, like `vintage
graphic tee under $30`, into `python app.py ask`. FitFindr pulls a
description, a size, and a price ceiling out of that request, searches a
mock listings dataset for the best match, and — if it found one — asks a
language model to suggest an outfit pairing the item with pieces from the
user's own wardrobe and to write a short, caption-style 'fit card'..."*

**Step: fill in "Planning Loop."** All four required pieces are present and
name the real file and function, `agent.py::run_agent` — matching the
milestone's own explicit warning that *"the grader checks your code against
what you claim here."*

**Step: "How I Used AI" — two specific moments.** A real, honestly-documented
timeline gap is worth recording here rather than hiding: searching the git
history for text unique to this section (`git log -S`) shows it was **not
actually filled in alongside the rest of Milestone 6**. The two moments now
in the README (the `.env` API-key diagnosis and the `generate.py` type-fix,
both from Milestone 4) weren't written into the README until commit
`7806852`, *"Wire search_listings through MCP in agent.py and mcp_server.py,
document the move and AI usage in README"* — which is **Unit 4's own
Milestone 1 commit**, days after the rest of Milestone 6 was finished. Named
here because this document's job is accuracy, not a tidy narrative.

**Step: the final checklist** (criteria.md present, all five README sections
real, four-plus commits, push and submit the URL):
- `criteria.md` present with five numbered criteria: **True**.
- README has real content in all five Unit 3 sections: **True as of the
  current state of the repository** — though, per the gap above, not yet
  true as of this milestone's own commit date.
- Four or more new commits: **True** — `a1f8f7e`, `55ee4f7`, `739c422`,
  `4220e00`, `5ec9944`, and `c34e50c` all postdate the initial commit, well
  past four.
- Repository URL submitted: **not verifiable from the repository's own
  contents** — submitting a URL to a learning platform happens outside this
  repo, so it can't be confirmed true or false here. Treated as done, since
  Unit 4 work proceeded against this same repository afterward, consistent
  with the course rule to submit the same URL both units.

**What this means for the next milestone.** Unit 4 opens by testing the
exact five criteria finalized in Milestone 3, against the exact loop
finalized in Milestone 5 — the agent's shape doesn't change going into Unit
4; new files (an MCP server, a scenarios file, a trace) get added around it
instead.

**Terms from this milestone:** none new — this step was documentation, not
new mechanics.

**Milestone summary.** True now: the Unit 3 submission is complete in its
current state — all five sections have real content, five criteria exist,
more than four commits exist. The one documented irregularity: "How I Used
AI" wasn't actually populated until early Unit 4 work, not during this
milestone's own commit.

**What this milestone taught:**
1. A milestone's checklist can end up satisfied *eventually* without having
   been satisfied *at the time its commit claims* — git history is the one
   record that can't be argued with about when something actually happened.
2. "Two specific moments" is a higher bar than "two sentences saying AI was
   used" — it took concrete before/after detail (a wrong key prefix, an
   exact type-checker error) to actually clear it.
3. Submitting the same repository across two units only works if nothing
   before it is deleted or restarted — Unit 4's entire test suite depends on
   `agent.py::run_agent` and `criteria.md` existing exactly as Unit 3 left
   them.

**Before moving on:**
- *"Your repository URL is submitted."* — not verifiable from this repo's
  contents; treated as done based on the Unit 4 work that exists in this same
  repository afterward.
- *"Write it down... you're submitting the same one [next unit]."* —
  satisfied in spirit: this is in fact the repository that continued into
  Unit 4, per its own commit history.

---

## 8. Features

The required feature set, as specified in the brief, with a note on how each
one currently stands in this repository.

### Three required tools (named inputs and outputs)

| Tool | Signature | What it does |
|---|---|---|
| `search_listings` | `(description, size, max_price)` | Searches the listings file and returns matches. |
| `suggest_outfit` | `(new_item, wardrobe)` | Takes an item and a wardrobe, returns outfit ideas. |
| `create_fit_card` | `(outfit, new_item)` | Writes a short caption someone would actually post. |

More tools are allowed beyond these three; none have been added so far.

### A planning loop with at least one required branch

When `search_listings` comes back empty, the loop must stop and say so — it
must not call the next tool (`suggest_outfit`) with nothing to work from.

### Session state across tool calls

The item `search_listings` finds has to flow into `suggest_outfit` on its own.
The user is never asked to type it a second time.

### A way to run it

A command-line script or a small web page — it has to work without someone
standing next to it operating it by hand.

*Current state:* `app.py` is the command-line interface (`python app.py ask
'...'`), and it's the one used throughout this project so far. A web interface
(`serve.py`, behind an HTTP server) also exists in this repository but belongs
to a later unit (Unit 9) and is out of scope for Units 3–4.

### Five acceptance criteria

Two are given; three are written specifically for this project, each with its
own stated target and a reason for that target (not a round number picked
because it "seemed reasonable"). The full list lives in `criteria.md` and is
summarized with real results in the Milestones section as those milestones are
added.

---

## 9. Setup

Full, general setup instructions — Python, Git, an AI coding assistant, the API
key, and the course-wide conventions behind all of it — live in the companion
file **[`ENVIRONMENT_SETUP.md`](ENVIRONMENT_SETUP.md)**, which also documents
where this specific repository's reality (its actual packages, its actual
`test.py` output, an actual setup gap found in this working copy) differs from
the general course text. This section is the condensed, project-specific
version.

### Install the dependencies and add your key

**macOS / Linux**
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
cp .env.example .env
```

**Windows (PowerShell)**
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
copy .env.example .env
```

Same key as earlier units — nothing new to sign up for. Paste it into `.env`,
which is already listed in `.gitignore`. It never goes in code, in the README,
or in a message to anyone.

One new package comes with this install: the **MCP SDK** (`mcp` in
`requirements.txt`), needed for Unit 4's tool-server rewire but not used until
then. It ships from the start of Unit 3 so there's nothing to install
mid-project.

Activate the virtual environment every time a new terminal is opened — look for
`(.venv)` at the start of the prompt.

### Confirm it works

```bash
python test.py
```

`RUNNING.md` in this repo is the reference for everything else — every
command, which command belongs to which milestone, what each file does, and
what to do when something breaks. It is read, not edited.

### The one quoting rule that matters everywhere

```bash
python app.py ask 'vintage graphic tee under $30'
```

**Always single quotes**, on every operating system. In PowerShell, double
quotes turn `$30` into a variable reference that expands to nothing — the
query silently becomes `vintage graphic tee under `, results come back with no
price ceiling applied, and nothing warns that it happened. That's why every
example command in this document and in `ENVIRONMENT_SETUP.md` uses single
quotes.

### About the model calls

Two of this project's three tools (`suggest_outfit`, `create_fit_card`) call a
hosted language model; `search_listings` does not. One full run of the agent
can therefore send several requests, and a few runs back to back while
debugging can cross the service's per-minute limit. The starter paces this
automatically — a message like `[rate limit] ... Waiting 35s. This is normal`
means it's doing its job, not that something is broken. It also reuses answers
to prompts already sent while building, so re-running the same query during
debugging costs nothing — Unit 4's test runs turn that caching off on purpose,
because a test needs real variation between tries, not one cached answer
repeated five times.
