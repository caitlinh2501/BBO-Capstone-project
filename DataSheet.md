# Datasheet: Eight-Function Black-Box Optimisation Project

This datasheet covers all eight functions in my black-box optimisation (BBO) capstone. I have kept them together because the data collection process and main modelling approach were similar, but I have included the differences between functions where they mattered.

## Function overview

The challenge was to find input values that maximised eight unknown functions. I couldn't see the equations or calculate gradients. Instead, I started with a small set of input/output pairs for each function, then submitted one new input per function each week and used the returned scores to decide what to try next.

The input values were continuous and restricted to `[0, 1]`. Each function returned one numerical objective value, and **higher was better**, even when the scores were negative. The functions had different dimensions and output scales, so I assessed progress separately rather than comparing scores between functions.

| Function | Input dimensions | Initial input shape | Initial output shape | Input shape entering Week 13 | Output shape entering Week 13 |
|---|---:|---|---|---|---|
| 1 | 2 | `(10, 2)` | `(10,)` | `(22, 2)` | `(22,)` |
| 2 | 2 | `(10, 2)` | `(10,)` | `(22, 2)` | `(22,)` |
| 3 | 3 | `(15, 3)` | `(15,)` | `(27, 3)` | `(27,)` |
| 4 | 4 | `(30, 4)` | `(30,)` | `(42, 4)` | `(42,)` |
| 5 | 4 | `(20, 4)` | `(20,)` | `(32, 4)` | `(32,)` |
| 6 | 5 | `(20, 5)` | `(20,)` | `(32, 5)` | `(32,)` |
| 7 | 6 | `(30, 6)` | `(30,)` | `(42, 6)` | `(42,)` |
| 8 | 8 | `(40, 8)` | `(40,)` | `(52, 8)` | `(52,)` |

**What the functions represent:** The repository does not contain confirmed real-world scenarios, descriptive names or measurement units for the eight functions. I have therefore referred to them by number and treated each output as the objective score returned by the course portal. I don't want to assign a real-world meaning to a score without that information.

## Nature of the data

The original observations were supplied as NumPy arrays. The inputs are two-dimensional arrays with one row per query and one column per input variable; the outputs are one-dimensional arrays with the corresponding scores. The starting data are in `week1/`. The later folders contain cumulative observations saved under filenames such as `initial_inputs.npy` and `initial_outputs.npy`, even though they no longer contain only the initial data.

Each function gained **12 new observations between its starting data and the data used to choose Week 13's query**. I then submitted one final query per function in Week 13. That means the final challenge produced 13 additional observed scores per function, although the `week13/` NumPy arrays are snapshots taken **before** the last results came back. Those final eight inputs and outputs are recorded in the results section below; they have not been silently added to the archived arrays.

The data became more focused as I went along. Earlier queries included broader exploration, while later ones were mostly around areas that had already returned good scores. This was useful for refining those areas, but it also means the dataset is not an even sample of the full search space, especially for Functions 7 and 8.

I did not have enough repeated evaluations at identical inputs to determine whether the underlying functions were noisy. My GP included a noise term, but that could also account for modelling errors. Some local results looked relatively smooth, particularly near the strongest region of Function 8, while other functions changed noticeably after a small input adjustment. I wouldn't label any function as definitely unimodal or multimodal based on this amount of data.

## My optimisation strategy

I mainly used **Bayesian optimisation with Gaussian Process (GP) regression**, fitting a separate GP for each function. This was useful because evaluations were limited: the model gave me both a predicted score for an untested point and an estimate of how uncertain that prediction was.

I compared Expected Improvement (EI), Upper Confidence Bound (UCB) and the highest predicted GP mean. I also sampled candidate points more widely or near the current best inputs. Earlier on I was happier to explore uncertain areas. Towards the end, I became more careful about giving up a query on a point that looked interesting to the model but did not match what previous results had shown.

My later notebooks used local trust regions, comparisons along individual coordinates, checks for points that were too close to earlier queries, and comparisons between predicted and actual scores. The final selections weren't all made in the same way: sometimes I followed the GP's highest mean, and sometimes I chose a simpler local move because the recent data supported it more strongly.

I also tried an alternative neural-network approach during the project. It was a useful experiment, but with so few observations I found the GP more useful, particularly because I could inspect its uncertainty. I kept the weekly notebooks so it is possible to see those changes rather than only the final method.

## Data handling and preprocessing

- **Inputs:** All values were already between `0` and `1`, so I did not need to rescale the input dimensions or encode categorical variables.
- **Main surrogate:** The later notebooks used a GP with a constant-amplitude **ARD Matérn 5/2 kernel** and a white-noise term. ARD allowed different fitted lengthscales for different input dimensions.
- **Hyperparameters:** I fitted kernel amplitude, lengthscales and noise-related parameters through the GP's log marginal likelihood, using optimiser restarts in the later notebooks. The Matérn smoothness setting was `nu=2.5`.
- **Outputs:** Most functions were modelled using their original scores with GP output normalisation. Function 1 needed extra care because the score scale varied so much; the final approach used positive linear scaling rather than the earlier log-transformation experiment. For Function 5, I manually standardised the large output values before fitting and converted predictions back to their original scale.
- **Unusual results:** I generally kept unexpected observations instead of treating them as bad data. If an actual score differed from the GP prediction, I used that as a reason to check the model and the search direction more carefully.

## Weekly iteration and learning

The most helpful queries were often the ones that tested whether a small change in direction was genuinely improving a function. I became more interested in comparing nearby observations than in simply accepting whichever acquisition function gave the highest score. The notes below describe what I observed in the sampled areas; they don't prove where the true global optimum is.

| Function | What I learned and how I chose the final query | What I'd change if I started again |
|---|---|---|
| **1** | The scores were extremely sensitive to input changes and the scale made GP predictions difficult to judge. In Week 12, moving `x1` upwards while keeping `x2` fixed performed worse. For the final query I moved `x1` down to `0.669556`, keeping `x2 = 0.734173`. **This produced a new best of `7.80718e-11`.** | I would deal with the unusual output scale earlier and check prediction errors on the original scale throughout. |
| **2** | The strongest area I found was along the boundary `x2 = 1`. Moving `x1` higher in Week 12 was not an improvement. I kept `x2 = 1` and tried a lower `x1` in Week 13, which returned `0.634828` but did not beat the earlier best of `0.679663`. | I would include boundary points earlier and make more controlled comparisons along `x1`. |
| **3** | The best observations were negative but quite close to zero. My final search compared individual-coordinate moves and a direction between earlier promising points. I tried `[0.376500, 0.525082, 0.442594]`, but the returned score (`-0.012963`) was worse than the incumbent (`-0.001268`). | I would take smaller steps and test one direction at a time rather than combining several changes in the last round. |
| **4** | Earlier results showed that reducing `x2` from `0.412010` to `0.401083` hurt performance. I then checked other directions around the best point and chose a small reduction in `x4`, from `0.427725` to `0.416700`. **This improved the best score from `0.670498` to `0.730122`.** | I would start the single-coordinate checks earlier because they were especially useful here. |
| **5** | The best observed point was the boundary corner `[1, 1, 1, 1]`. Lowering one coordinate near that corner had already reduced the score. I tried lowering two coordinates slightly in Week 13 and got `8284.891841`, below the best of `8662.4825`. The corner was still my strongest observed point, although that doesn't prove the whole function is monotonic. | I would examine the corners and boundary behaviour earlier and avoid overinterpreting nearby GP predictions. |
| **6** | Week 12's move towards lower `x3` and `x4` had performed worse. Rather than follow another larger GP move, I held those coordinates at the incumbent and adjusted `x1` slightly, to `0.452930`. **That improved the best score from `-0.195176` to `-0.136427`.** | I would use more local directional checks earlier and be quicker to challenge a GP suggestion after a poor prediction. |
| **7** | Week 12 had already found a new best (`3.077057`), and the GP's local predictions had been reasonably consistent with recent observations. I followed its highest predicted mean in Week 13 and got **another new best of `3.101315`**. | I would allow more broad exploration earlier in the six-dimensional space, then use the local GP once there was enough supporting evidence. |
| **8** | I had found a cluster of points scoring very close to `10`. The recent GP prediction errors were relatively small, so I made another nearby query in Week 13. It returned `9.998272`, just below the earlier best of `9.999646`. The strong local results were encouraging, but they didn't establish a global maximum. | I would explore more widely earlier, because in eight dimensions it is easy to miss other strong regions. |

I also noticed that a good predicted mean isn't always a good reason on its own to submit a point. Function 7 is an example of the GP helping in the final round; Function 3 is a good reminder that the next actual result can still be quite different from the prediction. The comparison between those two functions changed how much confidence I put in model recommendations.

## Performance and results

All the numbers below are **actual scores returned by the course portal**, not GP predictions. The first table records each Week 13 submission and its result. Higher is better in every case.

### Week 13: final submitted queries and returned values

| Function | Final query (input vector) | Returned Week 13 output |
|---|---|---:|
| **1** | `[0.669556, 0.734173]` | **`7.807182592694927e-11`** |
| **2** | `[0.693612, 1.000000]` | `0.6348281857464395` |
| **3** | `[0.376500, 0.525082, 0.442594]` | `-0.012962526170438072` |
| **4** | `[0.361985, 0.412010, 0.421437, 0.416700]` | **`0.7301217953890853`** |
| **5** | `[1.000000, 1.000000, 0.990000, 0.990000]` | `8284.891841122184` |
| **6** | `[0.452930, 0.409333, 0.631830, 0.739800, 0.129381]` | **`-0.1364265561489056`** |
| **7** | `[0.165154, 0.250080, 0.549909, 0.253314, 0.284687, 0.676488]` | **`3.101315084831645`** |
| **8** | `[0.095001, 0.139292, 0.119306, 0.147069, 0.819451, 0.469514, 0.203887, 0.575004]` | `9.9982724186289` |

**Four of the eight Week 13 queries gave me new best scores:** Functions 1, 4, 6 and 7. The other four were still useful for checking whether the model's suggestions were reliable, but they did not replace the previous best points.

### Best observed results over the whole project

This table uses the highest **observed** score for each function, including the final round where it improved the record. It is not just a list of the Week 13 outputs.

| Function | Best observed output | Input that produced it | First achieved |
|---|---:|---|---|
| **1** | `7.807182592694927e-11` | `[0.669556, 0.734173]` | Week 13 |
| **2** | `0.679662733033218` | `[0.711177, 1.000000]` | Before Week 13 |
| **3** | `-0.00126784051040226` | `[0.356112, 0.538429, 0.432459]` | Before Week 13 |
| **4** | `0.7301217953890853` | `[0.361985, 0.412010, 0.421437, 0.416700]` | Week 13 |
| **5** | `8662.4825` | `[1.000000, 1.000000, 1.000000, 1.000000]` | Before Week 13 |
| **6** | `-0.1364265561489056` | `[0.452930, 0.409333, 0.631830, 0.739800, 0.129381]` | Week 13 |
| **7** | `3.101315084831645` | `[0.165154, 0.250080, 0.549909, 0.253314, 0.284687, 0.676488]` | Week 13 |
| **8** | `9.9996456622444` | `[0.107323, 0.149041, 0.129094, 0.157441, 0.815313, 0.493175, 0.201585, 0.586041]` | Before Week 13 |

The best observed score improved compared with the original starting data for all eight functions. I was pleased with the final week in particular because the local, relatively cautious decisions worked for Functions 4 and 6, while the GP's highest-mean suggestion worked for Function 7.

**How close are these to the true maximum?** I can't know for certain. Function 5 gave consistent evidence around the upper corner, and Function 8 produced a number of very high nearby scores, but that isn't proof there is no better point somewhere else. There is even less coverage in six or eight dimensions. GP variance was helpful for deciding where to search, but its uncertainty estimates weren't always perfectly calibrated, so I wouldn't use them to claim a guaranteed optimum.

The project met the general aim of improving the starting results within a limited number of evaluations. Without confirmed real-world descriptions of the individual functions, I can't make a stronger claim about whether a result meets any domain-specific target.

## Ethical, practical and general considerations

The idea behind this task could apply to real situations where testing every possible option is expensive, such as tuning a process or deciding which experiments to run. I have also been thinking about how a similar approach might help with prioritising QA testing at work, although real QA decisions would need more than a single numerical score and human judgement would still matter.

There are limitations to learning from synthetic black-box functions. I didn't have to deal with real-world constraints, changing conditions, safety considerations or competing objectives. In practice, maximising one number might not be the right decision if the cost or risk of achieving that score is too high.

For more expensive or higher-stakes applications I would want more checks on data quality, uncertainty calibration and feasibility before trusting a recommendation. GP modelling can also become harder as the number of observations and dimensions grows. One lesson I would carry forward is to keep the model prediction separate from the actual measured result and to record why each decision was made.

## Data sources, access and maintenance

The initial data and subsequent function evaluations came from the Imperial College London Machine Learning and Artificial Intelligence capstone challenge. The notebooks and `.npy` arrays in this repository document the modelling work and the data available at the time of each decision; the actual black-box evaluator is not included.

The Week 13 notebooks and arrays were saved before the final submissions. The eight final returned values are documented above so the project outcome is complete without rewriting those historical modelling snapshots. The dataset is intended for capstone assessment and reproducibility of the analysis, subject to the course's rules on sharing its data.
