# Which parts to check before a vehicle's annual test

The **MOT test**, named after the Ministry of Transport that introduced it, is the roadworthiness check that most vehicles in Great Britain must pass every year once they are three years old.

![Left: the money saved at p_crit = 0.1 from 2020 to 2025, rising from 10.6% to 13.2% for our model, from 5.0% to 8.7% for Repeat, and falling from 4.3% to 3.2% for Most popular. Right: the six groups of defects that save the most in 2025, engine oil leaks to registration plates, 33.8% to 23.9%, and the six that save the least, between 0.0% and −0.6%.](figures/money_saved.png)

## Summary of findings

- **Our model would save a fleet manager 13.2% of what its MOT problems cost, and up to 34% on some groups of defects, such as engine oil leaks and rigid brake pipes**, where a missed problem costs ten times as much as a check. It beats simple rules such as re-checking whatever the last slot of tests found, which saves 8.7%, and its saving grows each year it is retrained, from 10.6% in 2020. Test year 2025, every vehicle in Great Britain.
  **These figures are probably understated**, for the reasons under [Caveats](#caveats).
- **The saving stops where a missed problem costs two checks, a limit of the data rather than of the model**, because almost no vehicle in the MOT records is more likely than not to have any given problem, which is where a check would start to pay (see [Approach](#approach)). Where a miss costs four checks, the model still saves 2.9%, while both simple rules lose money.
- **The model stays predictive and improves as it is retrained each year.** Its Brier skill, how much better its probabilities are than each problem's average rate, is above 0 in every test year from 2020 to 2025 and rises from 0.028 to 0.047.
- **A simpler model, with rare makes and models pooled, is not better, so we keep the first.** Under a rule fixed before scoring, it had to lead in every year; it saves 0.04 to 0.18 points less money in every year, and its skill falls behind from 2023.

## Summary of recommendations

- **Set a cut-off for each problem from the fleet's own costs, $p_{\text{crit}} = t/m$, and check a vehicle for that problem when the model's probability reaches it.** $t$ is what the check costs, $m$ what leaving the problem to the test costs.
- **Use the model for the problems whose miss costs at least four checks.** There it saves money and the simple rules lose it.
- **Keep checking by routine the parts the model does not flag**, such as wipers, washers and anti-lock brakes, rather than reading no flag as no problem.
- **Record what each check finds**, so that the model can still be measured once the fleet acts on its flags.

## Approach

**The client is a fleet manager** who runs vehicles in Great Britain, of any kind the MOT covers, such as motorcycles, cars, vans and buses. The manager pays for their MOT tests, their repairs and their time off the road, and decides which parts are inspected before each vehicle's test. The question: given what a vehicle's last tests found, how likely is each problem at its tests this year, and is acting on those probabilities worth it?

**The data** is every MOT test in Great Britain from 20 May 2018, when the current defect codes came into force, to the end of 2025: 308,127,375 tests with the defects and advisories each recorded, published by the Department for Transport and the Driver and Vehicle Standards Agency. Every source and decision is in [`DATA.md`](DATA.md).

**A problem** is a defect together with the way the test recorded it: as a failure, a failure put right at the testing station, a minor defect or an advisory. There are 4,821 problems that can be recorded. A vehicle's **slot** is a run of its tests no more than 20 weeks apart, such as a test and its retests, and the probability of each problem in a slot is estimated from the slot before it: the problems recorded there, their groups, the time between the two, and the vehicle's details. A slot in 2025 records 1.85 problems on average, and 23.0% of slots record at least one failure, so that the vehicle fails its test and has to come back for a retest.

**The model** is one logistic regression per problem, as large as its training data can carry, about ten problems recorded per weight: the 64 commonest problems get every input, 2,740 get a small model of 15 weights, and the 2,017 rarest get their average rate. **The probability** we give each vehicle and problem is the chance that its next MOT test records the problem. Each test year from 2020 to 2025 is predicted by a model trained only on the years before it.

**Two simple rules** are what the model has to beat: **Repeat**, re-check every problem the last slot recorded, and **Most popular**, check every vehicle for the commonest problem of the year before, a tyre advisory.

**We measure what matters to the manager.** Checking pays when $p\,m \ge t$, that is when $p \ge p_{\text{crit}} = t/m$. The **money saved** is the share of what the fleet's problems cost, if every one were left to the test, that checking the flagged vehicles saves:

$$S = \frac{m\,TP - t\,(TP + FP)}{m\,(TP + FN)} = \text{recall}\left(1 - \frac{p_{\text{crit}}}{\text{precision}}\right)$$

where a true positive is a flag on a problem the test then records, a false positive a flag on one it does not, and a false negative a problem it records unflagged. The **Brier skill**, $1 - \text{BS}/\text{BS}_{\text{base}}$, compares the mean of $(p - y)^2$ with that of each problem's average rate, and judges the probabilities themselves, whatever the cut-off.

## Findings

### The prediction

**The model saves money wherever a missed problem costs at least four checks, and the simple rules only where it costs ten.** Money saved in 2025, pooled over every vehicle and problem:

| A missed problem costs | $p_{\text{crit}}$ | Model | Repeat | Most popular |
|---|---|---|---|---|
| 10 checks | 0.1 | **13.2%** | 8.7% | 3.2% |
| 4 checks | 0.25 | **2.9%** | −4.8% | −4.6% |
| 2 checks | 0.5 | −0.1% | −27.3% | −17.8% |

At 0.1 the model's flags find a problem as often as Repeat's, 19.5% against 19.6% of the time, but find half as many problems again, 27.1% of them against 17.7%. Repeat's 17.7% is the share of 2025's problems that the last slot had already recorded.

**The saving comes mostly from the commonest problems.** At 0.1 the 64 problems with a full model hold 73.3% of the problems and save 16.6% of their cost, against Repeat's 11.7% on the same problems; the 2,740 with a small model save 3.8%, against 0.5%; the 2,017 with their average rate are never flagged.

**Where a missed problem costs two checks, nothing can be saved, because almost no vehicle is more likely than not to have any given problem.** A check pays at $p_{\text{crit}} = 0.5$ only on a vehicle whose problem is there more than half the time. Recalibrated on the year before, the model's flags above one half hold 0.3% to 0.7% of each year's problems from 2022 to 2025 and save 0.01% to 0.04%: they break even, as true probabilities must at the cut-off. This is what the records allow, not a fault of the model: two vehicles with the same history, one recording the problem and the other not, cannot be told apart from them. The model's own small loss at 0.5 comes from overconfidence there: its flags promise about 60% and deliver 47% to 48% in every year. At 0.1 and 0.25 what it promises it delivers, within 5 points.

**The model stays predictive and improves as each year adds training data.**

| Test year | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 |
|---|---|---|---|---|---|---|
| Brier skill | 0.028 | 0.034 | 0.038 | 0.041 | 0.043 | 0.047 |
| Money saved at 0.1 | 10.6% | 11.2% | 11.5% | 12.0% | 12.6% | 13.2% |

Retrained on outcomes shuffled among vehicles, which breaks any real link, the model scores a skill of 0.0004 at most and beats Most popular in no year: what it learns comes from the link between a vehicle's history and its problems, not from the method.

### Where it goes wrong

**The model saves most on problems that a vehicle's history shows coming, and almost nothing on those it barely flags.** We rank the 174 groups of defects with at least 100 flags and 100 problems in 2025, which hold 96.1% of the problems, by the money saved at $p_{\text{crit}} = 0.1$; 167 of them save money.

| Group | Problems | Flags right | Problems flagged | Money saved |
|---|---|---|---|---|
| Engine oil leaks | 1,772,120 | 28.7% | 51.9% | **33.8%** |
| Rigid brake pipes | 1,541,267 | 23.6% | 57.0% | **32.8%** |
| Tyre tread depth | 6,514,159 | 16.9% | 75.9% | **31.1%** |
| Wipers | 1,014,015 | 7.0% | 0.5% | −0.2% |
| Anti-lock braking system | 149,934 | 9.2% | 6.5% | −0.6% |
| Brake rods | 396 | 8.7% | 3.8% | −0.6% |

For wipers, with a million problems a year, the model flags almost none, so the fleet gains nothing from it and loses nothing either. A further 199 groups with at least 100 problems are never flagged; they hold 3.4% of the problems.

### A simpler model, tested and not kept

**Most of the first model's inputs are makes and models too rare to learn from, so we tested a model that pools them, and it is not better.** In the model for test year 2025, 89,476 makes and models each have an input, the median one with 5 training rows. The second model gives one only to those with 10,000 rows or more and pools the rest by make, which cuts the inputs from 114,060 to 25,363 and gives 150 problems a full model instead of 64. Before scoring, we fixed the rule: it would replace the first only if, in every test year, its Brier skill and its money saved at 0.1 and at 0.25 were higher, and its shuffled check passed.

| Test year | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 |
|---|---|---|---|---|---|---|
| Brier skill, first | 0.0280 | 0.0342 | 0.0381 | **0.0411** | **0.0432** | **0.0469** |
| Brier skill, second | **0.0296** | **0.0351** | **0.0384** | 0.0410 | 0.0428 | 0.0466 |
| Money saved at 0.1, first | **10.62%** | **11.23%** | **11.52%** | **12.04%** | **12.61%** | **13.18%** |
| Money saved at 0.1, second | 10.50% | 11.08% | 11.37% | 11.85% | 12.44% | 13.07% |

It saves less in every year, at 0.25 as at 0.1, so we keep the first. It loses on the problems it adds: their full models beat their small ones to 2022 and fall behind from 2023, and their fits, each starting from the year before, stop after a few iterations. Fitting two of them further closes about half and over a third of the gap; a stronger penalty adds almost nothing, so too many weights is not the cause. The rest is untested, as set out in `DATA.md` under "The second model".

## Recommendation

**Set a cut-off for each problem from the fleet's own costs, and check a vehicle for a problem when the model's probability reaches it.** For each problem, $t$ is what checking the part costs, the mechanic's time and the vehicle's time off the road for the check, and $m$ what leaving it to the test costs: the retest and the extra time off the road that a failed test brings. The test itself and the repair are paid either way, so they are in neither. The cut-off is $p_{\text{crit}} = t/m$.

**A cut-off for each problem saves money where one cut-off for all would not.** Rigid brake pipes save 32.8% of their cost at 0.1, 15.9% at 0.25 and still 1.6% at 0.5, while tyre tread saves 31.1% at 0.1 but 0.7% at 0.25 and nothing at 0.5. A fleet for which a missed brake pipe costs two checks and a missed tyre ten checks the brake pipes the model puts above one half and the tyres it puts above a tenth, and saves on both; with a single cut-off of 0.5 for every problem, the model as a whole saves nothing.

**Use the model for the problems whose miss costs at least four checks,** where it saves money and the simple rules lose it. Where a miss costs less, check only the problems whose own saving stays positive, such as brake pipes.

**Keep checking by routine the parts the model does not flag.** Wipers, washers and anti-lock brakes go almost unflagged, as do 199 groups that hold 3.4% of the problems: no flag there means the model cannot tell, not that the part is fine.

**Record what each check finds, in the MOT's terms.** A check that repairs a problem removes it from the test's record, so the test alone would score a right flag as a wasted check; the check's own record keeps the count. To retrain the model on the fleet's own data, also leave a random tenth of the vehicles to the test alone: their records stay as in ours, at the cost of a tenth of the saving.

## Caveats

- **Our figures are probably conservative, because a flag counts as right only when the MOT test recorded the problem.**
  A flag can be right and still count against us. Suppose we flag a van's brake pipes, and the owner replaces them the week before its test, or the tester does not write them down: the test records nothing, and we score the flag as a wasted check. Every such case lowers the precision and the money saved we report. The data holds no repairs, so we cannot count these cases, and we report the figures as measured. A tester recording a problem that is not there would push the other way; we expect it to be rarer.
- **The data holds no costs, repairs, owners or accidents, and fleet vehicles cannot be told apart from others.**
  The model is trained and scored on every vehicle in Great Britain. A fleet's vehicles, serviced on a schedule, may record fewer problems than the average, and its own $t$ and $m$ decide which of the results above apply to it.
- **The later years' scores are probably slightly understated, because we trained the largest models to scikit-learn's default tolerance**, the point at which a fit counts as finished.
  A tighter tolerance would have taken five to six days of computing for the last four years, against hours. With the default, and each year's fit starting from the year before, the largest fits stop after a few iterations from 2023, a median of 5.5 in 2025. Fitted further, two of the second model's problems gained 0.005 and 0.006 of skill; the first model's were not refitted.
