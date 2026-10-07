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
I picked 4 of 5 because my search is a plain keyword match and some phrasings will miss.
<!-- Why 4 of 5 and not 5 of 5? Something about your search, probably —
     "my search is a plain keyword match and some phrasings will miss" is a
     real answer. -->

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
The stop is decided by if not session["search_results"] in agent.py, after `search_listings()`, which is plain Python with no model call. The same query against the same 40 listings gives the same empty list every time, so there's no reason to tolerate a miss, unlike criterion 1 where the model is in play. A miss here would mean the branch is wrong, not that the model was unlucky. The query "designer ballgown size XXS under $5" is also below the cheapest listing ($12).
<!-- Why is 5 of 5 reasonable here when criterion 1 isn't? What's different
     about this path? -->

---

## 3. Something about state
The listing `id` in  `session["selected_item"]` from `search_listings()` should be the same as `new_item` in `suggest_outfit()` 5 out of 5 times. 

<!-- YOU WRITE THIS ONE.

     How would you know that the item your search found is the same item the
     next tool received? Name something countable or observable.

     This is the criterion people find hardest, because state failure doesn't
     look like state failure — it looks like a tool problem. Something that
     compares session["selected_item"] against what actually reached
     suggest_outfit is the shape you're after. -->



**Why this target:**
We want to make sure that the arguments passed from `search_listings` to `suggest_outfit` are the same. This shows that the system logic flow is working correctly and the same inputs are being passed from tool step 1 to step 2. 


---

## 4. The fit card should be concise.
3 out of 5 tries should return a caption that is 500 characters or less. 
<!-- YOU WRITE THIS ONE.

     The fit card calls a model, so the same input can produce different words
     each time. That's not a bug — it's the nature of the tool. So what would
     make it acceptable?

     Think about what you'd actually be unhappy to see. A caption that never
     mentions the price? Two different items producing the same opening
     sentence? A card longer than a caption anyone would post? Any of those can
     be turned into a number. -->



**Why this target:**
I picked 3 of 5 because without a length instruction in the prompt I expect some overruns. `create_fit_card` calls the model at `TEMPERATURE = 0.9` and its prompt (`tools.py`) sets no length limit, so length varies run to run. The spec asks for a 2–4 sentence caption, which fits well under 500 characters. According to social media character limits in 2026 (https://thetextgenerators.com/social-media-character-limits/), a Pinterest post caption is 500 characters. I think that's a reasonable length for most posts on a site that is popularly used to post outfit inpsiration.

---

## 5. The empty wardrobe still gets an outfit and a fit card

With an empty wardrobe (`get_empty_wardrobe()`) and a query that matches at
least one listing, the agent returns a non-empty `outfit_suggestion` and a
non-empty `fit_card`, with `session["error"]` still None, in at least 4 of 5
tries.

**Why this target:**
`suggest_outfit` has a separate branch for `wardrobe['items']` being empty
(tools.py), with its own prompt that asks for general styling advice instead of
combinations from owned pieces. That path is easy to forget, and the loop in
agent.py passes the empty wardrobe straight through with no check of its own,
so if the branch returned "" the fit card would be built on nothing.
I picked 4 of 5 and not 5 of 5 because both calls are model calls
(TEMPERATURE = 0.9), so a rate-limit or model-unavailable failure can still
produce an empty string. I picked 4 of 5 and not 3 of 5 because the search and
branch logic before the model is deterministic, so a miss should be rare and
mean a real bug.

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
