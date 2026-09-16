# Model Card: Bayesian Black-Box Optimisation Approach

## Overview

**Name:** Adaptive Gaussian Process Bayesian Optimisation  
**Type:** Sequential black-box optimisation  
**Version:** Final capstone approach developed across ten optimisation rounds

This approach was used to maximise eight unknown black-box functions under a limited query budget. Gaussian Process regression was used as a surrogate model, with candidate points assessed using posterior mean, Expected Improvement (EI), Upper Confidence Bound (UCB), trust-region searches and calibration checks.

The strategy evolved over time rather than relying on a fixed acquisition rule.

## Intended use

The approach is suitable for expensive or query-limited optimisation problems where gradients are unavailable and the input domain is bounded.

It is most appropriate where the objective function is reasonably smooth locally and previous observations can inform future queries.

It should not be treated as guaranteeing a global optimum, and it may perform poorly for highly discontinuous functions, very high-dimensional search spaces, or situations where GP uncertainty is badly calibrated.

## Details

Each function was modelled separately using Gaussian Process regression with an ARD Matérn 5/2 kernel and white-noise term.

Early rounds used broader exploration because little was known about the functions. As more observations were collected, searches became more concentrated around strong-performing regions.

Candidate pools included local, wider and global points. EI, posterior mean and UCB were compared rather than using one acquisition rule automatically.

In later rounds, previous GP predictions were compared with the actual returned values using the standardised residual:


Large residuals were treated as evidence that GP uncertainty was unreliable. In those cases, later searches used tighter trust regions and placed more emphasis on exploitation.

Function 1 used positive linear output scaling because its best values were extremely close to zero. Function 5 used manual standardisation because its objective values were on a much larger numerical scale.

## Performance

The main performance measure was the best observed objective value for each of the eight functions.

Additional diagnostics included:

- improvement between rounds;
- predicted mean and standard deviation;
- standardised prediction error;
- distance from the current best point;
- agreement between EI, posterior mean and UCB;
- boundary behaviour.

The approach successfully identified stable high-performing regions for several functions, although some functions exposed weaknesses in GP calibration. In particular, some realised observations fell several predicted standard deviations away from the GP mean.

These failures influenced later decisions and led to more conservative local searches.

## Assumptions and limitations

The approach assumes that the unknown functions are locally smooth enough for a Matérn Gaussian Process to provide useful rankings.

The main limitations are:

- sparse observations relative to the search-space size;
- adaptive sampling bias toward previously promising regions;
- possible GP overconfidence;
- uncertainty-driven acquisition functions moving into poorly explored regions;
- no guarantee of finding the true global optimum.

ARD lengthscales were treated as indicators of fitted variation, not definitive measures of feature importance.

## Ethical considerations

Transparency is important because optimisation decisions depend on modelling assumptions and uncertainty estimates.

The project records preprocessing choices, GP configuration, candidate generation, acquisition parameters, submitted coordinates and prediction errors. This supports reproducibility and makes it possible to understand both successful and unsuccessful decisions.

In real-world applications, this transparency would be especially important because a mathematically confident prediction can still be wrong if the surrogate model does not represent the underlying system well.

## Decision-making and transparency

The final query was selected using several signals rather than a single acquisition function.

Posterior mean, EI, UCB, uncertainty, distance from the incumbent, boundary behaviour and previous calibration were considered together.

This adaptive decision process is a strength because it allows the optimisation strategy to respond to model failures. Its main limitation is that the reasoning must be documented carefully so another person can reproduce the decisions.

More technical detail could improve exact reproducibility, but including every kernel fit, candidate pool and diagnostic would make the model card unnecessarily long. Those details are better retained in the optimisation notebooks, while this model card provides the methodological summary.
