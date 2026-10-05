# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
Perfection isn't realistic here. My search is a plain keyword match, so some phrasings will miss, and the fit card comes from a model that can vary. 4 of 5 (80%) is reliable enough to hand to a user, while 5 of 5 would be unfair to those two sources of variation.
<!-- Why 4 of 5 and not 5 of 5? Something about your search, probably —
     "my search is a plain keyword match and some phrasings will miss" is a
     real answer. -->

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
5 of 5 because this path never calls the model. The branch is a simple check for an empty list, so it should behave the same every time. The message names what to change (size, price, or keywords) so the user knows how to retry.<!-- Why is 5 of 5 reasonable here when criterion 1 isn't? What's different
     about this path? -->


---

## 3. Something about state
In 4 of 5 happy-path runs, the i.d. of session ["selected_item"] is identical to the i.d. of the item that reached 'suggest_outfit'. 

<!-- YOU WRITE THIS ONE.

     How would you know that the item your search found is the same item the
     next tool received? Name something countable or observable.

     This is the criterion people find hardest, because state failure doesn't
     look like state failure — it looks like a tool problem. Something that
     compares session["selected_item"] against what actually reached
     suggest_outfit is the shape you're after. -->



**Why this target:**
Checking the id through the session confirms that the item the search found is the same item the next tool received, so the outfit is built for the listing the user actually got.


---

## 4. Something about the fit card
In 4 of 5 happy-path runs, the fit card includes the listing's price and is 2 to 4 sentences long.
<!-- YOU WRITE THIS ONE.

     The fit card calls a model, so the same input can produce different words
     each time. That's not a bug — it's the nature of the tool. So what would
     make it acceptable?

     Think about what you'd actually be unhappy to see. A caption that never
     mentions the price? Two different items producing the same opening
     sentence? A card longer than a caption anyone would post? Any of those can
     be turned into a number. -->



**Why this target:**
A model writes the caption, so the wording changes from run to run and it will occasionally skip the price or run long. 4 of 5 allows for that variation without letting a caption that ignores the price count as working.

---

## 5. Your choice
In 4 of 5 searches with max_price=40 that match at least one listing, every listing returned costs $40 or less.
<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. Speed, the empty
     wardrobe path, what happens when the model can't be reached, whether the
     search respects a price ceiling — anything, as long as it names a number
     or an observable outcome. -->



**Why this target:**
The results must stay within the max_price the user gives and never include anything above it. This is plain filtering with no model, so a miss would point to a real bug in the search, not randomness. 
---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
