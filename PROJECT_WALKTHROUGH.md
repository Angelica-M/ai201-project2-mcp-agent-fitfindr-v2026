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
> **A note on scope:** this file was created with Unit 3's brief, and Unit 4's
> brief was added once it was supplied. Both units' overview sections are
> labeled by unit throughout, so it's always clear which unit a given piece of
> "what this project is" describes.

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

### Learning Objectives (Unit 3)

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

### Unit 4 — Testing an Agent

Unit 3 built the agent and committed five acceptance criteria before any
results existed. Unit 4 is where that commitment gets cashed in: the same
agent, the same five criteria, and two things done to it that it has never
had done before.

The first is **MCP** — the Model Context Protocol, an agreed shape for
describing a tool so that any agent can call it, not just the one that was
built around it. One of FitFindr's three tools moves behind this shape. That's
the difference between a function an agent happens to call, and a tool
anything can use.

The second is **deliberate breakage**. Each of the project's three failure
modes gets triggered on purpose, rather than waited for. An agent that only
works on the happy path is a demo; the happy path is the easy half.

Then the test itself: five criteria, five tries each — because two of the
three tools call a model, so the same input can produce different answers, and
one run proves nothing about a system built that way. Each criterion gets
called met or missed against the target committed in Unit 3, every miss gets
traced back to the tool and step that caused it, one thing gets fixed, and the
whole test runs again.

**Why a miss doesn't cost points, by design:** an agent that misses every
criterion, with a clear explanation of each miss, earns full marks here. What's
graded is whether a real standard was set and tested against honestly — not
whether the agent happens to pass. A criterion quietly loosened until it passes
leaves nothing to diagnose, and the diagnosis is where the actual engineering
judgment shows up.

**Why this matters (the brief's own framing, kept intact):** *"It worked when I
ran it"* is the weakest sentence in engineering, and the one most people
default to about AI systems. Repeated trials, a trace showing what the system
actually chose and why, and a diagnosis that names the failing step — that's
the difference between having run a system in production and having demoed it
once.

### Learning Objectives (Unit 4)

By the end of Unit 4, the goal is to be able to:

1. **Expose a tool through MCP** so a client other than the agent's own code
   can call it.
2. **Trigger specific failure modes deliberately**, and handle each one.
3. **Trace an agent's decisions step by step**, and explain each choice.
4. **Test a nondeterministic system** with repeated trials, not one run.
5. **Diagnose a failure** down to the specific tool and step that caused it.

---

## 2. Project Title, Problem, Goals, and Architecture

### Overview (Unit 3)

**The problem it solves:** thrifting (buying secondhand clothes) takes real
effort. A person searches several apps, tries to picture how a found item
would look with clothes they already own, and guesses whether the asking price
is reasonable. Most of that effort is *looking things up* and *comparing
them* — exactly the kind of repetitive decision-making an agent can take over.

**What it does:** someone describes what they want — *"a vintage graphic tee
under $30, size M"* — and FitFindr searches a listings dataset, works out what
outfit the item would complete using the person's own wardrobe, and writes a
short caption ("fit card") someone would actually post about the find.

### Goals (Unit 3)

By the end of Unit 3, the project demonstrates the ability to:

- Write a tool spec that states exactly what goes in and what comes out.
- Build a loop that picks its next step from what the previous step returned.
- Carry information from one tool call to the next without asking the user to
  repeat themselves.
- Write acceptance criteria for a system whose behavior changes from run to
  run (because part of it calls a language model).

### Overview (Unit 4)

Unit 3 wrote down what "working" would mean for FitFindr, before any results
existed. Unit 4 is the part where that's actually found out — against the
same repository, under the same URL, with no new feature added for its own
sake.

Two things happen to the agent in this unit that it has never had done to it:
one tool moves behind MCP (so something other than the agent's own code can
call it), and each of the three failure modes gets broken on purpose, to see
exactly what the agent says when something goes wrong rather than guessing.

### Goals (Unit 4)

By the end of Unit 4, the project demonstrates the ability to:

- Move a tool behind MCP so something other than the agent's own code can
  call it.
- Trigger a failure on purpose instead of waiting for one to happen by
  accident.
- Test a system with repeated tries, not one lucky run.
- Trace an agent's decisions and say why it chose each step.

### Architecture

One request, traced top to bottom through every file it touches, including
the two failure-handling stops that an earlier version of this diagram left
out. This is the real flow `python app.py ask '...' --trace` prints, with
the file and function responsible for each step named directly on the
diagram — checked line by line against the current `agent.py`, not a
simplified or remembered version of it.

```
+------------------------------+    +------------------------------+
| You, in a terminal           |    | --empty-wardrobe flag?       |
| $ python app.py ask '...'    |    | get_empty_wardrobe() or      |
+------------------------------+    | get_example_wardrobe()       |
                                    | -> data/wardrobe_schema.json |
                                    +------------------------------+
                |                   |
                v                   v
+----------------------------------------------------+
| app.py -- the command line                         |
| loads the wardrobe BEFORE calling the agent, then: |
| calls agent.py :: run_agent(query, wardrobe)       |
+----------------------------------------------------+
                           |
                           v
+---------------------------------------------------------------+
| agent.py :: run_agent()  --  THE PLANNING LOOP                |
|                                                               |
| session = { query, parsed, search_results, selected_item,     |
|             wardrobe, outfit_suggestion, fit_card, error }    |
|                                                               |
| trace.py :: step() prints one line per call below;            |
| trace.py :: check_iterations() guards against an endless loop |
+---------------------------------------------------------------+
                                |
                                v
STEP 1   _parse_query(query)  ->  session["parsed"]

         |
         v
STEP 2   search_listings(...)  --  the ONE tool moved onto MCP

         agent.py -> mcp_client.py -> (spawns, fresh, every call)
            mcp_server.py -> tools.py :: search_listings()
               -> utils/data_loader.py -> data/listings.json

         |
  +------+------+   <-- BRANCH 1 (the one Unit 3 requires)
  |             |
no results   results found
  |             |
  v             v

+------------------------------------+     STEP 3   suggest_outfit(item, wardrobe)
| STOP HERE.                         |                 -> tools.py -> generate.py -> Gemini
| session["error"] = "No listings    |                    (hosted model, via google-genai)
| matched..." -- names what to       |              if ModelUnavailable:  BRANCH 2a  -> STOP,
| change (price / size / keywords)   |              session["error"] = str(exc), step 4 never runs
|                                    |
| Steps 3 and 4 never run.           |     STEP 4   create_fit_card(outfit, item)
+------------------------------------+                 -> tools.py -> generate.py -> Gemini
                                                    if ModelUnavailable:  BRANCH 2b  -> STOP,
                                                    session["error"] = str(exc)

                                           Back to app.py: prints Found / Outfit / Fit card
                                           (or session["error"], if it stopped at any branch)
```

**Three branches, not one.** Unit 3 required exactly one — Branch 1, the
empty-search stop. Unit 4's deliberate-failure-handling work (Milestone 2)
added two more of the same shape, catching `ModelUnavailable` around each of
the two model-calling tools so a rejected API key produces a clean
`session["error"]` instead of an unhandled exception. All three follow the
identical pattern: set `session["error"]`, stop, let the later fields stay
`None` — which is exactly what makes them checkable by comparing a session
dict rather than by reading logs.

**Two data sources, loaded two different ways.** The wardrobe is read once,
by `app.py`, *before* `run_agent` is even called, and travels through the
session from the start. The listings file is read per search, inside the
MCP subprocess, via `search_listings` itself. They look similar (`data/*
.json`, both through `utils/data_loader.py`) but enter the system at
different points — conflating them was a real risk named back in Unit 3,
Milestone 1.

**`search_listings` runs in a genuinely separate, short-lived process.**
`mcp_client.py`'s own docstring says it plainly: "every call starts the
server, asks, and stops it again." It is not a persistent connection — each
of the 50 MCP round-trips made across Unit 4's two full test runs (see
Section 6) started a fresh `mcp_server.py` subprocess, asked its one
question, and shut it down.

**`suggest_outfit` and `create_fit_card` share one chokepoint on purpose:**
`generate.py`. It's the only place in the project that calls the hosted
model, reading `GEMINI_API_KEY` from `.env` and the pacing, caching, and
model-name settings from `config.py` — so request pacing, response caching,
and `ModelUnavailable` all only had to be written once, not twice.

**Not pictured, and why.** `scenarios.py` + `run_eval.py` call
`agent.py::run_agent` directly, bypassing `app.py` entirely — a third real
entry point into the loop, used for every Unit 4 test run, left off the main
diagram because it's a *testing* path rather than part of the request flow a
user triggers. `serve.py` (an HTTP alternative to `app.py`) exists in the
repository but belongs to Unit 9 and has never been run as part of this
project so far — see Section 7.

#### Sequence Diagram

Moved here from the README, where a sequence diagram covering this same
request flow previously lived as a standalone image
(`assets/FitFindr_webSeqDiagram.jpg`, generated with webSequenceDiagrams.com)
with accompanying prose. Checked against the current `agent.py`, `app.py`,
`tools.py`, and `generate.py` before relocating. Five inaccuracies found and
corrected below; the image itself is left in place in `assets/` as a
historical artifact but is no longer referenced anywhere active, since its
errors are baked into the pixels and can't be annotated:

1. **Wrong terminal-facing component.** The original named `agent.py` as the
   component that reads terminal input. It's `app.py` that parses CLI flags
   and calls `agent.py::run_agent` — `agent.py` has no terminal interaction
   at all.
2. **Wardrobe input mischaracterized.** The original showed "Enter query and
   wardrobe" as one freeform input step. The wardrobe is actually chosen by
   a CLI flag (`--empty-wardrobe` or its absence), loaded by `app.py` via
   `utils/data_loader.py` *before* `run_agent` is called — a separate path
   from the query, not part of the same input.
3. **`generate.py` and the hosted model were entirely absent.** The original
   collapsed "select first listing; suggest outfit; create fit card" into a
   single internal step on `agent.py`'s own lifeline, with no external call
   shown. In the real system, both `suggest_outfit` and `create_fit_card`
   call `generate.py`, which calls the hosted Gemini model via
   `google-genai` — the slowest, highest-latency, only-failable-by-network
   part of the whole request. Omitting it is not a simplification; it hides
   the one dependency most likely to need pacing, caching, or a failure
   handler, which is exactly what `generate.py` provides.
4. **Only one of three branches was shown.** The original documented the
   empty-search guardrail correctly, but it was drawn before Unit 4,
   Milestone 2 added two more: a `try/except ModelUnavailable` around each
   of `suggest_outfit` and `create_fit_card`. All three follow the same
   shape (set `session["error"]`, stop, skip remaining steps).
5. **Naming.** "FitFinder" and `fitfitr` are both typos of the project's
   actual name, FitFindr.

**The corrected flow**, participant by participant:

*Participants:* Terminal (end user) · `app.py` (command line) ·
`agent.py::run_agent()` (the planning loop) · `tools.py` (the three tool
implementations) · `mcp_client.py` / `mcp_server.py` (MCP transport,
`search_listings` only) · `generate.py` → Gemini, via `google-genai`
(`suggest_outfit` and `create_fit_card` only).

**Phase 1 — ingestion and query parsing**
1. User runs `python app.py ask '<query>' [--empty-wardrobe] [--trace]`.
2. `app.py` loads the wardrobe (`get_example_wardrobe()` or
   `get_empty_wardrobe()`, from `data/wardrobe_schema.json`) based on the
   flag, before calling the agent.
3. `app.py` calls `agent.py::run_agent(query, wardrobe)`.
4. `run_agent()` calls `_parse_query(query)` → `session["parsed"] =
   {description, size, max_price}`.

**Phase 2 — MCP tool invocation (`search_listings` only)**
5. `run_agent()` calls `call_tool("search_listings", arguments)` via
   `mcp_client.py`.
6. `mcp_client.py` starts `mcp_server.py` as a fresh subprocess over stdio
   for this one call — not a persistent connection; it starts and stops on
   every call.
7. `mcp_server.py` calls `tools.py::search_listings()`, which reads
   `data/listings.json` via `utils/data_loader.py`.
8. The server returns matching listing dictionaries; `mcp_client.py`
   unwraps the response into a native Python list.

**Phase 3 — branch 1: empty-results guardrail**
9. If `session["search_results"]` is empty: `session["error"]` is set to a
   message naming what to change, and `run_agent()` returns immediately.
   `suggest_outfit` and `create_fit_card` are never called.

**Phase 4 — outfit and caption generation (the part the original diagram
omitted)**
10. If results were found: `session["selected_item"] =
    session["search_results"][0]`.
11. `run_agent()` calls `tools.py::suggest_outfit(selected_item, wardrobe)`
    directly — not through MCP. `suggest_outfit` calls
    `generate.py::generate()`, which reads `GEMINI_API_KEY` from `.env`,
    applies pacing and caching from `config.py`, and calls Gemini via
    `google-genai`.
    - **Branch 2a:** if `generate()` raises `ModelUnavailable`,
      `run_agent()` catches it, sets `session["error"]`, and returns.
      `create_fit_card` is never called.
12. If `suggest_outfit` succeeds: `run_agent()` calls
    `tools.py::create_fit_card(outfit, selected_item)`, which also calls
    `generate.py::generate()` → Gemini.
    - **Branch 2b:** the same `ModelUnavailable` handling, mirrored.

**Phase 5 — response rendering**
13. `run_agent()` returns the finished session to `app.py`, which prints
    `Found:` / `Outfit:` / `Fit card:`, or `session["error"]` if the loop
    stopped at any of the three branches.

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
| **Listing** | One item for sale in the project's dataset — not a live marketplace, a fixed JSON file standing in for one. | A single entry in `data/listings.json`, with fields like `title`, `price`, `size`, `platform`. Introduced in Unit 3, Milestone 1. |
| **Wardrobe item** | One piece of clothing a user already owns, in the shape `suggest_outfit` expects. | An entry in `data/wardrobe_schema.json`, or an empty `{"items": []}` for a new user. Introduced in Unit 3, Milestone 1. |
| **Empty case** | What a tool returns when it has nothing to give — decided and written down *before* the tool is built, not discovered as a crash later. | `search_listings` returns `[]`, never `None`, on no match. Introduced in Unit 3, Milestone 2. |
| **Acceptance criterion** | A stated, numeric definition of "working" — a target plus a reason — written before any test results exist, so it can't be quietly adjusted after the fact. | The five entries in `criteria.md`. Introduced in Unit 3, Milestone 3. |
| **Stub** | A placeholder function that runs and returns something, but doesn't yet do the real work — used to prove the shape of a tool before its logic exists. | How all three tools in `tools.py` started, before Unit 3 Milestone 4. Introduced in Unit 3, Milestone 4. |
| **Cache** (response cache) | Reusing an answer already received for an identical prompt, instead of calling the model again — free while building, turned off for real tests. | `CACHE_ENABLED` in `config.py`. Introduced in Unit 3, Milestone 4. |
| **Temperature** | How much a model's wording varies between calls with the same prompt. `0.0` repeats itself; higher values vary. | `TEMPERATURE` in `config.py`, set to `0.9`. Introduced in Unit 3, Milestone 4. |
| **Stop condition** | A hard limit that ends a loop even if its own branches never do — a safety net, not the main logic. | `config.MAX_ITERATIONS`, checked by `trace.check_iterations()` in `run_agent`. Introduced in Unit 3, Milestone 5. |
| **MCP** (Model Context Protocol) | An agreed shape for describing a tool, so any agent can call it — not just the one built around it. | `search_listings` is exposed this way starting in Unit 4, Milestone 1. |
| **Server** (MCP server) | The side of MCP that offers a tool for others to call. | `mcp_server.py` — registers `search_listings` as a callable tool. Introduced in Unit 4, Milestone 1. |
| **Client** (MCP client) | The side of MCP that calls a tool offered by a server. | `agent.py::run_agent`, via `mcp_client.py::call_tool`. Introduced in Unit 4, Milestone 1. |
| **Trace** | A printed record of what the agent did, step by step, in order — what went in and what came back, for every tool call. | `trace.py::step()`, called once per step inside `run_agent`. Introduced in Unit 4, Milestone 2. |
| **Failure mode** | One specific way a tool or the agent can go wrong — named in advance, so it can be triggered on purpose rather than discovered by accident. | FitFindr has three: empty search results, an empty wardrobe, and the model being unreachable. Introduced in Unit 4, Milestone 2. |
| **Try** | One full run of the agent on one scenario. A criterion names a target out of five, so five tries is what it takes to actually measure one. | One row of a Run Log table in README.md. Introduced in Unit 4, Milestone 3. |

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
the Unit 4, Milestone 2 entry in Section 6):
```
[2] search_listings (via MCP)
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    branch: results found, continuing to suggest_outfit

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Fit card: Literal peak 2000s energy right here with the prettiest pink and purple butterfly graphic. Snagged this mint-condition baby tee for $18.00 and I'm about to live in it with baggy denim and chunky sneakers all season. Go check my Depop before I change my mind and keep it.

2 model calls this session, 763 prompt + 361 output tokens
```

### Known Failure Mode: Invalid API Key

An invalid or misconfigured `GEMINI_API_KEY` is the most likely thing to go
wrong when `suggest_outfit` or `create_fit_card` runs — both call the hosted
model, and a bad key surfaces there, not at startup. The full real diagnosis
(what the error looked like, what was actually wrong with `.env`, and the fix)
is recorded under **Unit 3, Milestone 4** below, since that's the step where
it actually happened — not repeated here.

---

## 5. Findings and Lessons Learned

Eight findings from both units, each stated with the evidence behind it.
Milestone-level detail for all of these is in Section 6.

**1. A specification is only as good as the data it was checked against.**
`search_listings`'s size field contains values like `"US 9"` and `"XL"`. A
naive substring match (`"s" in "us 9".lower()` → `True`) matches size query
`"S"` against listing size `"US 9"`. Fix: token-based matching — a query
size matches a listing only if the listing's size, split into tokens, includes
that exact token; `"S"` then matches `"S/M"` but not `"US 9"`. Caught at spec
time (Unit 3, Milestone 2), implemented correctly on the first attempt (Unit
3, Milestone 4). The same principle holds in reverse: a test scenario built
around one easy, generously-matching query doesn't exercise anything (Unit
4, Milestones 3–4) — a spec or a test is only as rigorous as the data or
scenario it's run against.

**2. Deterministic code and model-dependent output require different target
rigor.** Every 5-of-5 target in `criteria.md` sits behind a plain `if`
statement (e.g., `if not results:`); every 4-of-5 target sits behind a call
to a language model. Unit 4's first test run scored 5/5 on all five
criteria (Unit 3, Milestone 3) — on its face, uniform success. In fact it
was three architecturally-guaranteed passes and two that happened not to
fail. Distinguishing the two target types at the time criteria were written
(Unit 3, Milestone 3) is what made that distinction possible to draw later.

**3. An exception caught at the top level is not equivalent to an exception
handled at the point of failure.** Before a handler existed, a corrupted
`GEMINI_API_KEY` raised an unhandled `ModelUnavailable` inside `run_agent()`;
`app.py`'s generic `except Exception` prevented a stack trace from reaching
the CLI user, but `run_agent()` itself still crashed, and a caller using
`run_agent()` directly would have received the raw exception. After adding
`try/except ModelUnavailable` around both model-calling tool calls (Unit 4,
Milestone 2), the same corrupted key produces:
```
session["error"] = "The model rejected your API key. Check GEMINI_API_KEY
in your .env file, or create a fresh key at aistudio.google.com."
session["outfit_suggestion"] = None
session["fit_card"] = None
```
— a normal session dict, not an exception. The key was restored immediately
after this test.

**4. A criterion's wording and its numeric target are independent axes; fixing
one doesn't fix the other.** Criterion 4 originally specified a 4-of-5 target
with ambiguous wording about what counts as "mentioning the price." The
first revision fixed the wording but left the target at 4-of-5 — a number
the ambiguous case (`"thirty bucks"`) satisfied regardless. Re-grading the
same, already-collected data against a tightened 5-of-5 target produced a
provable result:

| Run | Tries passing | Verdict vs. 5-of-5 |
|---|---|---|
| Before (`create_fit_card` prompt unconstrained) | 4 of 5 | MISSED (4/5) |
| After (price format specified in the prompt) | 5 of 5 | MET (5/5) |

(Unit 4, Milestones 4–5.)

**5. "Nothing missed" and "nothing left to do" are not equivalent claims.**
This distinction matters because conflating them is the specific failure
mode repeated testing is meant to catch: a clean result can mean the system
works, or it can mean the test wasn't rigorous enough to find the failure.
Every criterion read MET after Unit 4's first test run (Milestone 3) — a
result that could have ended the diagnosis. It didn't: Milestone 4's revisit
treated the clean result as a question, not a conclusion, and produced two
outcomes of different kinds from that single question — a reproducible
diagnosis (criterion 4's price-format ambiguity, finding 4 above) and a
negative result reported rather than omitted (criterion 1's assumed failure
mode — empty results on an awkwardly-phrased but valid query — proved hard
to reproduce on this dataset, because the keyword-overlap scorer counts
common stop words too).

**6. Commit history is a primary source, not secondary documentation.** Three
facts about this project are visible only in `git log`, not in the README:
Unit 3, Milestone 1 produced no commit, because reading data doesn't modify
files; the "How I Used AI" section required by Unit 3, Milestone 6 was not
populated until Unit 4's first commit; and Unit 4's Milestone 2 work (trace
wiring) was committed before Unit 4's Milestone 1 work (the MCP move),
reversing the brief's stated order. All three are recorded as found, in
Section 6, rather than reconciled to read as sequential.

**7. An object-identity check and a value-equality check test different
claims.** The milestone instructions for criterion 3 state why this
distinction matters: state failure doesn't look like state failure, it looks
like a tool problem — comparing printed item titles would not have caught a
case where the correct-looking item was actually a different object. The
check used instead:
```
Try 1: session.selected_item.id='lst_004'  suggest_outfit received='lst_004'  create_fit_card received='lst_004'  -> PASS
```
5 of 5, verified by wrapping a spy around `suggest_outfit` and
`create_fit_card` and comparing the `id` each received against
`session["selected_item"]["id"]`. (Unit 3, Milestone 5; Unit 4, Milestone 3.)

**8. A trace that shortens under a failing condition demonstrates that a
branch executes; a trace of constant length under both conditions would
demonstrate that it doesn't.**
```
[1] parse_query ...
[2] search_listings (via MCP)
      out: [] (empty)
      ->   branch: empty, stopping before suggest_outfit
```
Two steps print on an impossible query, versus four on a matching query —
this is the required branch (Section 7's feature list) executing, as
distinct from code that only claims to branch. (Unit 4, Milestone 2.)

---

## 6. Milestones

Documented retroactively from this repository's real history — git commits,
`README.md`, `criteria.md` — rather than redone from scratch, per the decision
made for this project. Every claim below about what changed, when, and in
which commit was checked against the actual repository, not recalled from
memory. Where the real history has a gap or an irregularity, it's named rather
than smoothed over.

### Unit 3, Milestone 1: Read the Data and Run the Starter

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

### Unit 3, Milestone 2: Spec the Three Tools

**The idea.** Writing down exactly what each tool takes and returns — before
writing the tool itself — forces a decision a vague return value would
otherwise dodge. A loop can only branch on something definite; "returns a
list" gives it nothing to check.

**Where the answers went.** README.md's **Tool Inventory** section (the
four-line spec per tool, plus the Agent Loop Branching Rule). Real commit:
`a1f8f7e` (2026-09-30), *"README.md Tool Inventory (draft section; needs
cleanup)"* — README +81/−15 lines, `tools.py` +1/−1 (a minor stub comment, not
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

### Unit 3, Milestone 3: Write the Acceptance Criteria

**The idea.** An acceptance criterion names a target specific enough that
someone else could check it — a number or a plainly observable outcome, never
an opinion like "the agent handles errors." Writing these *before* any test
exists is deliberately uncomfortable: it's the only way a later "basically
works" can't quietly redefine what "working" meant.

**Where the answers went.** `criteria.md` — all five criteria and their
reasons. Real commit: `55ee4f7` (2026-10-02), *"citeria.md Complete"* (the
project's own commit message, typo included — left exactly as it is here,
since this document's job is accuracy, not tidying up history).
`criteria.md` +15/−26 lines; README +12/−71 (the Tool Inventory draft from
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

### Unit 3, Milestone 4: Build the Three Tools

**The idea.** Build and test each tool alone, before any loop joins them.
Three untested tools wired into a loop isn't three problems — it's one
problem wearing six disguises, because a failure anywhere could be any of
four different layers (a tool, the loop, the session, or the model).

**Where the answers went.** `tools.py` (all three implementations);
`config.py` and `generate.py` (two small fixes found while testing); README's
**Sample Run** section (the three per-tool terminal tests). Two real commits:
`739c422` (2026-10-02, *"Defined 3 funcs in tools.py..."*, `tools.py`
+103/−7 lines) for the implementations, and `4220e00` (same day, *"Robust
fixes to config.py and generate.py, ran tests on the tools.py's 3 ai agent
tools"*, README +20, `config.py` +1/−1, `generate.py` +1/−1) for the fixes
found while actually running the tests this milestone requires. Two commits
for the whole milestone, not one per tool — unlike Milestone 2's single
commit, but the same underlying pattern: real commits group by what was
worked on in one sitting, not by the milestone's own step boundaries.

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

### Unit 3, Milestone 5: Wire the Loop and the State

**The idea.** Connect the three tools with a loop that reads the result of
the last call before deciding the next one. Without that, calling all three
tools regardless of outcome is a list of function calls wearing a loop's
clothing — not an agent.

**Where the answers went.** `agent.py::run_agent` (and its helper
`agent.py::_parse_query`); README's **Planning Loop** section. Real commit:
`5ec9944` (2026-10-05, *"Defined run_agent() in agent.py, and README
additions"*) — `agent.py` +84/−2 lines, README +313/−28. One commit for the
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

### Unit 3, Milestone 6: Write It Up and Submit

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

### Unit 4, Milestone 1: Move One Tool onto MCP

**The idea.** Right now every tool only exists inside this one program —
nothing else can reach them. MCP (Model Context Protocol) is an agreed shape
to wrap a tool in, so anything speaking the same protocol can call it: this
agent today, a different agent tomorrow, someone else's app after that. Only
one of the three tools moves — the point is to see the seam, not to redo all
the work.

**Where the answers went.** `mcp_server.py` (the tool registration);
`agent.py::run_agent` (the direct call swapped for an MCP call); README's
"On the MCP move" subsection. Real commit: `7806852` (2026-10-05, "Wire
search_listings through MCP in agent.py and mcp_server.py, document the move
and AI usage in README"). One commit for the whole milestone.

**A real chronology note, named rather than smoothed over.** This commit is
dated *after* part of Unit 4 Milestone 2's work (`21583c1`, same day,
earlier — the trace wiring). In practice, half of Milestone 2 happened
before Milestone 1, even though the canonical order is MCP first. This
document never reorders entries, so Milestone 1 still appears first below —
but the real sequence is reported here rather than implied to match.

**Step: pick the tool.** `search_listings` was chosen — it doesn't call a
model, so nothing about pacing or caching complicates learning the shape of
the move, matching the milestone's own stated reasoning.

**Step: register the tool in `mcp_server.py`.**
- What changed: uncommented the `@mcp.tool()` block and wrote the tool's
  description from scratch — not copied from `tools.py`'s docstring —
  explicitly naming units (`max_price` in whole or fractional US dollars),
  the non-substring size-matching behavior, and the empty-list return.
- Decision made:

  | Option | Why not / chosen |
  |---|---|
  | Copy the existing docstring into the MCP description | Written for a reader of the code; the MCP description is read by a caller who never sees `tools.py` — a different audience needs different words |
  | Write a fresh description naming units, behavior, and the empty case explicitly | **Chosen** |

- How checked: `python mcp_client.py` — real output confirmed the server
  offers exactly `search_listings`, with `description: string`, `size:
  string (optional)`, `max_price: number (optional)`.

**Step: point the agent at the server as a client.**
- What changed: removed the direct `from tools import search_listings`
  import; swapped the call inside `run_agent()` for `call_tool
  ("search_listings", {...})` from `mcp_client`.
- Verified **before** switching, not after: the MCP response was compared
  against the direct call on the same input — `via MCP, count: 6`, `direct
  call, count: 6`, `identical content: True`. Only after that comparison
  passed was the direct call actually replaced.
- Confirmed afterward: both the happy path and the empty-search path still
  ran correctly end to end through `python app.py ask ... --trace`, with
  `[2] search_listings (via MCP)` in place of `[2] search_listings` in the
  trace — the only visible difference.

**What this means for the next milestone.** The `(via MCP)` label in the
trace is what makes Milestone 2's trace entries literally show the seam
described here, rather than just claiming it exists.

**Terms from this milestone:** MCP, Server, Client (see Section 3).

**Milestone summary.** True now: one tool (`search_listings`) runs over MCP,
verified identical to its direct-call output. Still a placeholder, by the
milestone's own design: `suggest_outfit` and `create_fit_card` stay direct
calls.

**What this milestone taught:**
1. Comparing old and new behavior *before* switching over — not after — is
   what actually earns the claim "nothing behaved differently"; checking
   after the fact can't rule out a difference nobody looked for.
2. A description written for a caller that can't see the code is a different
   writing task than a docstring, even for the same function.
3. A project's real chronology rarely matches its written milestone order,
   and that's fine to report plainly rather than pretend otherwise.

**Before moving on:**
- *"One tool runs through MCP and a normal query still works."* — **True**.
- *"Commit."* — **True**: `7806852`.

**If stuck:** the milestone's fallback — keep the direct call, document
exactly where it broke. Not needed; the rewire worked on the first attempt.

---

### Unit 4, Milestone 2: Break It on Purpose, Then Trace It

**The idea.** Trigger each failure deliberately rather than waiting to meet
one by accident in a graded run. A trace is the difference between
describing what an agent does and actually showing it, and Milestone 4's
diagnosis depends on having one.

**Where the answers went.** `agent.py::run_agent` (the `trace.step()` calls,
and a `try/except ModelUnavailable` handler added around the two
model-calling tools); README's "Loop Trace" and "Failure Handling"
subsections. Two real commits: `21583c1` (2026-10-05, trace wiring for the
happy path and empty-search paths — done before the MCP move, per the
chronology note above) and `33b0263` (2026-10-10, "Add 'Failure Handling'
and 'Loop Trace — full run, after the fix'").

**Step: trigger all three failures, write down what the agent said.**

| Failure mode | How triggered | What the agent said (real) |
|---|---|---|
| Empty search | `python app.py ask 'designer ballgown size XXS under $5'` | *"No listings matched. Try a higher price ceiling, a different size, or broader keywords in the description."* — stopped before `suggest_outfit` |
| Empty wardrobe | `python app.py ask '...' --empty-wardrobe` | Non-empty general styling advice (e.g. *"Since this is the very first piece in their wardrobe..."*) — not a crash, not an empty string |
| Model unavailable | One character of `GEMINI_API_KEY` flipped, then a fresh query run | First attempt: an **unhandled** `ModelUnavailable` exception, caught only by `app.py::main()`'s generic top-level handler — `run_agent()` itself had no handler yet |

Of the three, only "model unavailable" actually crashed; the other two were
already correctly handled by Milestone 5's branch (empty search, Unit 3) and
`suggest_outfit`'s own empty-wardrobe check (Unit 3, Milestone 4) — no
changes needed there.

**Step: add a handler for the one that crashed.**
- What changed: `run_agent()` now wraps both `suggest_outfit` and
  `create_fit_card` in `try/except ModelUnavailable`, setting
  `session["error"]` to the exception's own message and returning
  immediately — the same stop-the-session pattern already used for the
  empty-search branch.
- Decision made:

  | Option | Why not / chosen |
  |---|---|
  | Leave it to `app.py::main()`'s generic catch-all | Works for a CLI user, but a caller using `run_agent()` directly gets an unhandled exception and no usable session |
  | Add `try/except ModelUnavailable` inside `run_agent()` | **Chosen** — produces a normal session with `error` set, regardless of caller |

- How checked (after the fix): the key was corrupted again and a fresh query
  re-run — `session["error"]` was set to *"The model rejected your API key.
  Check GEMINI_API_KEY in your .env file, or create a fresh key at
  aistudio.google.com."*; `outfit_suggestion` and `fit_card` stayed `None`;
  the function returned a normal session instead of raising. The key was
  restored immediately after.

**Step: add a trace.**
- What changed: one `trace.step(name, inputs=..., returned=..., note=...)`
  call after each of the four steps (`parse_query`, `search_listings (via
  MCP)`, `suggest_outfit`, `create_fit_card`), with a branch note explaining
  why the loop continued or stopped.
- How checked: `python app.py ask 'vintage graphic tee under $30' --trace` —
  real output in README's "Loop Trace" section shows all four steps in
  order; a second run on the impossible query shows only two steps, visibly
  shorter, proving the branch actually ends the loop early rather than just
  claiming to.

**A real quirk found here, named rather than quietly fixed:** `trace.step()`
prints on every `ask` call, not only when `--trace` is passed, because
`trace.py::step()` calls `print()` unconditionally — `--trace` only controls
whether those lines are also *captured* for `get_trace()`. Carried forward
as a known, minor issue (see Milestone 6).

**What this means for the next milestone.** The trace format established
here is what Milestone 3's run log pastes directly as evidence, and the
`ModelUnavailable` handler closes exactly the gap that would otherwise crash
Milestone 3's repeated test runs.

**Terms from this milestone:** Trace, Failure mode (see Section 3).

**Milestone summary.** True now: all three failure modes were triggered
deliberately; two were already handled correctly and the one that wasn't now
is; a full trace prints and is pasted into the README. Still a known,
unfixed quirk: the unconditional `trace.step()` print.

**What this milestone taught:**
1. Triggering a failure mode on purpose shows exactly which of the four
   possible failure points (tool, branch, session, model output) is actually
   unguarded — here, specifically the two model-calling tool calls.
2. A failure that never shows a raw stack trace to a user isn't automatically
   "handled" — the generic top-level catch in `app.py` masked the real gap
   in `run_agent()` until it was looked for directly.
3. A trace is only as honest as what controls when it prints — the `--trace`
   flag's name implies it gates printing; it actually only gates capturing.

**Before moving on:**
- *"All three failures have been triggered and each produces a message
  you'd be willing to show a user."* — **True**.
- *"Your trace is in the README."* — **True**.
- *"Commit."* — **True**: `21583c1`, `33b0263`.

**If stuck:** the milestone's fallback — force a failure with a hardcoded
`return []`, test the handler, remove it. Not needed; all three failures
were triggered from genuinely real conditions.

---

### Unit 4, Milestone 3: Run the Test

**The idea.** One run tells you nothing about a system where two of three
tools call a model. A criterion named "4 of 5" or "5 of 5" can only actually
be measured by running it five times.

**Where the answers went.** `scenarios.py` (five scenarios, one per
criterion); `results/run_2026-10-10_1840_before.md` (the raw output);
README's "Run Log — Before" section. Real commits: `8d9e4c6` (2026-10-10,
scenarios.py filled in) and `816e983` (2026-10-10, "Complete Milestone 3:
Ran all tests... Result file generated... base app performance results in
table").

**Step: work out what each criterion needs, put it in `scenarios.py`.**
Three scenarios shipped filled in (criteria 1, 2, and one marked
`criterion: None` as a diagnostic). Rather than leaving the diagnostic
unused, it turned out to already test exactly what criterion 5 (empty
wardrobe) needs — a matching query with an empty wardrobe — so its
`criterion` key was pointed at 5 instead of writing a duplicate. Two new
scenarios were added for criteria 3 and 4:

| Criterion | Scenario | Query | Wardrobe | Why this shape |
|---|---|---|---|---|
| 1 | matching query completes *(shipped)* | vintage graphic tee under $30 | example | A query the data can match |
| 2 | impossible query stops early *(shipped)* | designer ballgown size XXS under $5 | example | The empty-search branch |
| 3 | selected item id persists *(new)* | 90s track jacket in size M | example | Any matching query works — this checks session identity, not the query text |
| 4 | fit card has required metadata *(new)* | silk slip dress in midi length under $40 | example | `run_eval.py` already runs each scenario 5 times, so one query gives 5 independent cards for the *same* item |
| 5 | empty wardrobe gets general advice *(relabeled, not new)* | denim jacket under $50 | empty | The shipped diagnostic already did exactly what criterion 5 needed |

**Step: run everything five times.** `python run_eval.py --label before` —
25 tries total, caching off. Real result: 0 crashes, 40 real model calls
(criterion 2's 5 tries cost 0 each, since the loop stops before any
model-calling tool runs).

**Step: fill in the run log.** The real first result for this project —
not the brief's illustrative example table:

| Criterion | Target | Verdict |
|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | MET (5/5) |
| 2. Impossible query stops before the second tool | 5 of 5 | MET (5/5) |
| 3. Selected item ID persists across tool calls | 5 of 5 | MET (5/5) |
| 4. Fit card contains required metadata and valid length | 4 of 5 | MET (5/5) |
| 5. Empty wardrobe triggers general styling advice | 5 of 5 | MET (5/5) |

Real output is pasted underneath each row in README's "Run Log — Before,"
naming the producing file and function per the milestone's requirement — for
criterion 3, that's a spy wrapped around `tools.py::suggest_outfit` and
`tools.py::create_fit_card`, run through `agent.py::run_agent`, showing the
literal id (`lst_004`) matching across all 5 tries.

The milestone's own caution — *if every try looks identical on a
model-involving criterion, check you're really in test mode* — was checked
directly: the five fit cards for criterion 4 were read individually and are
genuinely different sentences, confirming caching was really off.

**What this means for the next milestone.** A clean 5/5 on every criterion,
on the first real test run, is itself a finding Milestone 4 has to take
seriously rather than celebrate uncritically.

**Terms from this milestone:** Try (see Section 3).

**Milestone summary.** True now: five scenarios exist, one per criterion,
each run five times with real, non-identical output; a complete run log
exists in the README. Still worth saying plainly: this is one test pass, not
a guarantee of the same result on a different day.

**What this milestone taught:**
1. A scenario already in the starter file can already be doing the job a new
   criterion needs — relabeling it instead of duplicating it kept the
   scenario count honest.
2. `run_eval.py` running each scenario five times automatically means
   "the same item run more than once" doesn't require typing the query five
   times by hand.
3. A criterion that stops the loop early costs zero model calls per try —
   worth knowing when estimating what a five-scenario, five-try run actually
   costs (40 real calls here, not 50).

**Before moving on:**
- *"Your run log has five criteria, five tries each, with pasted output
  underneath."* — **True**.
- *"Commit."* — **True**: `8d9e4c6`, `816e983`.

---

### Unit 4, Milestone 4: Call Each Criterion and Diagnose Every Miss

**The idea.** "Met" is the comfortable reading, and there's always a
generous version of it available. The actual work is reading each number
plainly against its committed target, and for every real miss, naming which
of the four places it happened and why.

**Where the answers went.** README's "Verdicts and Diagnoses" section
(first pass); `criteria.md` and README's "The Improvement" / "What's Still
Broken" sections (a later, honest revisit). Real commit: `15ade1f`
(2026-10-10, "Milestone 4: Call each criterion and diagnose every miss").

**Step: write MET or MISSED against each target.** First-pass result: all
five criteria read **MET**, at the exact targets committed in Unit 3 (4 of 5
for criteria 1 and 4; 5 of 5 for 2, 3, and 5) — the actual pass count from
Milestone 3's real run, against the actual target number, no rounding.

**Step: for every miss, name the step and mechanism.** None existed in this
first pass, so there was nothing to diagnose under the tool/branch/session/
model-output rubric — stated directly rather than padded out.

**Step: look for a pattern; if nothing missed, say so and name what you'd
tighten.** The honest read: criteria 2, 3, and 5 are pinned to deterministic
code — a 5-of-5 result there is the expected floor, not evidence of a soft
target. Criteria 1 and 4 involve the model and came back 5/5 against a
4-of-5 target — on paper a clean win, but criterion 1's scenario reused one
easy query for all 5 tries rather than testing the harder phrasing the
target was written to anticipate, so the 5/5 proves the test was easy more
than it proves the agent is good. Criterion 4 turned up something concrete:
one of five fit cards read *"...for thirty bucks..."* instead of a numeral —
scored PASS at the time (it still communicates the price), but the
criterion's own wording never said whether a spelled-out amount counts.
Named as "the criterion I'd tighten," with the actual revision made in
`criteria.md`.

**A later revisit, after Milestone 6 (same milestone, additional real
work).** A later check against a grading rubric specifically asked whether
*every miss is diagnosed* — a file with zero misses on record, however
honestly reasoned, can't satisfy that literally. Rather than invent a
failure, criterion 1's actual named risk was tested for real: several
harder, synonym-heavy queries were run directly against `search_listings`
(e.g. `"motorcycle jacket"` targeting the leather bomber listing). None came
back empty — the dataset's own natural-language descriptions contain
ordinary stop words (`"the"`, `"with"`, `"and"`...) that the keyword-overlap
scorer counts too, so a true empty-result miss on an item that actually
exists turned out to be hard to reproduce on this specific dataset. That's a
real, reported finding, not a workaround.

Separately, criterion 4's target was tightened from 4-of-5 to 5-of-5 — not
just reworded — reasoned the same way criteria 2/3/5 already are: once
`create_fit_card`'s prompt (fixed in Milestone 5) pins the price format down
explicitly, the target should be as tight as the mechanism now is.
Re-grading the **already-collected** before-run data against that tightened
target (no new data invented) produced a genuine result:

| Run | Scenario | Tries passing | Verdict vs. tightened 5-of-5 |
|---|---|---|---|
| Before | fit card has required metadata | 4 of 5 (Try 3: "thirty bucks") | **MISSED (4/5)** |
| After (Milestone 5) | fit card has required metadata | 5 of 5 | **MET (5/5)** |

Diagnosed by step (the model's output inside `create_fit_card`) and
mechanism (the prompt at the time never constrained the price's format).
Real commit for this revisit: `64a4b85` (2026-10-10, "Tighten criterion 4's
target to 5/5, retroactively diagnose the real before-run miss...").

**What this means for the next milestone.** The retroactively-tightened
5-of-5 target on criterion 4 is exactly what Milestone 5's fix is measured
against, and it's a genuine before/after number, not a relabeled one.

**Terms from this milestone:** none new.

**Milestone summary.** True now: every criterion has a verdict read plainly
against its committed target, and there is one real, diagnosed miss on
record (criterion 4, before run, against the tightened target), reached by
tightening a target with evidence rather than inventing a failure. Still a
placeholder: criterion 1's "why this target" reasoning in `criteria.md`
hasn't been rewritten to reflect what the real stress test found.

**What this milestone taught:**
1. Reaching "met" honestly on the first pass doesn't mean the work is
   done — the harder question is whether the test that produced it was hard
   enough to prove anything.
2. A target can be reworded to fix an ambiguity without ever being
   tightened, and that's a real trap: 4-of-5 absorbed the ambiguous case
   either way, so "fixing the wording" alone left nothing at stake.
3. Retroactively re-grading already-collected data against a newly-justified
   target is honest; inventing new data to force a miss would not have
   been — the difference is whether the standard changed for a real reason.
4. A criterion's stated risk can be wrong about the actual failure mode even
   when the number still passes — criterion 1 assumed empty results would be
   the problem; the dataset's permissive scoring made a different problem
   (irrelevant top matches) the real one.

**Before moving on:**
- *"Every criterion has a verdict and a sentence on how you decided."* —
  **True**.
- *"Every miss names a step and a mechanism."* — **True**, after the later
  revisit — the first pass alone had none to diagnose, stated honestly
  rather than padded.
- *"Commit."* — **True**: `15ade1f`, then `64a4b85` for the revisit.

**If stuck:** the milestone's suggestion — argue the opposite verdict. Not
needed for the original five verdicts; the later revisit effectively did a
version of this by testing criterion 1's stated risk directly instead of
arguing about it.

---

### Unit 4, Milestone 5: Fix One Thing and Re-run

**The idea.** One change, measured properly, beats several changes nobody
can tell apart. The diagnosis from the previous milestone names what to fix,
not a menu of options.

**Where the answers went.** `tools.py::create_fit_card` (the one change);
`results/run_2026-10-10_1919_after.md`; README's "The Improvement" section.
Two real commits sharing the identical message *"Milestone 5: Fix one thing
and re-run"* — `61633df` (adds the raw after-run results file) and `2af1702`
(the actual `tools.py` prompt change and the README write-up). A real quirk
of this project's history, named rather than hidden: one milestone's work,
split across two commits with one message, not a duplicate.

**Step: pick the improvement.** Category chosen from the milestone's own
list: *"rewrite one prompt"* — the diagnosed failure was a model-calling
tool producing something unusable (a spelled-out price), not a missing field
shape or an unguarded branch. The one line changed:

| | Text |
|---|---|
| Before | *"Mention the item, its price, and the platform each exactly once..."* |
| After | *"Mention the item, its price as a numeral with a dollar sign exactly as given above (e.g. "$30" — never spelled out, like "thirty dollars" or "thirty bucks"), and the platform..."* |

Nothing else changed — not the loop, not the other two tools, not the
scenarios.

**Step: run the full test again.** `python run_eval.py --label after` —
same 5 scenarios, 5 tries each, caching off. Real result: 0 crashes, 40 real
model calls.

**Step: put both run logs side by side.** All 20 fit cards produced across
the after run (the targeted scenario plus three others that also call
`create_fit_card`) used a numeral price (`$18.00`, `$30`, `$42.00`,
`$45.00`) — zero spelled-out amounts, against one in the before run.
Re-graded against the tightened 5-of-5 target from Milestone 4's revisit:
**before = MISSED (4/5)**, **after = MET (5/5)** — a real, measured flip.

**Step: say whether it helped.** Yes, specifically: the thing the diagnosis
named (an ambiguous, spelled-out price) didn't recur once across 20 real
outputs after the fix, and the number now at risk (5-of-5) moved from a
genuine miss to a genuine pass. One-sentence version: *the fix worked, and
it's provable now against a target that was actually tight enough to fail.*

**What this means for the next milestone.** The fix is a prompt instruction,
not a code-level guarantee — the model could in principle still drift back
to a spelled-out price on some future call, which Milestone 6 names as a
residual risk rather than treats as closed.

**Terms from this milestone:** none new.

**Milestone summary.** True now: one change was made, the full test was
re-run, and both run logs show a genuine MISSED-to-MET flip. Still a
placeholder: the fix constrains the model's output through instruction only;
nothing in the code verifies the price format is actually a numeral.

**What this milestone taught:**
1. A fix aimed at the literal mechanism a diagnosis names is measurable in a
   way a vaguer fix ("make the fit cards better") never could be.
2. Two commits sharing one message is a normal, slightly messy real-project
   thing — recording it honestly costs nothing, and a cleaned-up retelling
   would have been a small lie.
3. "Did it help" deserves a precise answer — the improvement here only
   became provable once the target was tightened enough to have been
   failable in the first place.

**Before moving on:**
- *"Both run logs are in your README."* — **True**.
- *"You can say in one sentence whether the change helped."* — **True** (see
  above).
- *"Commit."* — **True**: `61633df`, `2af1702`.

**If stuck:** the milestone's fallback — keep what exists, put both run logs
in anyway, describe what broke. Not needed; the fix worked on the first
attempt, verified against real before/after data.

---

### Unit 4, Milestone 6: Say What's Still Broken

**The idea.** By the end of Milestone 5, every criterion's current verdict
reads MET — including criterion 4, whose real before-run miss (established
in Milestone 4's revisit) was fixed and re-verified. That's worth being
precise about: it means the test is clean *now*, not that nothing was ever
found broken, and it is not the same claim as "nothing left to do." This step
is the honest accounting of what's genuinely unresolved even once the
scoreboard is clean.

**Where the answers went.** README's "What's Still Broken," "On the MCP
move" (an added update), and "How I Used AI" (two more moments) sections.
Real commit: `c2602bb` (2026-10-10, "Add 'What's Still Broken'... Update MCP
move section with Milestone 6 results... Append AI Moments 3 and 4"); the
later Milestone 4 revisit (`64a4b85`) added a fifth AI moment and a further
update to this same section afterward.

**Step: name what's still broken**, with what would fix it and why it
wasn't done now:

| # | What's still broken | What would fix it | Why not done now |
|---|---|---|---|
| 1 | Criterion 1's target was never tested against the risk it names | Add harder, synonym-heavy scenarios to `scenarios.py` | Milestone 5 was one change; the diagnosis pointed at criterion 4's concrete evidence instead |
| 2 | Criterion 4's fix is a prompt instruction, not code-enforced | A regex check after `generate()` in `create_fit_card`; retry or flag on a non-numeral price | Closer to "add a branch" than "rewrite a prompt" — the cheaper, evidenced fix came first |
| 3 | `search_listings`'s MCP call has no failure handler, unlike the model-calling tools | A `try/except MCPError` around `call_tool(...)`, mirroring the `ModelUnavailable` pattern | Never observed failing — 50 MCP round-trips, zero failures — nothing to diagnose yet |
| 4 | `trace.step()` prints on every `ask` call, not only with `--trace` | Have `trace.py` track whether a trace is active before printing | Cosmetic; never the thing a diagnosis pointed at |

**Step: add a line on the MCP move.** The two full `run_eval.py` passes
called `search_listings` over MCP 25 times each — 50 subprocess round-trips
total, each one starting the server fresh — with zero MCP-related failures
across either run. More real exercise than the original single-call
comparison in Milestone 1, and it held up. This same exercise is what
surfaced gap #3 above: the handler that doesn't exist, not because anything
failed, but because nothing has tested that path yet.

**Step: update "How I Used AI."** Two further moments were added at this
point (the MCP rewire itself, and the Milestone 4 diagnosis that surfaced
the "thirty bucks" ambiguity). A fifth was added later, during the Milestone
4 revisit, documenting the troubleshooting session that tested criterion 1's
real risk empirically and tightened criterion 4's target with evidence.

**Step: submit the same URL as Unit 3.** Not verifiable from the
repository's own contents, same as Unit 3's equivalent step. Treated as
done, consistent with the course's rule that the same repository and URL
carry across both units.

**What this means going forward.** Nothing in this project's current state
is presented as finished when it isn't — the four named gaps above are the
actual, current state at the end of Unit 4, not a hypothetical "if there were
more time" list.

**Terms from this milestone:** none new.

**Milestone summary.** True now: every criterion has a verdict, the one real
miss has a diagnosis, one improvement was made and measured, and the
genuinely unresolved gaps are named with reasons. Still placeholders,
honestly: the four items in the table above.

**What this milestone taught:**
1. A clean scoreboard and a finished project are different claims — this
   milestone exists specifically to stop them from being conflated.
2. Something that's "never failed yet" is still a real gap, not a
   non-issue — the right question is whether it's been tested, not whether
   it's broken so far.
3. Documentation has its own git history, and that history can show a gap
   was named later than it feels like it should have been — worth recording
   plainly rather than editing the earlier claim to match.

**Before moving on:**
- *"For each criterion still missed, say what you'd do and why you
  stopped."* — addressed directly: no criterion's *current* verdict reads
  MISSED (the one real miss found, criterion 4's before run, was already
  fixed in Milestone 5), so the broader honest accounting above (four real
  residual gaps) stands in for a miss-by-miss list.
- *"Commit."* — **True**: `c2602bb`, plus `64a4b85` for the later additions.

---

### Other Repository Changes Outside This Walkthrough

The real commit history includes a few commits added directly to the
repository rather than through the process this document chronicles:
`10ea8cb` (an architecture/sequence-diagram section originally added to
`README.md`) and three image-placement commits (`221e9a7`, `257c5b5`,
`282f82d`). The sequence-diagram content from `10ea8cb` is no longer
"outside" this walkthrough as of a later pass — it was reviewed for
accuracy, corrected, and relocated into the Architecture section above (see
"Sequence Diagram"). The image-placement commits remain outside this
document's scope and aren't re-described here — named for completeness
rather than left silently out of the real commit list.

---

## 7. Features

The required feature set, as specified in each unit's brief, with a note on
how each one currently stands in this repository.

### Unit 3 Required Features

#### Three Required Tools

| Tool | Signature | What it does |
|---|---|---|
| `search_listings` | `(description, size, max_price)` | Searches the listings file and returns matches. |
| `suggest_outfit` | `(new_item, wardrobe)` | Takes an item and a wardrobe, returns outfit ideas. |
| `create_fit_card` | `(outfit, new_item)` | Writes a short caption someone would actually post. |

More tools are allowed beyond these three; none have been added.

#### Planning Loop with a Required Branch

When `search_listings` comes back empty, the loop must stop and say so — it
must not call the next tool (`suggest_outfit`) with nothing to work from.

#### Session State Across Tool Calls

The item `search_listings` finds has to flow into `suggest_outfit` on its own.
The user is never asked to type it a second time.

#### Execution Interface

A command-line script or a small web page — it has to work without someone
standing next to it operating it by hand.

*Current state:* `app.py` is the command-line interface (`python app.py ask
'...'`), and it's the one used throughout this project. A web interface
(`serve.py`, behind an HTTP server) also exists in this repository but belongs
to a later unit (Unit 9) and is out of scope for Units 3–4.

#### Five Acceptance Criteria

Two given, three written specifically for this project, each with its own
stated target and a reason tied to the mechanism behind it. The full list
lives in `criteria.md`; real results against all five are in Section 6's
Unit 4 milestones.

### Unit 4 Required Features

| Feature | What it requires | Current state |
|---|---|---|
| One tool moved onto MCP | Pick one of the three tools; the agent calls it through MCP from then on, the other two stay direct calls | **Done** — `search_listings`, verified identical to its pre-MCP output (Milestone 1) |
| Every failure mode triggered on purpose | Three named failure modes, each with a handler that tells the user what broke and what to try | **Done** for all three — empty search and empty wardrobe were already handled; a `ModelUnavailable` handler was added for the third (Milestone 2) |
| A trace of one full run | Every tool call in order, with what went in and what came back | **Done** — `trace.step()` calls in `agent.py::run_agent`, pasted into README's "Loop Trace" (Milestone 2) |
| Five tries per criterion | One run isn't a test; five tries is what it takes to score a criterion named "out of 5" | **Done** — `run_eval.py --label before` / `--label after`, 25 tries each (Milestone 3) |
| A verdict and a diagnosis for each criterion | Met or missed against the Unit 3 target; for every miss, which tool and which step | **Done** — one real diagnosed miss exists (criterion 4, before run), tightened and re-graded with evidence (Milestone 4) |
| One improvement, made and measured | Change one thing, run the test again, put both results side by side | **Done** — one prompt line changed in `create_fit_card`, before/after run logs show a genuine MISSED → MET flip (Milestone 5) |

---

## 8. Setup

Full, general setup instructions — Python, Git, an AI coding assistant, the API
key, and the course-wide conventions behind all of it — live in the companion
file **[`ENVIRONMENT_SETUP.md`](ENVIRONMENT_SETUP.md)**, which also documents
where this specific repository's reality (its actual packages, its actual
`test.py` output, an actual setup gap found in this working copy) differs from
the general course text. This section is the condensed, project-specific
version.

### Unit 3 Setup

#### Install Dependencies and Add Your API Key

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

#### Verify the Setup

```bash
python test.py
```

`RUNNING.md` in this repo is the reference for everything else — every
command, which command belongs to which milestone, what each file does, and
what to do when something breaks. It is read, not edited.

#### Required Quoting for CLI Queries

```bash
python app.py ask 'vintage graphic tee under $30'
```

**Always single quotes**, on every operating system. In PowerShell, double
quotes turn `$30` into a variable reference that expands to nothing — the
query silently becomes `vintage graphic tee under `, results come back with no
price ceiling applied, and nothing warns that it happened. That's why every
example command in this document and in `ENVIRONMENT_SETUP.md` uses single
quotes.

#### Rate Limits and Response Caching

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

### Unit 4 Setup

No new install — the MCP SDK was already in `requirements.txt` from the start
of Unit 3 (see above), so this unit opens with what's already there, not a
fresh `pip install`.

**Confirm the environment still runs:**
```bash
# macOS / Linux
source .venv/bin/activate
python test.py
```
```powershell
# Windows (PowerShell)
.venv\Scripts\Activate.ps1
python test.py
```
If `test.py` fails on something that passed in Unit 3, re-run
`pip install -r requirements.txt` inside the activated environment first.

**Confirm the agent still runs end to end:**
```bash
python app.py ask 'vintage graphic tee under $30'
```
Single quotes, same reason as above — this doesn't change for Unit 4.

**One `RUNNING.md` entry worth knowing before it shows up unannounced:**
`IncompleteFieldDefinitionWarning: Field 'lifespan'` is harmless noise from
the MCP library's own dependencies — not this project's code, and not an
error. `RUNNING.md`'s troubleshooting table confirms this exact line.

**One real change in behavior, not a bug:** the response cache that made
building free gets turned off for every Unit 4 test run
(`run_eval.py` sets this automatically). Five tries of the same question are
only a real test if the model actually answers five times — so a test run
sends more requests than a build run did, and the starter pauses and paces
itself more than it used to while building. That pause is expected; see
Section 4 above and the Unit 3, Milestone 4 entry in Section 6 for what a
rate-limit pause actually looks like when it happens.
