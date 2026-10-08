# Bayesian Black-Box Optimisation: Eight Unknown Functions

**Imperial College London – Machine Learning and Artificial Intelligence Capstone**

## Non-technical explanation

This project was about finding the best possible inputs for eight functions without knowing how the functions actually worked. I could submit a limited number of guesses each week and use the results to decide what to try next. I used a Gaussian Process model to predict which inputs looked promising and how uncertain those predictions were. As I collected more data, I experimented with different ways of balancing exploration with improving the best results I already had. By the end of the challenge, I had improved the best recorded result for all eight functions, although not every weekly experiment worked as expected.

## Repository guide

I kept the weekly work because it shows how my approach changed throughout the challenge, including experiments that didn't improve the score. The Week 13 notebooks show how I chose my final submissions.

| File or folder | What's included |
|---|---|
| [`week1/`](week1/) | Starting input and output data for all eight functions. |
| [`function1.ipynb`](function1.ipynb) to [`function8.ipynb`](function8.ipynb) | Early work for each function, covering Weeks 2–4. There are no separate Week 2, 3 or 4 folders. |
| [`week5/`](week5/) to [`week12/`](week12/) | Weekly experiments, notebooks and saved data. |
| [`week13/`](week13/) | Final-round notebooks, including model predictions, diagnostics and selected queries. |
| [`DataSheet.md`](DataSheet.md) | Details about the data, how it was collected and handled, and its limitations. |
| [`ModelCard.md`](ModelCard.md) | Description of the models, performance, assumptions and limitations. |
| [`Results.xlsx`](Results.xlsx) | Results workbook included in the repository. |
| [`BBO_capstone_results  (version 1).xlsb.xlsx`](BBO_capstone_results%20%20%28version%201%29.xlsb.xlsx) | Additional project results workbook. |

**Where to start:** The results table below gives the overall outcome. For the final decision on a particular function, open the corresponding `week13/functionX_week13.ipynb`. For the earlier development of the project, work through the root notebooks and weekly folders in order.

## Data

The starting observations were provided through the course's black-box optimisation challenge. I submitted new inputs to the course evaluation portal each week and recorded the output returned for each function. The underlying equations and gradients were not available to me.

Each function uses two NumPy arrays:

- `initial_inputs.npy` – the input vectors tested so far, with one point per row.
- `initial_outputs.npy` – the corresponding objective values.

The filenames still say `initial_` in later weeks, but those files contain **cumulative data**, not just the original observations. Inputs were restricted to the range `[0, 1]` in each dimension. The eight functions range from **2 to 8 dimensions**, and the datasets used to choose the final Week 13 points contained between **22 and 52 observations** per function.

The datasets are small enough to keep in the repository. One limitation is that, as the challenge progressed, a lot of my new points were deliberately concentrated near areas that were already performing well. That means the data became useful for local refinement but didn't describe the whole search space equally well. More details are in the [datasheet](DataSheet.md).

## Model and optimisation approach

My main method was **Bayesian optimisation using Gaussian Process (GP) regression**. I fitted a separate GP for each function. The idea was to build an approximate model of each unknown function so I could estimate both the likely result of an untested point and the uncertainty around that estimate.

I started with relatively simple GP models and experimented with different approaches as I went, including an alternative neural-network model and visual plots to help understand the results. The later GP notebooks mainly used a **Matérn 5/2 kernel with ARD lengthscales** and a white-noise term. ARD allowed different input dimensions to have different fitted lengthscales, which was useful when deciding how to search locally.

I compared **Expected Improvement (EI)**, **Upper Confidence Bound (UCB)** and the **highest predicted mean** when selecting points. Earlier on I was more willing to explore uncertain regions. Later, I introduced local trust regions, checks along individual input dimensions, comparisons with previous nearby observations and checks that new suggestions were not too close to existing points.

Towards the final week, I generally favoured small, evidence-based moves around strong observations. I didn't always take the point with the highest EI or UCB; I also checked whether its direction made sense given what the previous queries had actually returned.

There were a few function-specific decisions. For **Function 1**, I eventually stopped using the earlier logarithmic experiment and used positive linear scaling so that the original maximisation objective was preserved. For **Function 5**, I standardised the output manually before fitting its GP and converted predictions back to the original scale.

## Hyperparameter optimisation

I used scikit-learn's `GaussianProcessRegressor` to fit the kernel amplitude, ARD lengthscales and white-noise level by optimising the **log marginal likelihood**. Later notebooks used several optimiser restarts to reduce dependence on a single initial setting. The Matérn smoothness parameter was fixed at `nu=2.5`.

There wasn't one set of search parameters that worked for every function. I adjusted the candidate pools, trust-region widths and exploration settings based on the amount of data available and what recent observations showed. I also compared earlier predictions with the actual values returned by the portal. When the GP had been inaccurate in a particular area, I treated its next recommendation more cautiously.

These were practical adjustments during the challenge, rather than the result of an exhaustive hyperparameter sweep.

## Results

All eight functions were **maximisation** problems. Functions 3 and 6 returned negative values around their best observed points, so a result closer to zero (or positive) was better. The output scales are different between functions, so scores should be compared **within a function**, not across functions.

The table shows the best values in the starting data, the best values entering Week 13, the **actual output returned from the final Week 13 query**, and the overall best observed value at the end of the project.

| Function | Dimensions | Best starting value | Best before Week 13 | Week 13 output | Final best observed |
|---|---:|---:|---:|---:|---:|
| 1 | 2 | 7.71088e-16 | 6.59126e-12 | **7.80718e-11** | **7.80718e-11** |
| 2 | 2 | 0.611205 | 0.679663 | 0.634828 | 0.679663 |
| 3 | 3 | -0.0348353 | -0.00126784 | -0.0129625 | -0.00126784 |
| 4 | 4 | -4.02554 | 0.670498 | **0.730122** | **0.730122** |
| 5 | 4 | 1088.85962 | 8662.48250 | 8284.89184 | 8662.48250 |
| 6 | 5 | -0.714265 | -0.195176 | **-0.136427** | **-0.136427** |
| 7 | 6 | 1.364968 | 3.077057 | **3.101315** | **3.101315** |
| 8 | 8 | 9.598482 | 9.999646 | 9.998272 | 9.999646 |

**Four of the final eight queries produced new best results:** Functions 1, 4, 6 and 7. I was especially pleased that the more conservative, single-coordinate decisions worked for Functions 4 and 6, while the GP's local highest-mean recommendation worked well for Function 7. Not everything worked: Function 3's final move performed noticeably worse than the incumbent, and Function 5 showed how quickly the score dropped when moving away from `[1, 1, 1, 1]`.

This was probably my main takeaway from the project: a GP is useful for choosing what to try, but its predictions still need to be tested against what has actually happened. By the end I was using a combination of model predictions, uncertainty, local checks and past results instead of relying on a single acquisition score.

These results are the best **observed within the available query budget**. They are not proof that the true global maximum was found for any function. The `week13/` input and output arrays are the data that were available **before** the final submissions; the final returned values are recorded in the table above and should not be mistaken for predictions from those notebooks.

## Viewing and running the notebooks

GitHub should display the saved notebooks and their recorded outputs. To run them interactively, I used **Jupyter Notebook/JupyterLab**, with Python, NumPy, SciPy and scikit-learn.

The notebooks use **relative file paths**, so the working directory matters:

1. For the final-round notebooks, open Jupyter with `week13/` as the working directory. Paths such as `function1/initial_inputs.npy` then resolve to the Week 13 cumulative data.
2. The root-level notebooks are earlier experiments. Some references to `functionX/initial_inputs.npy` may need to be pointed at the corresponding files in `week1/functionX/` when rerunning them.
3. Check the paths in other weekly notebooks before running them and restart the kernel between experiments so that variables from another notebook do not affect the result.

Running the code can reproduce the surrogate-modelling and candidate-selection steps using the saved observations. **Evaluating a new candidate requires the course's external black-box portal**, which is not included in this repository. Some early randomly generated candidate lists may differ between runs, depending on random seeds and library versions. The saved notebooks show the analysis carried out during the project.

## Project limitations

The main limitations were the small evaluation budget, uneven coverage of the input space, uncertainty in the GP predictions and the fact that the actual black-box functions were unavailable. Some candidate-selection choices also involved judgement based on recent observations, rather than following one fully automatic algorithm. I have kept the weekly experiments so those decisions and their outcomes can be reviewed.

For more detail, see [DataSheet.md](DataSheet.md) and [ModelCard.md](ModelCard.md).
