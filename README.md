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

---



<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->

 FitFindr is an agent that helps you put together outfits and provide styling outfits based on a new item that you find. Someone says what they want — "a vintage graphic tee under $30, size M" — and the agent searches listings, works out what it would go with, and writes a caption for it.

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

- **What it does:**
  - searches the listings file and returns matches
- **Inputs:** <!-- name and type each: `max_price` (float), not "a price" -->
  - description (string)
  - size (string)
  - max_price (float or None to skip price filtering)
- **Returns:**
  - a list of matching list dicts, each id, title, description, category, style_tags (list), size,
        condition, price (float), colors (list), brand (str or None), platform
  - sorted by the best match first.
  
- **When it has nothing:**
  - returns an empty list when nothing matches

### `suggest_outfit`

- **What it does:**
  - takes an item and a wardrobe, returns outfit ideas
- **Inputs:**
  - new_item (a listing dictionary)
  - wardrobe (a wardrobe dictionary)
- **Returns:**
  - outfit_ideas (non-empty string)
- **When it has nothing:**
  - print general styling advice

### `create_fit_card`

- **What it does:**
  - writes a short caption someone would post on social media about the style
- **Inputs:**
  - outfit (string)
  - new_item (listing dictionary)
- **Returns:**
  - caption (string)
- **When it has nothing:**
  - Print a message describing the style
  
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

**Branch rule:**
If `search_listings` returns an empty list, put a message in the session and stop. Otherwise, take the first result and go to `suggest_outfit`.  

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** <!-- regex, string splitting, or asking the model — say which -->

1. Parse the query by using regex to find text containing a $ sign to signify price. Use regext to parse for "size" and the phrase that comes afterward, such as "size M" or "size 10."
2. Assign the results of `search_listings()` to `session["search_results"]`.
3. If the length of `session["search_results]` is 0, this will signal the logic to branch off.

**What moves through the session:** <!-- which fields, in what order -->

1. Parse the query using regex to find price and size attributes.
2. Consider the remaining string text as part of the style wardrobe description query.  
3. Call `search_listings()` with the results of the string parsing.
4. Assign the results to `session["search_results]`
5. If nothing comes back, assign an error message to `session["error"]`, saying "No results." Do not call `session_outfit`.
6. Otherwise, call `suggest_outfit` with the selected item and wardrobe.
7. Assign the result to `session["outfit_suggestion"]`

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

``` bash
$ python app.py ask 'vintage graphic tee under $30'
```

``` markdown
Lean into the hoodie's relaxed, worn-in aesthetic with a classic skater/grunge silhouette.

*   **Top:** Vintage Graphic Hoodie (`lst_015`) — *Faded Black*
*   **Bottoms:** Baggy straight-leg jeans, dark wash (`w_001`) — *Dark Blue/Indigo*
*   **Shoes:** Black combat boots (`w_008`) — *Black*
*   **Accessories:** Black crossbody bag (`w_010`) — *Black*

**Why it works:** The oversized, faded look of the hoodie pairs naturally with the low-slung, baggy silhouette of the dark wash denim. Finishing the look with combat boots leans hard into the grunge and streetwear style tags of both the hoodie and the boots.

---

### Outfit 2: Casual Contrast & Comfort
Play with proportions by pairing the cozy, oversized hoodie with lighter footwear and accessories for a balanced everyday look.

*   **Top:** Vintage Graphic Hoodie (`lst_015`) — *Faded Black*
*   **Bottoms:** Wide-leg khaki trousers (`w_002`) — *Khaki/Tan*
*   **Shoes:** Chunky white sneakers (`w_007`) — *White*
*   **Accessories:** Black crossbody bag (`w_010`) — *Black*

**Why it works:** Pairing the faded black hoodie with khaki trousers creates a nice contrast between dark tones and warm earth tones. The chunky white sneakers tie the streetwear vibe together while adding a fresh, casual pop to the lower half of theoutfit.

  Fit card: Here are a few options, depending on your vibe:

**Option 1 (Edgy & Streetwear)**
> One hoodie, two totally different moods. 🖤 Which one are you rocking today: full grunge with the combat boots, or keeping it clean with the khakis and chunky sneakers? Let me know below! 👇 #StreetwearStyle #OutfitInspo #StylingTips

**Option 2 (Short & Punchy)**
> Proof that your favorite vintage hoodie goes with literally everything. 🤌✨ Grunge or casual comfort? 

**Option 3 (Interactive)**
> Outfit 1 or Outfit 2? Styling this faded black graphic hoodie two ways. 🛹👟 #OOTD #StyleInspo #WardrobeStaples

2 model calls this session, 1111 prompt + 530 output tokens
```

**The three tools, tested one at a time**

### 1. `search_listings()`

``` bash
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
```

``` javascript
[{'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```
### 2. `suggest_outfit()`
#### a. Existing wardrobe
``` bash
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
```

``` markdown
Here are three outfit combinations you can wear with your new vintage Levi's 501 jeans (`lst_001`), pulling pieces straight from your wardrobe:

### Outfit 1: Effortless Everyday Casual
*Vibe: Clean, minimal, classic 90s streetwear.*

* **Top:** White ribbed tank top (`w_003`) — tucked into the jeans for a fitted silhouette.
* **Accessories:** Brown leather belt (`w_009`) to accent the waist and tie in the vintage tones.
* **Outerwear:** Vintage black denim jacket (`w_006`) layered on top for a denim-on-denim/textured look.
* **Shoes:** Chunky white sneakers (`w_007`).
* **Bag:** Black crossbody bag (`w_010`).

---

### Outfit 2: Cozy & Relaxed Streetwear
*Vibe: Comfy, oversized proportions, effortless layering.*

* **Top:** White ribbed tank top (`w_003`) as a base layer.
* **Outerwear/Layer:** Oversized grey crewneck sweatshirt (`w_004`) worn loosely over the tank and jeans.
* **Shoes:** Chunky white sneakers (`w_007`).
* **Bag:** Black crossbody bag (`w_010`).

---

### Outfit 3: Edgy Contrast
*Vibe: Grunge-leaning, structured, cool-toned.*

* **Top:** Black cropped zip hoodie (`w_005`) — the crop hits right above the mid-rise/high-rise of the 501s.
* **Shoes:** Black combat boots (`w_008`) for a tougher, grounded silhouette.
* **Accessories:** Brown leather belt (`w_009`) to break up the black and blue.
* **Bag:** Black crossbody bag (`w_010`).
```

#### b. Empty wardrobe returns generic styling advice. 
``` bash
python -c "from tools import suggest_outfit; from utils.data_loader import get_empty_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_empty_wardrobe()))"
```

``` markdown

Here is some general styling advice for your **Vintage Levi’s 501 Jeans (Medium Wash)**, broken down by aesthetic and occasion:

### 1. Casual Streetwear (Embracing the Vibe)
Since these already have that effortless vintage look and street style tag, lean into relaxed, skater-inspired silhouettes:
*   **Tops:** A slightly oversized graphic tee, a vintage band t-shirt, or a heavyweight hoodie (gray, black, or oatmeal). 
*   **Footwear:** Retro sneakers like Nike Dunks, Adidas Sambas, or classic Converse All-Stars.
*   **Accessories:** A canvas tote bag, a silver chain necklace, and a baseball cap.

### 2. Smart-Casual / "Model Off-Duty"
Medium-wash 501s are fantastic for dressing up slightly while keeping that effortless feel:
*   **Tops:** A crisp white button-down shirt (tucked in or worn open over a ribbed tank top) or a classic black turtleneck.
*   **Outerwear:** A structured blazer (oversized and charcoal or plaid looks great with denim) or a black leather biker jacket.
*   **Footwear:** Retro leather loafers, ankle boots, or classic black flats.
*   **Accessories:** A minimalist leather belt and a structured shoulder bag.

### 3. Season-by-Season Tweaks
*   **Fall/Winter:** Layer with a chunky knit cardigan or a trench coat. Throw on a ribbed beanie.
*   **Spring/Summer:** Pair with a simple white ribbed tank top or a cropped baby tee. Roll the cuffs slightly and wear with leather sandals or canvas sneakers.

### 💡 Styling Tip for 501s:
Because vintage 501s are made of 100% rigid cotton with little to no stretch, they look best when balanced with either a fitted top (to highlight the high waist) or a *deliberately* oversized top (for that relaxed, 90s aesthetic). A French tuck (tucking in just the front) works wonders for defining the waist!
```

### 3. `create_fit_card()`

#### a. with outfit suggestion
``` bash
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
```

``` markdown
**Option 1 (Casual & Classic):**
You can never go wrong with a classic. 👖👟✨

**Option 2 (Effortless):**
Put together in 5 minutes, looks good all day. 

**Option 3 (Short & Sweet):**
Jeans, white kicks, and good vibes. ☀️

**Option 4 (Confident):**
My kind of uniform.
```

#### b. no outfit suggestion
``` bash
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('', load_listings()[0]))"
```
``` markdown
Outfit suggestions is missing. Here's a description about the new item instead: 
Nothing beats a classic. ✨ Authentic vintage Levi’s 501s in the dreamiest medium wash with that perfectly worn-in knee fade. 

📏 Size: W30 L30
🏷️ Brand: Levi’s
💸 Price: $38
```

### c. different styling, same listing

``` bash
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('little black dress', load_listings()[0])"
```

``` markdown
Here are a few options, depending on the vibe you’re going for:

**Chic & Timeless:**
> You can never go wrong. ✨🖤 #LBD #TimelessStyle

**Sassy & Confident:**
> Mentally unstable, but my little black dress is doing the heavy lifting. 🥂🖤 

**Short & Sweet:**
> Always in style. 🖤

**Edgy:**
> Less talk, more little black dress. ⚡️🖤

**Dinner/Night Out:**
> Problem solver. (The problem is what to wear). 🍸✨
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* Break down a simple logic for parsing of the size attribute. 
- *What came back:* A suggestion for a helper function with tokens for expected sizes. A mapping to categorize size small and medium as well as identify shoe sizes. 
- *What I changed:* Keep the logic within the `search_listings` function. Even though it's messier, I just want to have all the information in one place during the simple first pass. 

**Moment 2**

- *What I asked for:* Draft a way to score listings by keyword overlap. Drop the scores of 0. 
- *What came back:* 
  - 1. Suggestion to lowercase all the text and split on spaces. 
  - 2. Then, create a set of the words parsed. 
  - 3. Create a score variable which represents how many query words appear in that set. 
  - 4. Store the results `(score, listing)'` in a new list called scored.
  - 5. Only append to `scored` when `score > 0`
  - 6. Sort `scored` by highest score first.
  - 7. Only keep 10 highest scores.
  - 8. If nothing is scored, return an empty list `[]`.  
  - 9. Suggestions to implement punctuation parsing. For example "t-shirt" and "tee shirt" would not be evaluated as similar words. 
- *What I changed:*
  - Steps 1-8 above.
  - Ignored additional suggestion for punctation parsing.
  - Keep it simple on first pass implementation.

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

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
