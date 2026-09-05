# A History of Measurement Failures, and Eight General Rules
### 50 submissions, 48 experiments, every one with a control — Pokémon TCG AI Battle Challenge

Participant **shiratama** (`dotshiratama`, solo) / Simulation Category, 2026-06-20 to 08-16

Over two months, conclusions I had recorded as "this is stronger" kept collapsing days later
**as defects in my own measuring instrument**. It happened eight times. The largest was a simulator
sitting **42.4pp** from the live ladder: for weeks,
**I was selecting candidates by looking at a difference that did not exist.**

---

## 1. Result

Score **846.1**, **673rd of 6,892 teams (top 9.8%)**, from 50 submissions across 48 experiments.
**These are provisional figures as of 2026-08-17 11:23 UTC**: the ladder kept playing after the
deadline, and while I checked it moved to 841.6, then 835.5.

Of the 48 experiments, **0 produced a significant improvement**, **2 a significant degradation**,
10 or more were null, **8 were defects in the measuring instrument**, and I retracted 10 or more of
my own conclusions. **That distribution is the finding**, and this report is
**a record of what does not work, measured with controls across 50 submissions.**

---

## 2. Method and final artifact

The submission is `main.py`, `model.pth`, `deck.csv` and the official engine.

**The policy is a small Transformer with two heads, AlphaZero style.** The encoder reads the board
and emits a value; the decoder reads the candidate actions and emits a policy.
With `d_model=128, heads=2, ff=256, enc=1, dec=1`, **98% of the 12.53M parameters are embeddings**
and the body is 0.265M. Cards enter as sparse (index, value) pairs summed by `EmbeddingBag`, making
set information order-independent, and obvious quantities such as damage dealt go to the decoder
rather than being rediscovered.

**Training is behaviour cloning (BC) and nothing else**: the top team's replays,
4,755 games / 428,461 samples, 12 epochs. Reinforcement learning was tried in four forms over ten
runs; every one was null. **Inference is greedy, with search disabled** (5.1).
Every analysis below comes from **145GB** of ladder replays covering seven days.

Rule-based play plateaued at ~450-570 and self-BC with self-play RL stalled at 46%;
**switching the teacher from myself to a stronger stranger jumped it to ~832**; cloning the top
team on its own deck reached **934**, and 3.09x the teacher data **938.8**.
**None of the eleven attempts after that transferred.**

---

## 3. Deck

**Alakazam ex + Dudunsparce (フーディンex + ノココッチ).
I did not build it. I used the teacher's 60 cards unchanged. That was deliberate.**

Once the strategy is "imitate a strong player's policy", the deck is a dependent variable of it.
Moving one card creates positions outside the teacher's data and breaks the premise of imitation,
so **I prioritised deck-policy consistency over novelty.** Twice I built clones that switched to
the dominant archetype; **both lost heavily** (−259.4 and −186.1 Elo, 5.5).

The 60 cards are **22 distinct cards**, **9 at 4 copies (36 cards, 60%)** — a build that repeats
one line regardless of the opening, which is what the policy needs:
**BC copies one stable win condition far better than a build full of branches.**

Across 61,994 games the field was Grimmsnarl
(Marnie's Grimmsnarl ex / マリィのオーロンゲex) **58.4%**, other 29.2% across 97 lists,
Alakazam (mine) 9.4%.
**All 20 teams playing Alakazam had a losing record against Grimmsnarl** (best 49.7%; the teacher
himself 49.2%), and field-weighted my deck ranked **9th of 10 at 45.4%** against 64.5% for Ogerpon.
**Even a perfect clone was capped at 49.2%: this deck was chosen because the teacher played it,
not because it was strong.**

And **one Grimmsnarl list accounted for 78.7% of that archetype across 175 teams**:
**deck construction here was finished, and the remaining difference lived on the policy side.**

---

## 4. What worked and what failed

Three things worked. **A better teacher** broke the 46% wall — though this later turned out to be
data volume, since at equal volume teacher purity is worth 0.3pp and the curve saturates near
5,000 games. **Greedy inference** was worth **+27.3pp, beating 14/14 opponents (z=9.14)**.
**Weight averaging** gives free gains within one basin, while independently trained models
collapse to **0.0%**. Ten further axes failed: four forms of RL, teacher pooling, larger BC
capacity, BC on won games only, forcing lethal, deck transplants and logit ensembling.

**The most expensive failures were defects in the instrument.**

| Defect | Damage |
|---|---|
| Opponent policy a month old | reported 86.0% where the ladder was **43.6%** |
| Opponent forfeits counted as my losses | identical conditions gave 81 / 86 / **15** / 87 / 81 / 81% |
| Silent initialisation bug | **inference ran on random weights**: 26.6% was really 61.9% |
| Distillation target weaker than the student | sole cause of ten null RL runs |
| The ladder's detection floor | byte-identical tars scatter with sd≈27 |

---

## 5. Discussion

### 5.1 What an improvement operator must be

Ten null RL runs had one cause, and it was neither hyperparameters nor reward design.
The policy target was the MCTS visit distribution at budget 10.

| Search budget | Win rate | Used for |
|---|---|---|
| **1** | **85.7%** | **every shipped submission** |
| 10 | 64.1% | **the RL training target** |
| 30 | 58.4% | local default |

**The target was 21.6pp weaker than the student.**

> **An improvement operator must be stronger than the current policy.
> You cannot distil from something weaker than yourself.**

That sentence explains three results. **BC worked** only because the teacher was stronger,
**RL was null** because the target was weaker, and **MCTS was harmful** because rollouts guessing
the opponent's deck, hand and prizes return **values computed over boards that never occur**.
**Once the policy is good enough, the bias search adds exceeds the error it removes.**

I once blamed a coarse value head, so I retested with a network whose value loss was **−56%**
better. The result fell from 64.0% to 38.0% (z≈−5.4): **the value head was not the bottleneck.**
It holds with the **same sign against all 14 opponents (z=9.14)**, so **it does not depend on a
particular matchup.** AlphaZero-style work assumes search improves a policy;
**that is an assumption to measure, not a given.**

### 5.2 Fidelity is not a proxy for strength

Assuming the BC ceiling was imitation accuracy, I separated capacity from epochs. Training CE −29%,
value loss −56%, teacher agreement +2.8pp (z≈4.3): **every fidelity metric improved and mirror win
rate moved 48.0% to 52.5%, i.e. not at all. The best arm on every metric was last in play.**

Controls showed why. **Agreement carries a floor set by the deck.** The champion agrees with its
teacher 78.5% of the time and **with a different team on the same deck 70.2%** — teacher-specific
signal is **only +8.3pp**. **About 70% of decisions are effectively forced**, so what decides games
is a fraction of the rest and an average metric barely weights it.
**The 325 Elo gap between teacher (1252.8) and clone (~914) is explained by neither imitation error
nor missing search.** Covariate shift is the suspect, but testing it needs online queries to the
teacher. I did not solve it.

### 5.3 Offline instruments have a half-life

The evaluation opponent was **built on 07-06**, and **its deck matched the current meta on 58 of 60
cards, so its age was invisible.** Against a live measurement of 43.6% (n=3,408) the simulator
reported **86.0%**. **I was losing to an opponent I believed I was beating.**
That invalidated "the champion is at the ceiling" and every conclusion resting on external agents.
The mechanism: **card lists are static, policies are not.**

> **Judge an evaluation opponent by when its policy was made, not by whether its deck matches.
> The instrument's half-life is set by the adaptation rate of the meta.**

I caught it only because I had **a reference using no simulator**: archetype-versus-archetype win
rates from replays (30,997 games). My own team appears in n=6 games, but **archetype pairs do not
need my team**, so tens of thousands become usable, and validity is self-checking — every mirror
came out at exactly 50.0%.

> **An instrument's validity cannot be checked from inside it. Keep one external reference.**

Rebuilt, the opponent reproduced the ladder to **0.7pp**, proving
**the limitation had always been data volume** (48% from 1,416 games, 55.7% from 6,925,
against 56.4% live): **supply, not algorithm.**

### 5.4 Stability under repeated matches, and the detection floor

**I measured stability directly by resubmitting byte-identical tars.**

| Tar (byte-identical) | n | Draws | Mean | sd |
|---|---|---|---|---|
| champion | 5 | 936.7 / 980.4 / 976.0 / 945.0 / 917.8 | **951.2** | 26.6 |
| soup | 3 | 938.0 / 973.5 / 921.1 | **944.2** | 26.7 |

**Identical contents scatter with sd≈27. Differences carry sd≈38, so one paired submission detects
only ~76 Elo at 2σ, and every ladder A/B I had run fell below that floor.**
What this closes is **the methodology**: calibrating an offline metric with one pair cannot work
when the effect judged (~+30 Elo) is below the resolution.

The same underestimate returned at the end: refreshing both slots 24.6h before the deadline gave
**846.1 / 817.3, 3.7σ below** six prior draws (mean 944.7), with the field flat.
**I had trusted an sd estimated from n=6.**

### 5.5 Offline evaluation cannot predict a deck switch

| Clone | Offline evaluation | Ladder |
|---|---|---|
| First | **58.1%** against the champion (n=2,400, 8.0σ) | **−259.4 Elo** |
| Second | **+5.1pp** field-weighted; archetype win rates reproduced to **0.7pp** | **−186.1 Elo** |

**The second was not a broken build.** It was the only clone reproducing live archetype win rates
to 0.7pp and it beat the champion on every offline metric. **It still lost 186 Elo.**
The first beats the second head-to-head at 53.5%: **the one that looked better lost by more.**
I re-verified it against the shipped artifact (weights, md5 and all 60 cards matching): a correctly
functioning agent.

My hypothesis, **stated as untested**: the archetype matrix measures the average over *all pilots*
of an archetype, so using it as my own expectation assumes I pilot at that average skill.
This clone agreed 66.3% with a team outside its teacher pool — it had learned **an average pilot** —
and on the destination deck **58.4% of games are mirrors**, exactly
**where a gap in policy quality shows up undiluted.**

Deck choice was worth 19.1pp, against +0.6pp per 2,000 extra games and 0pp for fidelity:
**the highest-leverage variable was the one I could not measure.**

> **When that happens, the right move is not to rebuild the metric.
> It is to decide whether to bet without measuring.**

### 5.6 Handling human intuition

The champion declines 29 of 36 available knockouts (**80%**). It looked like an obvious bug.
Before fixing it, I measured with controls.

| Condition | Win rate |
|---|---|
| Unmodified | **44.7%** |
| Search invoked, decision not overridden | 41.7% (ns) |
| **Lethal forced** | **28.3% (z=−3.33)** |

**Fixing it made things significantly worse, by −13.4pp. Invoking search was harmless; the damage
was concentrated in the override**, ruling out confounds such as computation cost.
The mechanism is in the rules: **attacking ends your turn**, so taking a knockout hands over the
initiative. **Positions where declining a knockout is correct really exist.**

> **Measure with a control before injecting a human prior into a policy as a hard constraint.**

---

## 6. Summary

| # | Claim | Evidence |
|---|---|---|
| 1 | An improvement operator must be stronger than the policy | target 21.6pp weaker |
| 2 | Under hidden information, search can degrade a good policy | +27.3pp, 14/14, z=9.14 |
| 3 | Imitation fidelity is not a proxy for play strength | agreement +2.8pp, no change |
| 4 | Opponent freshness cannot be judged by deck match | 58/60 match, 42.4pp off |
| 5 | An Elo ladder has a detection floor | sd≈27, 2σ floor ~76 Elo |
| 6 | Offline evaluation cannot predict a deck switch | 0.7pp reproduction, −186.1 Elo |
| 7 | An obvious human-visible mistake can be correct | forcing lethals cost −13.4pp |
| 8 | The meta had converged onto one list | 78.7% on identical 60 cards |

> **For two months the binding constraint was neither algorithms nor compute.
> It was being unable to notice that my own instruments were lying.
> And an instrument's lie always arrives in the shape of a good result.**
