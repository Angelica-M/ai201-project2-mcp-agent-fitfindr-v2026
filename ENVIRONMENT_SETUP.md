# Environment Setup

> **What this file is.** Two things, stitched together: the course-wide setup guide
> that's handed to every student before Unit 1 (installing Python, Git, an AI
> assistant, and so on), and the setup steps specific to *this* repository —
> **FitFindr**, the Units 3–4 project pair. The course-wide part applies to every
> unit you'll ever open in this program; the FitFindr-specific part only applies
> here.
>
> Wherever the course-wide text doesn't match what this specific repo actually
> does or contains, there's a **"Reality check for this repo"** callout
> correcting it. Nothing below has been softened — if a check genuinely failed
> when this was verified, that's shown, not edited out.

---

## Part 1 — Course-wide setup (applies to every unit)

Do this before your first class. Set aside about 45 minutes, and don't leave it to
the morning of.

Every session in this course is hands-on from the first minute. There is no setup
block at the start of Unit 1 — the time is spent building. That only works if your
machine is ready when you arrive.

### The contract

You're ready when `test.py` passes. That's one script in your starter repo that
checks your Python version, your packages, your API key, and one real call to the
model. It's the same file in every unit, so you'll run it again whenever something
feels off.

```bash
python test.py
```

If it doesn't pass and you've made one honest attempt at the error, post the full
output in the help channel before class. Setup problems are normal, they're
fixable, and nobody has ever been judged for having one.

### What you'll need

| | Details |
|---|---|
| A laptop | Mac, Windows, or Linux, with 8 GB of RAM (16 GB recommended) and at least 20 GB of free disk — the requirement in the syllabus. Units 1–4 use little of that; units 5–8 train and run models on your own machine and use most of it. Check the disk number now. |
| Python 3.11, 3.12, or 3.13 | Not 3.14 — see below |
| A Google account | For the model API in units 1–4. No credit card, ever |
| A GitHub account | You'll fork a starter repo in Unit 1 |
| An AI coding assistant | Copilot or Claude Code, inside VS Code |

You do not need a GPU, a paid API plan, or a powerful machine for this pair. Two
of FitFindr's three tools run entirely on your own laptop and cost nothing — only
the two that write text (`suggest_outfit`, `create_fit_card`) call out to a
hosted model.

### 1. Install Python 3.11–3.13

Not 3.14. It shipped as the default on recent macOS versions and several of this
course's pinned packages don't support it yet. If `python3 --version` reports
3.14, install 3.13 alongside it — you don't have to remove anything.

Download from [python.org/downloads](https://python.org/downloads). On Windows,
check "Add Python to PATH" during install.

Verify:

```bash
python3 --version      # macOS / Linux
python --version       # Windows
```

You want 3.11.x, 3.12.x, or 3.13.x.

> **Reality check for this repo:** verified on this machine with **Python
> 3.13.7 on Windows** — inside the supported range.

### 2. Install VS Code

Download [Visual Studio Code](https://code.visualstudio.com), open it once, then
install the Python extension from the Extensions panel (the one published by
Microsoft).

### 3. Install Git and sign into GitHub

Download Git from [git-scm.com](https://git-scm.com), then verify:

```bash
git --version
```

Confirm you can sign into github.com. Create an account if you don't have one.

### 4. Set up an AI coding assistant

You'll use AI throughout this course, deliberately and out loud. Set up at least
one inline assistant before day 1: **Claude Code** (if you got a course
invitation) or **GitHub Copilot** (free through the GitHub Student Developer
Pack — approval can take a few days). Also confirm you can reach a conversational
AI (Claude, ChatGPT, or Gemini) for thinking things through, separately from the
one that writes code with you.

> **Reality check for this repo:** this is a real, working example of that setup
> in practice, not a hypothetical — most of FitFindr's unit 3–4 build (and this
> walkthrough itself) was done with an AI coding assistant, with every change
> explained and verified rather than taken on faith. The README's "How I Used
> AI" section has the specific record.

### 5. Get an API key

Your projects call a hosted model to generate text. It's free, and it needs a
Google account.

1. Go to [aistudio.google.com](https://aistudio.google.com) and sign in.
2. Create your own project when prompted, and get an API key inside it.
3. Copy the key somewhere safe — you can't view it again after you close the
   page.

Create your own project. Don't reuse someone else's, and don't share yours. The
free quota is counted per project, not per key.

Your key is a secret. It goes in a `.env` file, listed in `.gitignore` so it
never reaches GitHub. Never paste it into your code, your README, a screenshot,
or a message to anyone.

> **Reality check for this repo — the actual failure, not a hypothetical one:**
> this exact mistake happened during this project's build. The `.env` file ended
> up with a key under the misspelled variable name `GEMINI_API_KEY1` instead of
> `GEMINI_API_KEY` (so the real key was never read), *and* the key itself
> started with `AQ.Ab8RN6...` rather than the standard Google AI Studio
> `AIzaSy...` prefix — a sign the wrong kind of credential had been copied in.
> The fix: go back to aistudio.google.com, copy the genuine `AIzaSy...` key, and
> replace `.env` with a single, correctly-named `GEMINI_API_KEY=` line. See the
> Milestones section of `PROJECT_WALKTHROUGH.md` for the full diagnosis.

### 6. Set up the project

You'll do this once per starter repo. Fork and clone it, then from inside the
repo folder:

**macOS / Linux**
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**Windows (PowerShell)**
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```
If PowerShell refuses with a script-execution error, run this once and try again:
```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

**Windows (Command Prompt)**
```cmd
python -m venv .venv
.venv\Scripts\activate.bat
python -m pip install --upgrade pip
pip install -r requirements.txt
```

You'll see `(.venv)` at the start of your prompt when it's active. Activate it
every time you open a new terminal. Installing into your system Python instead
of the virtual environment is the single most common cause of "it worked
yesterday."

> **Reality check for this repo:** this is the recommended, correct path, and
> it's what you should do. But documenting this honestly means saying that **the
> working copy this walkthrough was built from does not actually have a
> `.venv`** — there's no `.venv` folder in the repo, and `test.py` confirms it:
> `sys.prefix` and `sys.base_prefix` are identical (`C:\Python313`), meaning
> every package was installed straight into the system Python, not an isolated
> environment. That's a real deviation from the documented practice above, not
> a recommendation — it's flagged here because this file's job is to reflect
> what's actually true of this repo, not what should be true of it. If you're
> setting this project up fresh, follow the venv steps above; don't copy this
> shortcut.

### 7. Add your key

Copy the example file and paste your key in:

```bash
cp .env.example .env        # Windows: copy .env.example .env
```

Then edit `.env` so it reads:

```
GEMINI_API_KEY=paste_your_actual_key_here
```

No quotes, no spaces around the `=`. The file is called `.env` — not
`.env.txt`, not `.env.example`.

> **Reality check for this repo:** confirmed — `.env.example` in this repo
> matches this format exactly, including the optional `AI201_CACHE=0` line for
> turning the response cache off (which `run_eval.py` does automatically for
> every unit 4 test run).

### 8. Run the check

```bash
python test.py
```

The generic course guide shows a sample "clean run" from the **Units 1–2**
starter, which checks things this repo doesn't have at all (an embedding model,
a vector database). That table is reproduced below for reference, but it is
**not what this repo's `test.py` actually checks or prints** — see the
"Reality check" callout immediately after it for the real, verified output.

<details>
<summary>Generic Units 1–2 example (not applicable to this repo — click to expand)</summary>

```
AI201 environment check
------------------------------------------------------------
[PASS] Python version
[PASS] Virtual environment
[PASS] Pinned packages
[PASS] Free disk space
[PASS] Memory
[PASS] Key hygiene
[PASS] API key
[ .. ] Embedding model — first run downloads ~80 MB, this is the slow part
[PASS] Embedding model
[PASS] Vector store
[PASS] Model call
------------------------------------------------------------
10 passed, 0 failed, 0 to look at, 0 skipped
```
</details>

> **Reality check for this repo — the real, verified output**, captured by
> actually running `python test.py` in this repo (not transcribed from
> documentation):
>
> ```
> AI201 environment check
> ------------------------------------------------------------
> [PASS] Python version
>          3.13.7 on Windows
> [FAIL] Virtual environment
>          Not active. Run the activate command for your OS, then try again.
>          Installing into your system Python is the most common cause of
>          'it worked yesterday'.
> [PASS] Pinned packages
>          all 5 import cleanly
> [PASS] Free disk space
>          287.7 GB
> [PASS] Memory
>          15.3 GB
> [PASS] Key hygiene
>          .env is ignored by git
> [PASS] API key
>          loaded, 53 characters
> [PASS] MCP
>          server and client both import
> [PASS] Project data
>          40 listings, 10 wardrobe items
> [PASS] Model call
>          gemini-3.5-flash-lite replied "Ready"
> ------------------------------------------------------------
> 9 passed, 1 failed, 0 to look at, 0 skipped
>
> Not ready yet. Fix the FAIL lines above, then run test.py again.
> ```
>
> This is a genuinely useful example of **reading the tool's output rather than
> panicking at a FAIL**: everything that matters for building and running
> FitFindr — the API key, the MCP packages, the project's own data files, and a
> real round-trip call to the model — passed. The one FAIL is exactly the venv
> gap named above, and `test.py` tells you precisely what it means and how to
> fix it, which is the whole point of running it.
>
> Notice this repo's checks differ from the generic example: there's no
> "Embedding model" or "Vector store" line (this project never loads one), and
> there are two checks the generic doc doesn't mention at all — **MCP** (do the
> Model Context Protocol server and client import correctly — Unit 4's one new
> package) and **Project data** (does `data/listings.json` actually load, and
> does it contain real listings and a real wardrobe). `test.py`'s own docstring
> explains why: "It is the same file in every unit's starter repo — the checks
> adapt to whatever that unit's requirements.txt pins."

`[SKIP]` lines are fine — they're packages a later unit adds. `[WARN]` is worth
reading. `[FAIL]` is what you fix.

### What's pinned, and why

Every package in `requirements.txt` carries both a lower and an upper bound —
`>=1.4,<1.5`, never just `>=1.4`. That isn't caution for its own sake: an
unpinned package published a breaking release mid-term last year and took a
live class demo down with it. Don't loosen the pins, and don't run
`pip install --upgrade` inside a project virtual environment.

The generic course guide's package table describes **Units 1–2**'s
`requirements.txt` (`chromadb`, `posthog`, `rank-bm25`, and so on). **None of
those packages are in this repo.** Here is what `requirements.txt` actually
pins for FitFindr, verified by reading the file directly:

| Package | What it does | Runs where |
|---|---|---|
| `python-dotenv` | Loads your API key out of `.env` | Your machine |
| `google-genai` | Calls the hosted model that writes outfit suggestions and fit cards | Hosted API |
| `mcp` | Model Context Protocol — Unit 4's rewire of `search_listings` into a standalone tool server. The *only* new package this pair adds; it ships from the start of Unit 3 so there's nothing to install mid-project | Your machine |
| `flask` | The web server behind `serve.py` — Unit 9, when you deploy | Your machine |
| `gunicorn` | How a host runs `serve.py` (`gunicorn serve:app`) — also Unit 9, never run locally before then | Your machine |

Unlike the generic Units 1–2 table, this repo's `requirements.txt` does not
separately pin `certifi`. That's a minor, harmless difference (certifi still
gets installed as a dependency of the other packages) — noted here only because
this file's job is to say what's actually true, not what the general template
says should be true.

### About rate limits

**Two of FitFindr's three tools call the hosted model** (`suggest_outfit` and
`create_fit_card`; `search_listings` is local keyword search and costs nothing),
so one agent run is several requests. A few runs back to back while debugging
can cross the per-minute limit.

The starter handles this for you, and this behavior was directly verified while
building this project:

- **It paces itself.** When you cross the per-minute limit, you'll see something
  like `[rate limit] 15 requests used this minute. Waiting 35s. This is
  normal.` — a pause, not a hang. Seen for real during this project's
  `run_eval.py` test runs.
- **It reuses answers** to prompts already sent, while you're building —
  controlled by `CACHE_ENABLED` in `config.py`.
- **It stops** rather than silently draining your allowance if something loops
  — `QuotaGuard` in `generate.py`, backed by `MAX_ITERATIONS` in `config.py`.
- **It counts your calls** and prints a total when the program exits, e.g.
  `2 model calls this session, 763 prompt + 361 output tokens`.

Test runs (`run_eval.py`) turn caching off on purpose — five tries of the same
question need to be five real answers, not one answer repeated five times.

None of this applies past Unit 4. Units 5–8 call no hosted model at all.

### Later in the term (general context, not part of this repo)

Each project pair has its own starter repo, its own `requirements.txt`, and its
own virtual environment — you repeat step 6 for each one. **Units 3–4 (this
repo) are the lightest install of the course**, per the general guide, and
that's confirmed above: one new package (`mcp`), nothing heavy, no local model
training. Units 5–6 train a classifier locally (PyTorch, ~1.6 GB baseline
model); units 7–8 run a local language model (~550 MB detector); unit 9 is the
capstone with a hosting step. None of that applies to the files in this
repository — it's included here only because this file otherwise only needs
writing once per student, not once per unit.

### When something breaks

| What you see | What it means |
|---|---|
| `Virtual environment: Not active` | You opened a new terminal and didn't activate, or (as in this repo's current state) never created one. Re-run the activate command for your OS |
| `Pinned packages: could not import` | Either the venv isn't active, or `pip install -r requirements.txt` didn't finish. Re-run it and watch for errors |
| `Python version: found 3.14` | Install 3.13, delete the `.venv` folder, and redo step 6 with the new Python |
| `Key hygiene: .env not in .gitignore` | Add a line containing exactly `.env` to `.gitignore` before you commit anything |
| `API key: still holds the placeholder` | You copied `.env.example` but didn't paste your key in |
| `API key: no .env file` | The file may have saved as `.env.txt`. On Windows, turn on file extensions in Explorer to see |
| `MCP: Could not import` | Re-run `pip install -r requirements.txt` — this repo-specific check isn't in the generic guide, but it's in this repo's `test.py` |
| `Project data: ...` failure | `data/listings.json` didn't load, or came back empty. Confirm you're running from inside the repo folder |
| `Model call: 429 / resource exhausted` | A rate limit. Wait a minute and re-run |
| `Model call: key rejected` | Create a fresh key in your own project and paste it again |

Still stuck after one honest attempt? Post the whole `test.py` output in the
help channel — all of it, not just the last line.

---

## Part 2 — FitFindr-specific setup (this repo, Units 3–4)

This is the setup text specific to this project pair, reformatted but not
reworded — it was verified accurate against this repo during the walkthrough's
creation.

### Install the dependencies and add your key

Create a virtual environment inside the repo, install what's in
`requirements.txt`, and make your `.env`:

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

Same key you set up in Unit 1. Nothing new to sign up for — paste it into
`.env`, which is already listed in `.gitignore`. It never goes in your code,
your README, or a message to anyone.

One new package comes with that install: the MCP SDK, which you don't need
until Unit 4. It ships now so there's nothing to install mid-project — confirmed
present in `requirements.txt` from the start of this pair.

Activate the virtual environment every time you open a new terminal. You should
see `(.venv)` at the start of your prompt.

### Confirm it works

```bash
python test.py
```

Come to class with `test.py` passing. If it doesn't pass, post the whole output
in the help channel the day before.

**`RUNNING.md` in the repo is your reference for everything else** — every
command, which command goes with which milestone, what each file does, and what
to do when something breaks. Read it once now, and come back to it whenever
you're stuck. **You don't edit that file.**

### Quote your queries with single quotes, on every operating system

```bash
python app.py ask 'vintage graphic tee under $30'
```

In PowerShell, double quotes turn `$30` into a variable that expands to
nothing — your query silently becomes `vintage graphic tee under `, you get
results with no price ceiling, and nothing warns you. Single quotes are literal
in PowerShell, bash, and zsh alike.

> **Verified directly:** this exact PowerShell behavior was confirmed while
> building this project — it's the reason every command example in
> `PROJECT_WALKTHROUGH.md` and this file uses single quotes.

### About the model calls (project-specific restatement)

Two of this project's three tools call the model, so one run of your agent
sends several requests. That matters more in Units 3–4 than earlier units: a
few runs back to back while debugging can put you over the per-minute limit.
The starter paces you — if you go too fast it waits a moment and tells you it's
waiting. That pause is the starter doing its job, not a bug.

The starter also reuses answers to prompts it has already sent, so re-running
the same query while you debug costs you nothing. Unit 4's testing turns that
off, because there you need the real variation between runs.

---

**See also:** [`PROJECT_WALKTHROUGH.md`](PROJECT_WALKTHROUGH.md) for what this
project does, why it's built this way, and the full milestone-by-milestone
record. [`RUNNING.md`](RUNNING.md) for the complete command reference (not
edited here or there — it's the starter's own file).
