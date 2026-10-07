# Bayesian Black-Box Optimisation of Eight Unknown Functions

**Author:** Caitlin Hale  
**Project:** Imperial College London Machine Learning and Artificial Intelligence capstone

## Non-technical explanation of the project

This project explores how to find better solutions when the relationship between inputs and results is unknown and only a limited number of experiments is available. I worked with eight hidden functions, using previous results to decide which inputs to test next. A statistical model estimated promising locations and the uncertainty around them. Each new result helped me adjust the balance between exploring unfamiliar areas and improving known good solutions. The recorded best results improved for all eight functions. The project also showed that confident predictions can be misleading, making careful checks and transparent reporting essential throughout the optimisation process.

## Repository guide

**Weeks 2–4 are combined in the root-level `function1.ipynb` to `function8.ipynb` files.** Each `functionX.ipynb` contains the early work for that function, including the combined Weeks 2–4 analysis. There are therefore no separate `week2/`, `week3/` or `week4/` folders. These notebooks should be read as the early optimisation history; the final query-selection work is in `week13/`.

| File or folder | Contents |
|---|---|
| [function1.ipynb](function1.ipynb), [function2.ipynb](function2.ipynb), [function3.ipynb](function3.ipynb), [function4.ipynb](function4.ipynb), [function5.ipynb](function5.ipynb), [function6.ipynb](function6.ipynb), [function7.ipynb](function7.ipynb), [function8.ipynb](function8.ipynb) | Early notebooks, with Weeks 2–4 combined by function. |
| [week1/](week1/) | Starting input and output datasets for all eight functions. |
| [week5/](week5/) and [week6/](week6/) | Subsequent weekly optimisation notebooks. |
| [week7/](week7/), [week8/](week8/), [week9/](week9/), [week10/](week10/), [week11/](week11/), [week12/](week12/) | Later weekly notebooks and cumulative data snapshots. |
| [week13/](week13/) | Final query-selection notebooks, including the reasoning, diagnostics and selected inputs for each function. |
| [DataSheet.md](DataSheet.md) | Data composition, collection, preprocessing, intended use and limitations. |
| [ModelCard.md](ModelCard.md) | Model approach, performance diagnostics, assumptions and limitations. |
| [BBO_capstone_results  (version 1).xlsb.xlsx](BBO_capstone_results%20%20%28version%201%29.xlsb.xlsx) | Results workbook recording the optimisation history. |

For a quick review, start with the results below, then read the model card and the relevant `week13/functionX_week13.ipynb` notebook. To follow how the approach developed, read the root function notebooks and then the weekly folders in numerical order.

## Data

The data were supplied and generated through the course's black-box optimisation task. The starting observations are stored in `week1/`. Further observations were obtained by submitting input vectors to the course evaluation portal and recording the returned objective values.

Each function has its own dataset:

- `initial_inputs.npy`: an array of input vectors, with one observation per row.
- `initial_outputs.npy`: the corresponding observed objective values.

Despite retaining the `initial_` filenames, the arrays in later weekly folders contain cumulative observations. They should not all be interpreted as the original starting data.

Inputs lie within `[0, 1]` in each dimension. The functions have between two and eight input dimensions. The starting datasets contain 10–40 observations per function; the data used for Week 13 contain 22–52 observations per function. These are small datasets and are included directly in the repository.

The functions' internal formulas and gradients are unavailable. Consequently, the observations provide limited evidence about the full search space. Later observations also reflect the optimisation strategy's preference for promising regions, rather than an independent random sample. Further context is documented in [DataSheet.md](DataSheet.md).

## Model

I fitted a separate Gaussian Process (GP) regression model for each function. The GP acts as a surrogate: it estimates an objective value and predictive uncertainty for untested inputs, helping choose the next experiment without access to the underlying function.

Early experiments used an RBF kernel. Later notebooks use a constant-amplitude term multiplied by a Matérn 5/2 kernel with separate length scales for each input dimension (automatic relevance determination, or ARD), plus a white-noise term. This provides a flexible model of local variation while allowing different behaviour along different input dimensions.

Candidate selection evolved across the project. I compared Upper Confidence Bound (UCB), Expected Improvement (EI), posterior uncertainty and predicted mean. Candidate pools included global samples and local samples around the best observed input. Later rounds used tighter trust regions and directional checks where appropriate.

For Function 1, early notebooks explored a logarithmic transformation; the final approach uses positive linear scaling to preserve the original maximisation objective. Function 5 uses manual output standardisation, with predictions interpreted on the original scale. Most other functions use the GP's internal output normalisation.

## Hyperparameter optimisation

The GP's kernel amplitude, ARD length scales and white-noise level were fitted using scikit-learn's Gaussian Process optimiser, which maximises the log marginal likelihood within the bounds specified in each notebook. Later notebooks use multiple optimiser restarts to reduce dependence on a single starting point.

The Matérn smoothness parameter is fixed at `nu=2.5`. Acquisition settings, such as the UCB exploration weight and EI improvement offset, and search settings, such as candidate counts and trust-region widths, were chosen and adjusted using the observed results and calibration diagnostics. These settings were adapted across functions and weeks rather than selected through one exhaustive hyperparameter search.

Predictions from previous rounds were compared with the returned observations. Large errors relative to predicted uncertainty prompted more cautious local searches. The final round generally placed greater emphasis on exploiting established strong regions because there was no remaining round in which to benefit from additional exploration.

## Results

The table reports the best observed objective in the starting data and in the cumulative data loaded by the Week 13 notebooks. **It describes observations available before the final Week 13 query, rather than predictions or returned results from that final query.** All functions are maximised, so a less negative value is an improvement for Functions 3 and 6.

| Function | Input dimensions | Best starting value | Best observed value entering Week 13 |
|---|---:|---:|---:|
| 1 | 2 | 7.71088e-16 | 6.59126e-12 |
| 2 | 2 | 0.611205 | 0.679663 |
| 3 | 3 | -0.0348353 | -0.00126784 |
| 4 | 4 | -4.02554 | 0.670498 |
| 5 | 4 | 1088.85962 | 8662.48250 |
| 6 | 5 | -0.714265 | -0.195176 |
| 7 | 6 | 1.364968 | 3.077057 |
| 8 | 8 | 9.598482 | 9.999646 |

The best observed value improved for every function, although individual queries did not always improve on the incumbent. Function 5's best observed input was the upper corner `[1, 1, 1, 1]`. Function 8 produced several closely spaced high-performing observations, supporting a focused local search. Function 1 remained difficult to model in absolute terms because its outputs span very different numerical scales.

The main lesson was that predictive confidence must be checked against actual observations. Some functions showed substantial calibration errors, so uncertainty alone was not a reliable reason to move away from a proven region. Comparing acquisition rules, inspecting boundaries and using recent directional evidence helped inform later decisions.

These results establish the best values found within the available query budget. They do not establish the true global maxima, and objective values should be compared within each function because their scales differ. The Week 13 notebooks retain the final selected coordinates, predicted values and selection reasoning; a predicted improvement should not be treated as an observed improvement.

## Viewing and running the notebooks

GitHub can display the saved notebook code and outputs. For interactive use, open the notebooks in JupyterLab or Jupyter Notebook with Python, NumPy, SciPy and scikit-learn installed.

The notebooks use relative data paths. Run the Week 13 notebooks with `week13/` as the working directory so paths such as `function1/initial_inputs.npy` resolve correctly. The root notebooks use paths such as `function1/initial_inputs.npy`; when revisiting those early experiments, point them to the starting data under `week1/function1/` and the corresponding folders for the other functions. Check earlier weekly notebooks' load paths before running them, and restart the kernel before rerunning a notebook to avoid carrying state across experiments.

Running a notebook fits the surrogate and generates candidate recommendations. Obtaining a new objective value requires access to the course's evaluation portal. Exact regenerated candidates may vary with software versions or unseeded sampling in early experiments. The saved outputs preserve the recorded analysis.
