# Brief

The **MOT test**, named after the Ministry of Transport that introduced it, is the roadworthiness check that most vehicles in Great Britain must pass every year once they are three years old.

## The client and the question

**The client is a fleet manager** who runs vehicles in Great Britain, of any kind the MOT covers, such as motorcycles, cars, vans and buses. The manager pays for their MOT tests, their repairs and their time off the road, and decides which parts are inspected before each vehicle's test.

> *Given what last year's tests found on a vehicle, how likely is each problem at its tests this year, and is acting on those probabilities worth it?*

We answer with a memo, not with software: we build and test the model such a product would run, measure how well it predicts for each vehicle against two simple rules, and say how a fleet manager would use it. The software a fleet would run it in is not part of this work.

## The question, as data

- **The prediction.** For each vehicle of every class, the probability of every **problem** in each of its slots of MOT tests that starts from 2019 to 2025, scored from 2020, estimated from the problems recorded in its previous slot, from the vehicle's details and from the time between the two slots. A slot is a run of tests no more than 20 weeks apart, such as a test and its retests, and belongs to the year of its first test. A problem is a defect together with the way it was recorded: as a failure, a failure put right at the testing station, a minor defect or an advisory.
- **Where it goes wrong.** The prediction read by defect group, the 452 groups of parts, defined in the publisher's lookup tables, that the defects we keep fall under, such as shock absorbers or position lamps.

## How a fleet manager would use it

Our model gives each vehicle the probability that its MOT test records each problem. For a given problem, a vehicle the model flags, when its probability reaches the cut-off below, is a positive and one it does not flag a negative; the flag is true or false according to whether the test then records the problem.

For a given vehicle and problem, let $p$ be the probability our model assigns to the problem, $t$ the cost of checking the vehicle for it before the test, and $m$ the cost of leaving it to the MOT test to find: the retest and the extra time off the road that a failed test brings. The test itself and the repair are paid either way, so they are in neither.

Checking costs $t$ whatever it finds. When the part turns out fine, the check is a false positive, and that money is wasted. Not checking costs $m$ when the problem is there, a false negative, which happens with probability $p$, so $p\,m$ would be the average cost of not checking. Checking pays when $t \le p\,m$, that is when

$$
p \;\ge\; p_{\text{crit}} = \frac{t}{m}
$$

The memo sets out what goes into $t$ and $m$.

## Data and provenance

The anonymised MOT tests and results of Great Britain, published by the Department for Transport up to 2023 and by the Driver and Vehicle Standards Agency from 2023, under the Open Government Licence: every test of every vehicle class from 20 May 2018, when the current defect codes came into force, to 31 December 2025, with the defects and advisories each test recorded and the lookup tables that decode them. The files, their units and every decision taken on them are in [`DATA.md`](DATA.md).

## Scope

- **We estimate the probability of problems, not of test results, and not which repair to make.** Once a check finds a problem, what to repair is for the fleet's mechanics to decide; the data hold no repairs, so they could not say which repair lowers the risk either.
- **Costs are not in the data, and they vary with the fleet, its garages and where its vehicles run**, so each fleet supplies its own $t$ and $m$, and the memo gives the method for the cut-off.
- **Fleet vehicles cannot be told apart** from others in the data, so the model is evaluated on all vehicles.
- **One kind of model**, logistic regression, measured against two simple rules: the problems of the vehicle's previous slot again, and the most frequent problems of the year before.

## Deliverables

- **`REPORT.md`, the memo:** the recommendation, the evidence for it, the method for setting the cut-off, and the caveats.
- **`README.md`:** what the project found, and how to run it.
- **`DATA.md`:** every source, unit and decision about the data.
- **The repository:** the code that fetches the files, builds the tables, and trains and scores the model, regenerating every number in the memo.
