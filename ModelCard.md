# Model Card: Gaussian Process Bayesian Optimisation

**Project:** Imperial College London – Machine Learning and Artificial Intelligence BBO Capstone  
**Model:** Gaussian Process (GP) regression used for sequential Bayesian optimisation  
**Scope:** Eight separate black-box functions, developed and tested over the capstone project

## Model Description

I used a Gaussian Process to help decide which inputs to test next for eight unknown functions. I couldn't see the equations behind the functions, and I only had a limited number of queries, so the idea was to use the results I already had rather than guess each new point from scratch. I fitted a separate GP for each function and updated it as new results came in.

**Input:** Each model takes a vector of continuous numbers between `0` and `1`. The functions have different numbers of inputs, ranging from **2 to 8 dimensions**. The training data are the input vectors I had already tested and the scores returned by the course's evaluation portal. By the time I chose the Week 13 queries, there were **22 to 52 observations per function**.

**Output:** For a point I haven't tested yet, the GP gives a **predicted objective value (posterior mean)** and an **estimate of uncertainty (posterior standard deviation)**. I used those predictions to choose the next input to submit. The actual score only became known after submitting that input to the course portal. All eight functions were **maximisation** problems.

**Model architecture:** I used scikit-learn's `GaussianProcessRegressor`. The later notebooks mainly used a constant-amplitude kernel multiplied by an **ARD Matérn 5/2 kernel**, plus a white-noise term. ARD gives each input its own fitted lengthscale, which helped me see where the GP expected the function to change more quickly. The kernel settings were fitted by optimising the log marginal likelihood, with multiple restarts in later weeks.

I started with simpler GP experiments, including an RBF kernel, and also tried a neural-network model during the project. I kept coming back to the GP because the uncertainty estimates were useful when I had so little data. I compared **Expected Improvement (EI)**, **Upper Confidence Bound (UCB)** and the **highest predicted mean**, rather than relying on one acquisition function every time. Later I added local trust regions, individual-coordinate checks, boundary searches and checks to avoid submitting points too close to ones I'd already tested.

There were a couple of exceptions to the usual output handling. For **Function 1**, I used positive linear scaling in the later rounds to keep the original maximisation objective intact, after experimenting with a logarithmic approach earlier. For **Function 5**, I standardised the large output values before fitting the GP and converted predictions back to the original scale.

## Performance

I measured success mainly by the **best objective value actually returned by the evaluation portal** for each function. I also checked how close GP predictions were to the next observed result, and whether a proposed point improved on the previous best. The functions have very different output scales, so their scores shouldn't be compared directly with each other.

| Function | Best before Week 13 | Actual Week 13 result | Final best observed |
|---|---:|---:|---:|
| 1 | 6.59126e-12 | **7.80718e-11** | **7.80718e-11** |
| 2 | 0.679663 | 0.634828 | 0.679663 |
| 3 | -0.001268 | -0.012963 | -0.001268 |
| 4 | 0.670498 | **0.730122** | **0.730122** |
| 5 | 8662.4825 | 8284.8918 | 8662.4825 |
| 6 | -0.195176 | **-0.136427** | **-0.136427** |
| 7 | 3.077057 | **3.101315** | **3.101315** |
| 8 | 9.999646 | 9.998272 | 9.999646 |

**Four of my eight Week 13 queries produced new best results:** Functions 1, 4, 6 and 7. I was pleased with Functions 4 and 6 in particular, because I chose small changes to one coordinate rather than following the GP's unrestricted suggestion. For Function 7, I did follow the highest predicted mean and it gave me another improvement. Function 3 was a useful reminder that a point can look sensible in the model and still perform worse when it's actually tested.

I also looked at **prediction error relative to the GP's predicted standard deviation** as a practical calibration check. For example, the Week 12 predictions for Functions 7 and 8 were close to their returned values (approximately `+0.23` and `-0.33` predicted standard deviations away). Other predictions were less reliable. This wasn't a formal held-out test: with so few observations, I mainly used the following week's real result to check how the model was doing locally.

The table reports **observed scores, not GP estimates**. The `week13/` datasets and notebooks show the information available when I selected the final points; the returned Week 13 scores were recorded afterwards. More detail is in the [README](README.md) and the [Week 13 notebooks](week13/).

## Limitations

The biggest limitation was the amount of data. Even at the end, I only had a small number of observations for each function, especially compared with the size of the higher-dimensional search spaces. As I found better regions, I sampled near them more often, which was good for local improvement but meant other parts of the space were barely explored.

A GP also makes assumptions about how smoothly a function behaves. Its predicted uncertainty isn't a guarantee that the next result will fall where expected. Some of my queries showed this quite clearly. The fitted white-noise term can account for residual variation, but it doesn't prove that the underlying black-box function itself is noisy. Likewise, **ARD lengthscales are useful model diagnostics, not proof that a variable is important or unimportant**.

I couldn't see the underlying functions or their gradients, and the evaluation portal isn't part of this repository. This means I could reproduce the modelling and candidate-selection steps from my saved observations, but I couldn't independently check the true global maximum. The best results in this project are therefore **the best points I observed**, not confirmed global optima.

## Trade-offs

The main trade-off was **exploration versus exploitation**. Early on it made sense to investigate uncertain areas because I didn't know much about the functions. As the data built up, I found it was often better to stay near strong observations and make smaller changes. In the final week I leaned towards exploitation because there wasn't another round in which I could use information from an exploratory query.

There was also a trade-off between **following the model and listening to the actual results**. EI or UCB sometimes preferred an uncertain point or a direction that had already given disappointing results. I began comparing those suggestions with the highest predicted mean, axis profiles and previous nearby observations before deciding. That helped for Functions 4 and 6, but it didn't mean every conservative decision would work. Function 7 was a good example of trusting the GP when the recent predictions had been reasonably accurate, while Function 3 showed that a coordinated local move could still miss.

Finally, the GP gave me useful uncertainty estimates, but fitting it repeatedly and searching large candidate pools took time, particularly in the later notebooks. I felt that was worthwhile given how limited the actual function evaluations were. If I were doing this again, I'd track prediction errors from the start and be more systematic about testing which parts of my optimisation process were genuinely improving the results.

This card is a summary of the final approach. The full code, choices and experiments are kept in the [project notebooks](week13/) and described further in the [datasheet](DataSheet.md).
