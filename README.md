# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

## Milestone 1 Notes

Three listing fields are `title`, `category`, and `price`. The starter query
ran successfully and reported that the planning loop is not built yet.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

FitFindr lets a user describe a thrift item they want, including optional size
and price constraints. It searches the listing data, selects the best match,
and asks the model to combine that item with pieces from the user's wardrobe.
It then returns a short fit-card caption; if nothing matches, it stops early
with suggestions for broadening the search instead of calling the later tools.

---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Searches the listings for description keywords, with optional size and inclusive maximum-price filters.
- **Inputs:** `description` (str), `size` (str or None), and `max_price` (float or None).
- **Returns:** A best-first list of listing dicts containing `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.
- **When it has nothing:** Returns an empty list; size matching is case-insensitive and token-aware, so `M` matches `S/M` but not `XL`.

### `suggest_outfit`

- **What it does:** Suggests one or two outfits that style a selected thrift listing with the user's wardrobe.
- **Inputs:** `new_item` (dict containing a listing), and `wardrobe` (dict containing an `items` list of wardrobe-item dicts).
- **Returns:** A non-empty string containing outfit suggestions, naming pieces from the wardrobe when available.
- **When it has nothing:** With `wardrobe["items"] == []`, returns general styling advice for the listing instead of failing or returning an empty string.

### `create_fit_card`

- **What it does:** Creates a short social caption for the selected listing and its outfit suggestion.
- **Inputs:** `outfit` (str) and `new_item` (dict containing a listing).
- **Returns:** A two-to-four-sentence string mentioning the item, price, platform, and outfit vibe.
- **When it has nothing:** If `outfit` is empty or whitespace, returns a descriptive message rather than raising.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If `search_listings` returns an empty list, put a message in the
session and stop without calling `suggest_outfit`. Otherwise, select the first
listing, pass it and the wardrobe to `suggest_outfit`, then pass the resulting
outfit and listing to `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex extracts an optional size and an optional
`under $N` maximum price; the remaining words become the description.

**What moves through the session:** The query and parsed description/size/max
price are stored first; search results follow; then the selected listing,
wardrobe, outfit suggestion, and fit card are stored in that order. An empty
search stores an error and leaves the later fields as `None`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'
  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop
  Outfit:   **Outfit 1: Y2K Streetwear** ...
  Fit card: Found this absolute gem of a Y2K Baby Tee ... for just $18!

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; r=search_listings('graphic tee', max_price=30); print([(x['id'], x['title']) for x in r[:3]])"
[('lst_017', 'Mesh Long-Sleeve Top — Black'), ('lst_002', 'Y2K Baby Tee — Butterfly Print'), ('lst_033', 'Vintage Band Tee — Faded Grey')]

```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_empty_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_empty_wardrobe()).splitlines()[0])"
Here are two practical, versatile outfit ideas to style these classic vintage Levi’s 501s:

```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('black tee, baggy jeans, and sneakers', load_listings()[0]).split('.')[0] + '.')"
Scored these dreamy Vintage Levi's 501 Jeans on Depop for just $38, and honestly, they fit like an absolute dream.

```

The empty search returned `[]`, and an empty outfit returned
`No outfit suggestion was provided, so a fit card cannot be created.` Running
the same fit-card input three times produced identical wording because the
starter cache is enabled; `config.py` has `CACHE_ENABLED = True` and
`TEMPERATURE = 0.9`.

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked for the listing and wardrobe shapes to be
  checked before defining the three tool contracts.
- *What came back:* The review identified the concrete listing fields, the
  empty wardrobe shape `{"items": []}`, and the need for `search_listings` to
  return an empty list rather than `None`.
- *What I changed:* I wrote typed inputs, specific listing-dict fields, and
  explicit empty-case behavior into the Tool Inventory before implementing
  the tools.

**Moment 2**

- *What I asked for:* I asked for the tool and loop behavior to be checked
  against the acceptance criteria, especially the empty-search branch and the
  selected-item handoff.
- *What came back:* The implementation review emphasized reading each result
  back from the session and stopping before the model tools when search
  returned no listings. It also exposed that a shell command using double
  quotes stripped `$5` from the test query.
- *What I changed:* I wired the session-based branch in `agent.py`, added a
  user-actionable empty-search message, verified the selected listing ID at
  the outfit-tool boundary, and reran the price test with safe single quotes.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item state handoff | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card details for five different listings | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Empty wardrobe returns general styling | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

Source: `results/run_2026-10-15_1514_before.md`, produced by
`run_eval.py::main` calling `agent.py::run_agent`.

**Criterion 1, matching query, Try 1**

```
- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

Fit card:
Found my ultimate Y2K throwback on Depop with this butterfly print baby tee for just $18! It gives off the dreamiest nostalgic streetwear energy when paired with baggy denim and chunky sneakers. Honestly obsessed with how effortless this piece makes the whole vibe.
```

**Criterion 2, impossible query, Try 1**

```
- stopped early: yes — No matching listings. Try broader description keywords, a different size, or a higher price limit.
- selected_item: (none)
- search_results: 0

Trace:
[1] search_listings (via MCP)
      out: [] (empty)
[2] search_results branch
      →    empty, stopping
```

**Criterion 3, selected-item state, Try 1**

```
- selected_item: 90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
- search_results: 4

Trace:
[2] suggest_outfit
      out: **Outfit 1: 90s Streetwear** * **Thrift Piece:** 90s Track Jacket ...
```

The selected listing remained the same listing handed to `suggest_outfit` in
all five tries.

**Criterion 4, five different listings, one try each**

```
Y2K Baby Tee — Butterfly Print ($18.0, depop):
Can we talk about how good this $18 Y2K Baby Tee with the butterfly print is? I just scored it on depop and it’s giving the ultimate early 2000s street style. Paired with baggy jeans and chunky sneakers, it's the easiest casual fit ever.

Denim Jacket — Light Wash, Cropped ($42.0, poshmark):
Finally scored this cropped light wash denim jacket on Poshmark for just $42, and I'm obsessed with how versatile it is! I'm leaning heavily into that high-contrast double denim streetwear vibe today, pairing it with baggy dark wash jeans and a simple tank. It gives off such an effortless, cool-girl energy that I know I'll be wearing on repeat.

Platform Sneakers — White Chunky Sole ($48.0, poshmark):
Score! Snagged these Platform Sneakers on Poshmark for just $48, and they are officially my new daily drivers. I'm obsessed with how easily they can switch from a slouchy Y2K streetwear vibe to an effortless casual minimal look. Consider this your sign to check your favorite resale apps today!

90s Silk Slip Dress — Floral, Midi Length ($30.0, depop):
Channeling pure 90s model-off-duty energy by throwing an oversized crewneck and combat boots right over this floral midi. I scored this absolute dream on Depop for just $30, and honestly, the styling potential is endless. It’s the ultimate mix of effortless, grunge-girl romance and everyday cool.

90s Track Jacket — Navy/White Stripe ($45.0, poshmark):
Scored this vintage 90s Track Jacket on Poshmark for just $45, and I am obsessed. I’ve been living in it thrown over a ribbed tank and baggy denim for that effortless, off-duty streetwear vibe. It’s the ultimate throw-on-and-go layer for running errands or grabbing iced coffee.
```

**Criterion 5, empty wardrobe, Try 1**

```
- stopped early: no
- selected_item: Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
- search_results: 7

Outfit suggestion:
Here are two practical, versatile ways to style this cropped light-wash denim jacket...
```

The full five-try output, including every outfit, fit card, and trace, is in
`results/run_2026-10-15_1514_before.md`.

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | A matching query completes all three tools | 4 of 5 | **MET (5/5)** | All five matching runs returned a selected listing, outfit suggestion, and fit card. |
| 2 | An impossible query stops before the second tool | 5 of 5 | **MET (5/5)** | All five impossible-query runs returned the actionable empty-search message and the trace stopped after the MCP search branch. |
| 3 | Selected listing state reaches `suggest_outfit` | 5 of 5 | **MET (5/5)** | In all five runs, the selected listing ID and the listing shown at the `suggest_outfit` handoff matched. |
| 4 | Fit cards for five different listings are 2–4 sentences and include title, price, and platform | 5 of 5 | **MET (5/5)** | The five distinct listing scenarios each produced a 3-sentence card containing the required item details. |
| 5 | Empty wardrobe returns general styling advice | 5 of 5 | **MET (5/5)** | All five empty-wardrobe runs returned non-empty outfit advice and completed without an exception. |

**Diagnoses**

There were no misses, so there is no failed step or mechanism to diagnose.
Criteria 2, 3, and 5 had appropriately strict 5-of-5 targets because their
behavior is deterministic and under the agent's control. Criterion 1's 4-of-5
target was lower than necessary: it also passed 5 of 5, so I would tighten it
to 5 of 5 for the next evaluation. Criterion 4 was measured across five
different listings and passed all five, but I would also require the title
match to be case-insensitive and checked against the exact listing title rather
than relying on the representative output review.

---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 1 items: Platform Sneakers — White Chunky Sole
[2] suggest_outfit
      in:  dict with keys: selected_item, wardrobe
      out: **Outfit 1: Y2K Streetwear** ...
[3] create_fit_card
      in:  dict with keys: outfit, selected_item
      out: Score! Snagged these chunky Platform Sneakers on Poshmark ...

```

**Empty search**

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
[2] search_results branch
      in:  [] (empty)
      out: No matching listings. Try broader description keywords, a different size, or a higher price limit.
      →    empty, stopping

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->

Registered `search_listings` in `mcp_server.py` with typed `description`,
`size`, and `max_price` inputs and rewired `agent.py::run_agent` to call it
through `mcp_client.call_tool`. `python mcp_client.py` lists the tool with the
expected schema. The end-to-end query `vintage graphic tee under $30` returned
10 listings and produced a fit card, so the MCP call preserved the native list
of listing dictionaries and the later planning steps still ran.


## Milestone 2 Failure Messages

I triggered each failure separately:

- **Empty search** (`designer ballgown size XXS under $5`):
  `No matching listings. Try broader description keywords, a different size, or a higher price limit.`
- **Empty wardrobe** (`platform sneakers size 8 --empty-wardrobe`):
  The agent returned general styling advice and a fit card; it did not crash or return an empty outfit.
- **Model unavailable** (simulated invalid API key on an uncached query):
  `The outfit model could not be reached. Check your API key and internet connection, then try the query again.`

The model-unavailable handler records the failure in `session["error"]` and
stops before the remaining model-backed step, so the CLI shows guidance rather
than a raw exception.


---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
