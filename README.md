# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

device: cpu  (CPU)
torch:  2.14.1+cpu

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

TakeMeter sorts r/stocks posts into three labels: `analysis` (a conclusion supported by a specific, checkable fact), `hot_take` (a confident claim with no support), and `reaction` (an immediate response to something that just happened). The question is whether a fine-tuned small model can tell a well-supported argument apart from a loud opinion. That matters in a retail-investing community, where both often sound equally confident.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `hot_take`

**Definition:** A confident claim or prediction about a stock or the market, with no support offered.

**Example 1:**
> the key to make money in this market is buy calls on stocks that are extremely overbought and keep shorting those that are extremely oversold. works about 99.8% of the time

**Example 2:**
> SMCI is a cheap stock compared to DELL & HPE. It will catch up soon IMHO.

### `analysis`

**Definition:** A conclusion backed by a specific, checkable fact (a figure, a filing, a rule, or an event).

**Example 1:**
> For anyone with an account there: the stock dropping and your account being at risk are two different things. Your securities are held separately from the company's own money, and SIPC covers up to $500k per customer (of which $250k cash) if a US broker fails. This report is a congressional panel, not a regulator action, so nothing changes for accounts today. If the cash part worries you, that's the bit to keep under the limit.

**Example 2:**
> AVGO is fundamentally supported. Based on forward guidance, both NVDA and AVGO trade at <20x forward EV/EBITDA and have 50%+ net margins. By comparison, AMD is over 60x EV/EBITDA with 20% net margins. I just don't see AMD growing into its current valuation.

### `reaction`

**Definition:** An immediate response, observation, or question about an event, price, or data point that just happened, which states it without drawing a conclusion.

**Example 1:**
> 30Y hit 5.703% today, fresh new 24 year high. Please do your part to help markets by buying this and become a bagholder of US debt. Thank you for your attention to this matter.

**Example 2:**
> SPY finally went red like last 15 minutes... after 10Y absolutely exploded following PCE today. Down only 0.2%.

### The hardest boundary

**Which two labels:** `hot_take` vs `analysis`

**The decision rule I used every time:**
<!-- e.g. "If the post names a specific checkable fact, it's `analysis`, even
     if the tone is heated." -->

If the post names a fact concrete enough to check (e.g. "<20x forward EV/EBITDA", "SIPC covers $500k") and uses it to support a conclusion, it is `analysis`. If it only gestures at vague metrics ("forward PE and other metrics", "strong fundamentals") or offers only personal interpretation with no fact, it is `hot_take`. If it draws no conclusion and just describes something that just happened, it is `reaction`.


---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** r/stocks

**How I labelled them:** 
<!-- Cold first? Pre-labelled with AI and corrected?
Say so plainly — the disclosure is required, not penalised. -->

I first labelled about 20–30 posts by hand, cold, to work out the definitions below. I then had an AI pre-label the whole set using those definitions, and I read every post and corrected the labels one by one.

- `analysis`: makes an argument whose conclusion is supported by a specific, checkable fact. The fact must be concrete enough to verify (for example, "the company's revenue grew this quarter"), but it does not need to include a number or a cited source. The fact must still hold up after removing anecdotes ("I know people", "anecdotally") from the reasoning. If the author leans on a vague reference ("strong fundamentals", "numbers look good", "other metrics") as a reason, it is not checkable.

- `hot_take`: a confident claim with no support offered.
- `reaction`: an immediate response, observation, or question about something that just happened. Because many posts are very short, a post that clearly ties to a specific moment, price, or event counts as `reaction` when it only states it and draws no conclusion.

Decision order:
1. Conclusion plus a specific, checkable fact -> `analysis`
2. Otherwise, an immediate response to a just-happened event -> `reaction`
3. Otherwise, a confident claim with no support -> `hot_take`


**Counts per label:**

| Label | Count | Share |
|---|---|---|
| analysis | 50 | 24.6% |
| hot_take | 107 | 52.7% |
| reaction | 46 | 22.7% |
| **Total** | 203 | 100% |

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:*  
AVGO is the only choice here and based on forward PE and other metrics, an undervalued gem. TSLA just toxic  

> *Could have been:* analysis

> *I chose hot_take, because:*
Mentions forward PE but gives no supporting figure. (Forward PE doesn’t count as checkable, since estimates may differ across institutions.)

**2.**
> *The post:*  
I think the market is treating this as a growth scare that buys cuts, which is why futures jumped. What I watch is the revisions: July flipped negative and August got cut hard. If the next couple reports keep that pattern, the soft-landing story gets rewritten as labor already weakening while stocks sit at highs. One print is noise; the revision trail is the tell.

> *Could have been:* analysis

> *I chose hot_take, because:*
The revised data is verifiable, but the interpretation is a personal judgment.

**3.**
> *The post:*  
Twilio $TWLO is set to join the S&P 500 on October 6. What an incredible journey for this company
https://finance.yahoo.com/markets/stocks/articles/twilio-join-p-500-p-222951025.html
Twilio has had one of the most remarkable turnarounds and is now set to join the S&P 500. This sets the stock up for more growth ahead. The future looks bright for 2027 and beyond as tailwinds for agentic AI, and their own AI products released this year come to fruition. The stock is now up 105% YTD and 184% in 1 year and has reached a $46B market cap. Twilio has bought back billions of dollars of shares over the past 3 years and is now generating strong free cash flow.
The future looks bright!

> *Could have been:* hot_take

> *I chose reaction, because:*
Overall, it expresses optimism about the inclusion in the S&P 500.


---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** "distilbert-base-uncased"

**Settings:** <!-- epochs, learning rate, batch size, seed -->
EPOCHS = 10  
LEARNING_RATE = 2e-5  
BATCH_SIZE = 16  
MAX_LENGTH = 256  

**Anything I changed from the defaults, and why:**  
**Max length: 128 → 256**
In the training set, 55.9% of `analysis` posts (19 of 34) were longer than 128 tokens, compared with 13.3% of `hot_take` posts (10 of 75) and 18.8% of `reaction` posts (6 of 32). The median `analysis` post is 154 tokens, so the evidence behind an argument was often cut off. I chose 256 rather than 512 because `padding="max_length"` makes compute grow with the limit, and 512 would only add the few longest posts (the longest are 690 to 765 tokens). I did not compare 128 and 256 directly, so this choice rests on the truncation counts, not on a score difference.

**Epochs: 3 → 10**
With 3 epochs (about 27 updates), validation accuracy was 0.516 and macro F1 was 0.227 at every epoch. That equals always predicting `hot_take` (16 of 31 validation posts), so the model had only learned the class prior. With 10 epochs (about 90 updates), validation accuracy rose to 0.774 and macro F1 to 0.756. Validation loss kept falling through epoch 10 (0.997 to 0.669), and epochs 9 and 10 gave the same scores, so I kept 10. I left the learning rate at 2e-5 because more epochs alone fixed the collapse.

**Split sizes:** 
<!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->
'reaction' has only 7 examples in the test split.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* I wrote the accuracy and F1 criteria myself, then asked the AI to explain what the other areas (balance, consistency, confidence) mean and how they are usually tested on a classifier like mine.
- *What came back:* An explanation of each area and how it is measured, plus draft criteria with numbers and reasons. It also suggested label balance as a criterion.
- *What I changed:* I dropped balance because I control it by hand and it doesn't test the model. I rewrote the accuracy reason in my own words, changed confidence to a top-third vs bottom-third gap of 0.15 so it doesn't repeat accuracy.

**Moment 2**

- *What I asked for:* I gave the AI my three label definitions and asked it to pre-label my collected posts, and to flag the hard cases while describing which label definitions might need to be more explicit. I then checked its labels and flags myself.
- *What came back:* Pre-labels with a short note for each post, plus a list of hard cases with the competing label and the reason each was hard. While we discussed the `analysis` boundary, it suggested a general deletion test: remove every unverifiable statement and see whether the checkable facts still support the conclusion (supported → `analysis`, not supported → `hot_take`).
- *What I changed:* I did not adopt the general deletion test. I rewrote the `analysis` definition so that the fact must be concrete enough to verify but does not need a number or a cited source, and it must still hold after removing anecdotes ("I know people", "anecdotally"). A vague reference such as "strong fundamentals", "numbers look good" or "other metrics" used as a reason does not count as checkable, so only anecdotes are deleted before judging.

**Pre-labelling disclosure:** An AI pre-labelled the full set using my definitions, and I read and checked every post myself. I changed about five labels, and in the same pass I rewrote the `analysis` definition (Moment 2), so the definitions above are the revised version.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->
**Moment 1**

- *What I asked for:* I asked how to rewrite the `reaction` definition so it separates cleanly from `hot_take`, and which of my existing posts would break the new rule.
- *What came back:* A rewritten `reaction` definition and a list of posts that broke it, plus a suggestion to add more reaction examples.
- *What I changed:* I kept the idea of the rewrite but wrote the final wording myself. I read each flagged post and relabelled 3 from `reaction` to `hot_take`, then wrote 6 new short, clear reaction posts myself instead of using generated ones, bringing the file to 209 rows.

**Moment 2**

- *What I asked for:* Which pile my disagreement on #2 (the wrong-size shoes post) belongs in. I labeled it `hot_take` because "every PB this year in wrong-size shoes" implies that shoe fit matters less than people think.
- *What came back:* Claude kept supporting `reaction`. It said the post only reports something about my own shoes, and that "asserts" in rule 2 means a general claim, not an implied one.
- *What I changed:* I kept my `hot_take` label and put the post in pile (b). The discussion showed that rule 2 does not say whether an implied general point counts, so the disagreement comes from a gap in the definitions, not from me applying the rule loosely.


---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.645 | 0.839 | +0.194 |
| Macro F1 | 0.542 | 0.788 | +0.246 |
| F1 — `analysis` | 0.222 | 0.875 | +0.653 |
| F1 — `hot_take` | 0.737 | 0.889 | +0.152 |
| F1 — `reaction` | 0.667 | 0.600 | -0.067 |

**Note:** This baseline was run without label_definitions.txt (bare label names only). With my definitions it scored accuracy 0.258 / macro F1 0.200; without them, 0.645 / 0.542. I report the stronger run as the baseline.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->
I predict the trained model will beat the baseline most on `analysis`, because the zero-shot baseline barely recognizes it (F1 0.222) while fine-tuning can learn that long, data-heavy posts are analysis.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->
Accuracy rose from 0.645 to 0.839 and macro F1 from 0.542 to 0.788. Most of the gain came from analysis (F1 0.222 → 0.875), which matches my prediction. The one exception is reaction (0.667 → 0.600), but with only 7 test posts a difference of one or two posts moves that score a lot


---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.839 | 0.710 | 0.742 | 0.129 |
| Macro F1 | 0.788 | 0.604 | 0.678 | 0.184 |
| F1 — `analysis` | 0.875 | 0.800 | 0.800 | 0.075 |
| F1 — `hot_take` | 0.889 | 0.789 | 0.789 | 0.099 |
| F1 — `reaction` | 0.600 | 0.222 | 0.444 | 0.378 |

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | 0.7 | PASS | PASS | PASS | MET |
| 2. Per-label F1 | at least 0.75 for `hot_take`, 0.55 for `analysis`, and 0.55 for `reaction` | 3/3 | 2/3 | 2/3 | MISS |
| 3.Confusion matrix | at most 30% of true `analysis` posts are predicted as `hot_take` (about 2 of 7) and at most 20% of true `hot_take` posts are predicted as `analysis` (about 3 of 16) | PASS | PASS | PASS | MET |
| 4. Beats the baseline on accuracy | 0.66 | PASS | PASS | PASS | MET |
| 5. Confidence | accuracy on the most-confident third of predictions is at least 0.15 higher than accuracy on the least-confident third | PASS | PASS | PASS | MET |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 7 | 1 | 0 |
| **hot_take** | 0 | 16 | 0 |
| **reaction** | 1 | 3 | 3 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->
3 real `reaction` posts were called `hot_take`, and none went the other way. `reaction` has precision 1.0 but recall 0.43, so the model only recognises its most typical form. I suspect this is because `reaction` posts are varied (short questions, emotional remarks, price comments) and I have few of them, so the model did not see enough variety. With only 7 test posts this is a hint, not a firm conclusion.


---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | >=0.7 | MET | accuracy are 0.84, 0.71, 0.74, all larger than 0.7 |
| 2 | Per-label performance | at least 0.75 for `hot_take`, 0.55 for `analysis`, and 0.55 for `reaction` | MISS | `reaction` label's F1 (seed 7 and 2024) is lower than 0.55, not all three labels met the criteria|
| 3 | Confusion matrix | at most 30% of true `analysis` posts are predicted as `hot_take` and at most 20% of true `hot_take` posts are predicted as `analysis` | PASS | true `analysis` posts are predicted as `hot_take` is 12.5% < 30%, and true `hot_take` posts are predicted as `analysis` is 0% < 20% |
| 4 | Beats the baseline on accuracy | 0.66 | MET | accuracy are 0.84, 0.71, 0.74, all larger than 0.66 |
| 5 | Confidence | accuracy on the most-confident third of predictions is at least 0.15 higher than accuracy on the least-confident third | MET | difference are 0.3, 0.3, 0.4 seperately, all >= 0.15 |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->
Criterion 2 (per-label F1) — MISSED. 
Cause: reaction is under-represented, and my own labelling of it is the least stable.
`reaction` F1 was 0.600 / 0.222 / 0.444, so only seed 42 reached 0.55. analysis and `hot_take` were fine (hot_take 0.789 to 0.889).
- (too few examples): reaction has 32 training posts against 75 hot_take, and only 7 test posts, so one post moves F1 by a lot. This explains the spread across seeds.
- (direction): In seed 42, reaction has precision 1.0 but recall 0.43. 3 of 7 were called hot_take and 1 analysis, and none went the other way. The model only recognises the typical form of reaction and sends the rest to hot_take.
- (agreement report): Both of my disagreements with staff involved reaction. #1 was a rule I applied loosely (pile a). #2 was a gap in the definitions about whether an implied general point makes a post a hot_take (pile b).
- So the reaction boundary is where I was inconsistent, and the same inconsistency may be in my 200 labels.

Revising my assumption: I expected the `analysis`/`hot_take` boundary to be the hard one, but in seed 42 it was the cleanest (1 and 0 errors). The weak spot was `reaction`, which I had treated as the easy class because it looks distinctive.


---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** _28_ / 30 = _93_%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:* "I tracked sleep against session quality for eleven weeks. Under seven hours the next day's intervals were 6-8 seconds per mile slower, every time bar one. Small sample, one runner, but it's consistent enough that I've stopped scheduling hard days after late shifts."
>
> *Staff said / I said:* `analysis`, `reaction`
>
> *My call, and why:* The 6-8 seconds per mile gap over eleven weeks is what the conclusion rests on, and the writer flags the sample's limits, which is how someone weighing evidence writes. I read the first-person voice ("I've stopped scheduling hard days") as a personal report.
> *Which pile:* (a). The rule covered it and I applied it loosely.


**2.**
> *The post:* Just realised the shoes I set every PB in this year are the ones I bought by accident in the wrong size.
>
> *Staff said / I said:* `reaction`, `hot_take`
>
> *My call, and why:* I read it as a hot take. The post says every PB this year came in wrong-size shoes, which works as a general point that fit and shoe choice matter less than people think. The writer is making that point through irony, and "just realised" only frames how they got there. Staff read it as a personal discovery with nothing general asserted, which is also reasonable.
>
> *Which pile:* (b). Rule 2 separates a reaction from a general assertion but does not say whether an implied general point counts.


**What the pattern in my disagreements tells me:** Both misses were first-person posts, and in both I leaned on how the post sounded instead of the rule's test. In #1 I let the "I" voice outweigh the eleven weeks of data. In #2 the rule was silent on an implied general point, so that one is a gap in the definitions. Two of 30 is too small to find the pattern.

---

## The Improvement

**What I changed:** : I rewrote the reaction definition in label_definitions.txt. It now covers a response to something that just happened (a price move, news, a personal gain or loss) that does not draw a general conclusion. If a post can be restated as a prediction or advice, it is a hot_take. I then relabelled the posts that broke this rule (3 reaction posts became hot_take) and added 6 new short, clear reaction posts, bringing the file to 209 rows.

**Which diagnosis pointed at it:**
The new definition closes that gap, and the extra reaction examples address the small class size.


### Run Log — After

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.812 | 0.812 | 0.750 | 0.062 |
| Macro F1 | 0.762 | 0.782 | 0.719 | 0.063 |
| F1 — `analysis` | 0.875 | 0.824 | 0.667 | 0.208 |
| F1 — `hot_take` | 0.865 | 0.857 | 0.824 | 0.041 |
| F1 — `reaction` | 0.545 | 0.667 | 0.667 | 0.121 |


| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | 0.7 | PASS | PASS | PASS | MET |
| 2. Per-label F1 | at least 0.75 for `hot_take`, 0.55 for `analysis`, and 0.55 for `reaction` | 2/3 | 3/3 | 3/3 | MISS |
| 3.Confusion matrix | at most 30% of true `analysis` posts are predicted as `hot_take` (about 2 of 7) and at most 20% of true `hot_take` posts are predicted as `analysis` (about 3 of 16) | PASS | PASS | PASS | MET |
| 4. Beats the baseline on accuracy | 0.66 | PASS | PASS | PASS | MET |
| 5. Confidence | accuracy on the most-confident third of predictions is at least 0.15 higher than accuracy on the least-confident third | PASS | FAIL | PASS | MISS |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->
Partly. F1 went from 0.600 / 0.222 / 0.444 to 0.545 / 0.667 / 0.667, and accuracy spread fell from 0.129 to 0.062. Criterion 2 is borderline: seed 42's reaction F1 is 0.545, which is below 0.55 strictly but rounds to 0.55, so I count it as missed by 0.005. Criterion 5 now misses (confidence gap 0.3 / 0.1 / 0.3, seed 7 is under 0.15), but each third has only about 10 posts, so one post moves the gap by 0.1. The test set is also different from before (209 rows instead of 203), so this is not a like-for-like comparison.


---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->
Criterion 2 is borderline (seed 42 reaction F1 is 0.545 against 0.55) and Criterion 5 is missed (seed 7 confidence gap is 0.1). I stopped because the test split holds only 7 reaction posts and about 10 posts per confidence third, so a single post moves these numbers more than any further label change would. 


**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 7 | 1 | 0 |
| **hot_take** | 1 | 15 | 1 |
| **reaction** | 1 | 2 | 4 |

In seed 7 the model still labels 2 of the 7 true reaction posts as hot_take (and 1 as analysis), catching only 4, while just 1 hot_take is called reaction. So it only recognises the plainest reaction posts, and sends anything with a hint of a conclusion to hot_take. About 35 training examples are too few for it to learn that boundary reliably, or my own labelling of it is still not consistent enough.


<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
