# Model Card: Bayesian Black-Box Optimisation Approach

## Overview

**Model name:** Adaptive Gaussian Process Bayesian Optimisation  
**Model type:** Sequential black-box optimisation approach using Gaussian Process surrogate models  
**Version:** Final capstone version

This approach was developed to optimise eight unknown black-box objective functions under a limited sequential query budget.

The method uses Gaussian Process regression as a surrogate model for each objective function. Candidate points are evaluated using a combination of posterior mean predictions, Expected Improvement (EI), Upper Confidence Bound (UCB), local trust-region searches, and diagnostic checks on previous prediction accuracy.

The approach is adaptive rather than fixed. The same basic Gaussian Process framework is used throughout, but the balance between exploration and exploitation changes depending on the behaviour of each function and the reliability of previous predictions.

---

## Intended use

The approach is intended for sequential optimisation problems where:

- the objective function is expensive or restricted to query;
- gradients are unavailable;
- only a limited number of evaluations can be made;
- the input domain is bounded;
- previously observed data can be used to guide future queries;
- there is value in balancing exploitation of strong regions with exploration of uncertain ones.

In this capstone, the task was to maximise eight independent black-box functions with different input dimensionalities.

The approach is particularly suitable when the objective is reasonably smooth locally and a Gaussian Process can provide a useful approximation of the response surface.

### Uses that should be avoided

The approach should not be relied on without modification when:

- the objective function is highly discontinuous;
- the search space is extremely high-dimensional relative to the available data;
- the function changes over time;
- there are strong categorical or discrete variables that are not represented appropriately;
- uncertainty estimates from the Gaussian Process are assumed to be perfectly calibrated;
- a global optimum must be guaranteed;
- the cost of a poor query is safety-critical or otherwise unacceptable.

The optimisation procedure should also not be interpreted as proving that the best observed point is the true global maximum.

---

## Details

### Core modelling approach

Each of the eight functions was modelled separately using Gaussian Process regression.

The main kernel used was an ARD Matérn 5/2 kernel with a constant scale term.

Automatic Relevance Determination (ARD) was used by fitting a separate lengthscale for each input dimension.

Shorter estimated lengthscales were interpreted as indicating that the fitted GP expected faster variation along that dimension. However, they were not treated as definitive feature-importance measures.

Kernel hyperparameters were estimated by maximising the Gaussian Process log marginal likelihood, using multiple optimiser restarts.

---

## Optimisation strategy

The optimisation approach evolved throughout the project rather than applying exactly the same acquisition rule in every round.

### Early rounds

In the earlier rounds, there was relatively little information about the objective functions.

The main goal was therefore to balance:

- exploration of uncertain parts of the search space;
- exploitation of regions with high predicted objective values.

Candidate pools were generated across relatively broad areas of the domain, and acquisition functions such as Expected Improvement and Upper Confidence Bound were used to identify potentially useful queries.

At this stage, the Gaussian Process models were based on limited data, so predictions were treated with greater uncertainty.

---

### Middle rounds

As more observations became available, several functions began to show stable high-performing regions.

The optimisation process increasingly concentrated candidate generation around the best observed points while still retaining broader candidate pools for comparison.

Candidate pools typically included:

- local candidates near the current incumbent;
- wider candidates around the same region;
- global random candidates.

This allowed the optimisation process to distinguish between a strong local opportunity and an acquisition function that was simply attracted to a highly uncertain region.

Expected Improvement, posterior mean and UCB were compared rather than using any one of them automatically.

---

### Later rounds

In the later rounds, the strategy became more calibration-aware.

For each new observation, the previous Gaussian Process prediction was compared with the actual returned objective value.

A small residual provided some support for continuing to trust local GP rankings.

A very large residual indicated that the GP uncertainty had not fully represented the behaviour of the function. In those cases, subsequent searches were made more conservative.

For example, some functions produced realised values several predicted standard deviations away from the previous mean. After these events, the optimisation process reduced the emphasis placed on exploratory acquisition functions and concentrated more tightly around observed high-performing points.

---

## Trust-region strategy

For functions where repeated local optimisation was appropriate, a trust region was constructed around the actual best observed input.

The trust-region size was informed by:

- ARD lengthscales;
- the distance to existing observations;
- previous prediction calibration;
- the dimensionality of the function.

Candidate points were then generated inside this restricted region.

A near-duplicate filter was used to prevent repeatedly querying inputs that were effectively identical to previous observations.

The trust region was not automatically expanded simply because the best candidate lay near its boundary.

Boundary behaviour was instead treated as a diagnostic. If the model repeatedly pushed toward a boundary but previous extrapolation had performed poorly, the search could remain restricted.

---

## Acquisition functions

Three main decision signals were used.

### Highest posterior mean

The posterior mean was used as the strongest exploitation criterion

This became particularly important in later rounds when a reliable high-performing basin had already been identified.

---

### Expected Improvement

Expected Improvement was used to assess the probability and magnitude of improvement relative to the best observed value.

EI was useful for identifying potentially valuable exploratory queries.

However, it sometimes preferred points with a lower predicted mean but much higher uncertainty.

For this reason, a high EI value was not automatically treated as evidence that a candidate should be selected.

---

### Upper Confidence Bound

UCB was evaluated using several exploration weights.

Rather than selecting a single fixed value of \(\beta\), multiple values were compared.

Agreement between low- and moderate-\(\beta\) UCB candidates and the highest posterior mean was treated as evidence of a stable local recommendation.

High-\(\beta\) solutions were treated more cautiously when they moved substantially farther away from the best observed region.

---

## Decision process

The final query for each function was selected using several pieces of evidence rather than one numerical score.

The main decision factors were:

1. posterior mean;
2. predictive uncertainty;
3. Expected Improvement;
4. UCB behaviour across different exploration weights;
5. distance from the best observed point;
6. whether the candidate was near a domain or trust-region boundary;
7. agreement between different acquisition criteria;
8. calibration of previous GP predictions;
9. observed performance of recent nearby queries.

This decision process was deliberately adaptive.

For functions with good recent calibration, the optimisation procedure was more willing to follow the surrogate model.

For functions where the GP had recently made a large prediction error, the strategy became more conservative.

---

## Function-specific preprocessing

Most functions were modelled using their original observed objective values with internal output normalisation in the Gaussian Process.

Two functions required additional treatment.

### Function 1

Function 1 contained objective values extremely close to zero near the optimum, while other observations were much more negative.

A positive linear scaling was used

This transformation preserves the ordering of the original objective values and therefore preserves the maximisation problem.

An earlier nonlinear transformation was not retained because it changed the optimisation objective.

For Function 1, the GP was therefore treated primarily as a local ranking model rather than as a reliable predictor of absolute objective magnitude.

---

### Function 5

Function 5 had objective values on a much larger numerical scale than the other functions.

Its outputs were manually standardised.

The GP was fitted to the standardised values and predictions were converted back to the original scale for interpretation.

---

## Performance

Performance was assessed separately for each of the eight functions.

The primary optimisation metric was the **best observed objective value**, because the task was to maximise each unknown function.

Additional diagnostic metrics included:

- improvement over previous rounds;
- predicted mean at the submitted query;
- predicted standard deviation;
- realised prediction error;
- standardised prediction residual;
- distance between the submitted candidate and the current incumbent;
- agreement between EI, posterior mean and UCB;
- whether a candidate lay at a search boundary.

Performance was therefore evaluated using both optimisation success and model reliability.

The approach successfully identified stable high-performing regions for several functions.

For some functions, later queries remained very close to the best observed values, indicating convergence around a strong basin.

Other functions were more difficult. In particular, several rounds demonstrated that a Gaussian Process could become locally overconfident and produce prediction errors much larger than its reported uncertainty.

These failures were useful in shaping the later strategy because they motivated tighter trust regions and greater emphasis on realised calibration.

The optimisation results should therefore not be interpreted only in terms of whether every new query improved the incumbent. A sequential optimisation query can still provide useful information by revealing that an apparently promising direction is less valuable than predicted.

---

## Strengths

The main strengths of the approach are its adaptability and transparency.

It does not depend on a single acquisition rule.

Instead, it combines several pieces of model evidence and adjusts its behaviour when the surrogate model performs poorly.

Other strengths include:

- explicit modelling of uncertainty;
- dimension-specific ARD lengthscales;
- preservation of the true maximisation objective during preprocessing;
- systematic candidate generation;
- near-duplicate filtering;
- calibration checks using realised observations;
- reproducible random seeds;
- use of the actual incumbent as the centre of local search;
- explicit comparison between exploration and exploitation.

The approach also records both successful and unsuccessful predictions rather than only reporting final best values.

---

## Assumptions and limitations

The most important assumption is that the unknown functions have enough local smoothness for a Matérn Gaussian Process to provide useful rankings.

This may not hold if a function contains:

- discontinuities;
- sharp local changes;
- narrow isolated optima;
- strong interactions that are poorly represented by the observed data.

The optimisation method is also limited by the small number of observations relative to the size of the continuous search spaces.

This is particularly important for the higher-dimensional functions.

As optimisation progressed, later observations became increasingly concentrated around promising regions. This introduces adaptive sampling bias and means that large areas of the search domain remain poorly explored.

The approach therefore provides much stronger evidence about local high-performing regions than about the complete global objective surface.

Another limitation is Gaussian Process uncertainty calibration.

Several rounds showed that acquisition agreement can be misleading. EI, UCB and posterior mean may all identify similar candidates because they are derived from the same underlying surrogate model.

Agreement between them is therefore useful but does not provide independent confirmation that the GP is correct.

The optimisation method also cannot guarantee identification of the global optimum.

---

## Failure modes

Important failure modes observed during the project include:

### GP overconfidence

The model occasionally predicted a candidate with low uncertainty but the realised value differed by several predicted standard deviations.

This showed that posterior uncertainty should not be treated as perfectly calibrated.

### Uncertainty-driven exploration

Expected Improvement or high-beta UCB sometimes preferred points far from the best observed region because their uncertainty was large.

These points could appear attractive despite having substantially worse posterior means.

### Boundary-seeking behaviour

Acquisition functions sometimes pushed candidates toward either the domain boundary or the edge of a trust region.

This may represent a genuine improvement direction, but it can also result from extrapolation into poorly observed regions.

### High-dimensional sparsity

For the higher-dimensional functions, relatively few observations are available compared with the volume of the search space.

This makes it possible for unexplored strong regions to remain undetected.

### Adaptive sampling bias

Because later data were selected using earlier surrogate models, errors in those models can influence where subsequent data are collected.

This can reinforce confidence in a local basin while leaving alternatives insufficiently explored.

---

## Ethical considerations and transparency

Although this optimisation task does not directly involve sensitive personal data or human decision-making, transparency remains important.

A model card documents:

- what assumptions were made;
- how queries were selected;
- which transformations were applied;
- when the model failed;
- how uncertainty was interpreted;
- how the strategy changed over time.

This supports reproducibility by allowing another researcher to understand not only the final submitted points but also the reasoning and modelling choices that produced them.

Transparency is also important if this type of optimisation method is transferred to a real-world application.

For example, in engineering, healthcare, finance or operational optimisation, a model that appears mathematically confident may still be wrong because its assumptions do not match the underlying system.

Documenting calibration failures and limitations helps prevent Gaussian Process uncertainty from being interpreted as a guarantee of safety or correctness.

Reproducibility is further supported by recording:

- cumulative observations;
- exact submitted coordinates;
- kernel specification;
- preprocessing;
- random seeds;
- candidate-generation rules;
- duplicate thresholds;
- acquisition parameters.

---

## Decision-making transparency

The approach does not make decisions using a single fixed rule.

Instead, it uses the Gaussian Process as a decision-support model and evaluates multiple signals before selecting each query.

This is an important distinction.

The posterior mean, EI and UCB provide quantitative evidence, but the final decision also considers model calibration and the behaviour of previous observations.

As a result, the approach can respond differently to two functions even when their acquisition outputs appear similar.

For example, a high-uncertainty candidate may be considered reasonable for a well-calibrated function but rejected for a function whose previous uncertainty estimates were unreliable.

This adaptive behaviour is one of the main strengths of the approach, but it also means that the decision process must be documented carefully to remain reproducible.

---

## Would additional detail improve the model card?

Some additional detail could improve reproducibility, particularly if another person wanted to recreate every optimisation round exactly.

A fully reproducible technical appendix could include:

- the fitted GP kernel for every function and round;
- all estimated ARD lengthscales;
- candidate-pool sizes;
- trust-region dimensions;
- EI parameters;
- UCB beta values;
- prediction residuals;
- random seeds;
- final submitted coordinates.

However, including all of this directly in the main model card would make it unnecessarily long and could obscure the main behaviour of the optimisation approach.

The current structure is therefore sufficient for explaining:

- what the model does;
- how it makes decisions;
- how the strategy evolved;
- how performance was assessed;
- its main assumptions;
- its main failure modes.

Detailed round-by-round numerical outputs are better retained in the optimisation notebooks and supporting project files.

The model card should therefore function as a clear summary of the methodology, while the notebooks provide the detailed reproducibility record.
