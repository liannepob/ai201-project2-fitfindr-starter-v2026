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

<!-- Three or four sentences: what a user asks for, and what they get back. -->
A user types what they want to thrift in plain language, like "vintage graphic tee under $30, size M". FitFindr parses the query, searches the listings file, and picks the top match. It then suggests one or two outfits that pair the item with pieces from the user's wardrobe and writes a short caption they could post. If nothing matches, it stops before the outfit step and tells the user what to change (keywords, size, or price ceiling).


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

- **What it does:** Searches the listings file for items matching a description, an optional size, and an optional price ceiling.
- **Inputs:** description (str), size (str or None), max_price (float or None)
- **Returns:** A list of listing dicts, best match first, capped at config.SEARCH_RESULT_LIMIT. Each dict has id, title, description, category, style_tags, size, condition, price, colors, brand (can be None), and platform. Listings are scored by keyword overlap with the description and anything scoring zero is dropped. max_price is inclusive. A size matches if any slash-separated token in the listing's size equals the requested size (so "M" matches "S/M"), and "ONE SIZE" listings match any size.
- **When it has nothing:** Returns an empty list [], never None.

### `suggest_outfit`

- **What it does:** Takes a thrifted listing and the user's wardrobe and suggests one or two outfits that pair the item with pieces the user already owns.
- **Inputs:** new_item (dict, one listing), wardrobe (dict with an "items" key holding a list of wardrobe item dicts, each with id, name, category, colors, style_tags, notes)
- **Returns:** A non-empty str of outfit suggestions.
- **When it has nothing:** If wardrobe["items"] is empty, it returns a non-empty str of general styling ideas for the item. It never raises and never returns "".

### `create_fit_card`

- **What it does:** Writes a short caption someone would actually post about the find.
- **Inputs:** outfit (str, the suggestion text from suggest_outfit), new_item (dict, one listing)
- **Returns:** A str caption, two to four sentences, mentioning the item, price, and platform.
- **When it has nothing:** If outfit is empty or whitespace, it returns the str "No outfit suggestion was provided, so I can't write a fit card yet." instead of raising.

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

**Branch rule:** If search_listings returns an empty list, put a message in session["error"] naming what the user could change (keywords, size, price ceiling), leave session["fit_card"] as None, and return without calling suggest_outfit. Otherwise, store results[0] in session["selected_item"] and continue to suggest_outfit, then create_fit_card.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex in agent.py::parse_query, not a model call. It pulls out a price ("under $30"), a size ("size M" or a bare size after a comma), and treats the rest as the description. The same query always parses the same way. Limitation: phrasing like "under thirty dollars" parses to no price.

**What moves through the session:** **Known limits and what was provided:** Search is a plain keyword match, so it can return loosely related items (for example, a slip dress appeared for a graphic tee query with a size and price ceiling). agent.py shipped with run_agent and parse_query already written; my work was the three tools in tools.py and testing the loop's branch on both a matching and an empty query.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30, size M'
[1] parse_query
      in:  vintage graphic tee under $30, size M
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Mesh Long-Sleeve Top — Black, 90s Silk Slip Dress — Floral, Midi Length … +7 more
      →    10 match(es)
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Here are two ways to style your new Y2K butterfly baby tee using pieces from your existing wardrobe:  ### Outf…
      →    10 wardrobe item(s)
[5] create_fit_card
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Found the ultimate 2000s streetwear piece scrolling on depop and honestly cannot get over this butterfly baby …

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two ways to style your new Y2K butterfly baby tee using pieces from your existing wardrobe:

### Outfit 1: Classic Y2K Streetwear
* **New Item:** Y2K Butterfly Baby Tee
* **Bottoms:** Baggy straight-leg jeans (dark wash)
* **Outerwear:** Black cropped zip hoodie (worn open or partially zipped to show off the graphic)
* **Footwear:** Chunky white sneakers
* **Accessories:** Black crossbody bag

*Why it works:* The fitted, graphic nature of the baby tee contrasts perfectly with the relaxed, baggy fit of the dark wash jeans, nailing that quintessential 2000s silhouette. Tying it together with the black cropped hoodie and chunky white sneakers keeps the color palette balanced and casual.

### Outfit 2: Sweet & Edgy Contrast
* **New Item:** Y2K Butterfly Baby Tee
* **Bottoms:** Wide-leg khaki trousers
* **Outerwear:** Vintage black denim jacket
* **Footwear:** Black combat boots
* **Accessories:** Brown leather belt

*Why it works:* This plays on the "cottagecore meets street" vibe. The pink and purple butterfly print pops against the neutral khaki trousers, while the brown belt adds a nice earthy accent. Throwing on the vintage black denim jacket and black combat boots grounds the outfit and gives the sweet butterfly tee a cool, edgy contrast.

  Fit card: Found the ultimate 2000s streetwear piece scrolling on depop and honestly cannot get over this butterfly baby tee. For only $18.0, it's giving major pop-princess-goes-to-the-mall energy and I am so ready to wear it with baggy denim.
```

**Empty search**

```
$ python app.py ask 'designer ballgown size XXS under $5'
[1] parse_query
      in:  designer ballgown size XXS under $5
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    0 match(es)
[3] branch
      →    search returned []: stopping before suggest_outfit

  Nothing in the listings matched description 'designer ballgown', size XXS, under $5.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; drop the size, or try a neighbouring one; raise the price ceiling above $5.
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Great find! Vintage Levi’s 501s in a medium wash are a classic staple that will add a nice textural and color contrast to your darker denim and neutral wardrobe. 

Here are two outfit suggestions using pieces you already own:

### Outfit 1: Casual Streetwear Vibe
*This look leans into the streetwear aesthetic, pairing the vintage medium wash with monochrome basics for an easy, everyday fit.*

* **Top:** White ribbed tank top
* **Outerwear:** Black cropped zip hoodie (worn open or layered)
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag

### Outfit 2: Edgy Layered Look
*This outfit plays with textures and layers, contrasting the classic blue denim with heavy black outerwear and accessories.*

* **Top:** Oversized grey crewneck sweatshirt
* **Outerwear:** Vintage black denim jacket
* **Shoes:** Black combat boots
* **Accessories:** Brown leather belt (to add a nice vintage-complementing earth tone at the waist)
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Found the holy grail of 90s denim on depop today and my butt has literally never looked better. These medium wash Levi's 501s were only $38 and they are giving major off-duty supermodel energy. Can't wait to beat them up and wear them on repeat with crisp white sneakers and a beat-up leather jacket.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* The body of search_listings, filtering on price and size and scoring by keyword overlap, using the helpers I'd already written.
- *What came back:* Working code, but I pasted it in the wrong place, inside suggest_outfit right after its docstring. That caused an IndentationError, and after I fixed that, all my test searches printed [] because search_listings still had its starter `return []`. I found the problem when VS Code's breadcrumb showed the code sitting under suggest_outfit.
- *What I changed:* I moved the block into search_listings, removed the old `return []`, and re-ran the tests. Search then returned the right listings, the size filter narrowed "M" down to the S/M items, and "spacesuit" returned [].

**Moment 2**

- *What I asked for:* Feedback on my draft acceptance criteria.
- *What came back:* My criterion 5 said max_price=40 but "$30 or less," so it contradicted itself. My state and fit card criteria also weren't checkable as written, and my state target of 4 of 5 was soft since no model is involved in passing a dict through the session.
- *What I changed:* I rewrote the state criterion to compare the id of session["selected_item"] with the id of the item that reached suggest_outfit, fixed the price mismatch, and made the fit card criterion about including the price and staying 2 to 4 sentences.

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
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

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
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



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

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



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



