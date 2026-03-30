My R Learning Journey — From Deployment Engineering to Open Source Packages

How I got here
My entry into R was not through a course or tutorial. It came sideways, from a completely different domain. My first serious engineering project was a fire and weapon detection system running on live CCTV feeds — YOLO, PyTorch, VGG16, Faster RCNN, trained from scratch and deployed in production. The model accuracy was the easy part. What stayed with me was everything else: the gap between something that works in a notebook and something that works reliably when real data hits it. Reproducibility, explicit state, edge cases you never anticipated — that is where the actual engineering lives.
When I started looking at open source software, I kept coming back to R packages specifically because they sit at exactly that intersection — software engineering inside scientific practice. Scientists write the logic. The package is what makes that logic reusable, testable, and trustworthy across contexts no one anticipated when the code was first written. That was the same problem I had been thinking about since the CCTV project. So I stopped using packages and started reading them.

How R packages actually work — the fundamentals
The first thing that surprised me was how much of an R package is infrastructure rather than logic. The actual functions you write are a small fraction of what makes a package work. The rest is:
Package structure — R/, tests/, man/, NAMESPACE, DESCRIPTION. Understanding why each exists matters. NAMESPACE controls what your package exposes to users and what it imports from other packages. Getting imports wrong causes silent failures that are hard to trace.
roxygen2 for documentation — you write documentation above functions using #' tags. @param, @return, @export, @examples, @details. The @details tag is where you document non-obvious behavior — silent fallbacks, edge cases, behavior that differs from what the function signature implies. I learned this the hard way when I found a metric function that silently switches between two computation methods without warning the user. The fix was a @details entry and an inline comment. That is what documentation actually is — not describing what the code does, but describing what the user would not expect.
devtools workflow — devtools::load_all() reloads your package in place without reinstalling. devtools::check() runs the full CRAN check suite. devtools::test() runs testthat. These three commands are the core development loop.
DESCRIPTION dependencies — Imports means your package requires it. Suggests means it is optional — used in tests or vignettes but not required for the package to function. Getting this wrong breaks other people's installs.

Testing with testthat
Before contributing to packages I thought tests were for confirming that functions return the right answer. That understanding was wrong.
The more important use of tests is revealing behavior the author did not document. When you write a test for an edge case and it fails unexpectedly, that is information — either the function has a bug, or it has undocumented behavior that the next person will stumble on too. Both are worth fixing.
The testthat structure:
rtest_that("description of what should happen", {
  result <- function_under_test(input)
  expect_equal(result, expected_value)
  expect_true(condition)
  expect_error(bad_call(), "error message pattern")
})
What I learned about writing good tests:
Writing tests for metric_RMSE, metric_MAE, metric_R2, and metric_cor taught me several things. First, test the normal case, then test boundary conditions — all identical values, single-element vectors, vectors with NA. Second, test that errors are thrown when they should be, not just that correct inputs return correct outputs. Third, the most useful tests are the ones that make you uncomfortable to write — the ones where you are not sure what the function should do.
metric_R2 was the clearest example. Writing a test for it revealed that it silently switches between two R-squared computation methods depending on input structure, with no warning to the user. The function worked. But it did not behave transparently. That distinction — working vs. transparent — is one of the most important things I learned.
CI-safe tests are tests that pass regardless of the environment they run in. No network calls. No file system dependencies. No hardcoded paths. When your test requires a file on disk, mock it or create a temporary file in tempdir() and clean up after. When your test requires a package that is not available in CI, guard it with skip_if_not_installed().
Dependency issues in restricted network environments — where GitHub API access is blocked — can be resolved by installing packages locally via remotes::install_local() and sourcing files directly with a mocked logger. This came up when setting up tests for PEcAn's benchmarking module locally.

Reading large codebases
The skill that improved most through open source contribution was reading code I did not write, in a codebase I did not design, looking for something I could not fully describe until I found it.
The approach that works:

Start from the user-facing function and trace inward. What does it call? What does that call?
Look at the tests before reading the implementation — tests describe intended behavior concisely
Read the git log for files you care about — commit messages tell you why things changed
Search for where a function is called, not just where it is defined — usage context reveals intent

Reading SDA_runner.R and the WillowCreek and hf_landscape workflow scripts in PEcAn taught me how scientific workflow code is structured differently from software engineering code. It is organized around the experiment, not around reusability. The same seven steps — load settings, load observations, align, preprocess, compute metrics, plot, store — appear in every script, written from scratch every time. That is not a criticism of the scientists who wrote them. It is a structural observation: the tooling did not exist to do otherwise. Recognizing that pattern is what led to the validation framework design.

Navigating maintainer feedback
The most technically dense part of open source contribution is not the code. It is the conversation around the code.
What I learned from multiple review cycles:
Maintainers push back on scope for reasons that are not obvious from the outside. A PR that does two things when it should do one is harder to review, harder to revert, and harder to reason about in the future. Scoping strictly — even when you can see a larger fix — is a sign of understanding how the codebase will be maintained, not a sign of doing less work.
Path detection logic for platform differences (Windows vs. Unix vs. macOS) is harder than it looks because the edge cases are environmental. Nested paths, symlinks, and platform-specific separators all behave differently. Testing this reliably in CI means not relying on the specific environment you developed on.
When a maintainer gives detailed feedback across multiple rounds, the right response is not to defend the original implementation. It is to understand why it was wrong. The gap between "I fixed what they asked for" and "I understand why my first approach was wrong" is where actual skill development happens.
Communicating in issues and PRs:

Describe what the PR does, not what you changed
When tests fail in CI but pass locally, investigate whether it is a pre-existing infrastructure issue before assuming your code is wrong
When you find a pre-existing bug while writing tests, document it clearly and separately — do not silently fix it inside a different PR


Key R ecosystem concepts
The units package — handles physical unit tracking and conversion automatically. Instead of writing custom conversion logic for every variable-dataset combination, you attach units to numeric vectors and the package handles dimensional analysis. This is how PEcAn standardizes units across datasets from different sources (FLUXNET, NEON, custom CSV).
rmarkdown::render() and parameterized reports — R Markdown documents can accept parameters via a params: block in the YAML header. This means the same .Rmd template can produce different reports depending on inputs without any code changes:
yaml---
title: "PEcAn Validation Report"
output:
  html_document:
    toc: true
    toc_float: true
    self_contained: true
params:
  ensemble_dir: ~
  obs_loader: "fluxnet"
  variable: "GPP"
  site_ids: NULL
  aggregation: "monthly"
---
The self_contained: true option embeds all assets (CSS, JS, images) in the HTML file so it can be shared without a server.
S3 generics — R's primary mechanism for polymorphism. A generic function like load_obs() dispatches to different methods based on the class of its input. This is how you build extensible interfaces in R without requiring users to touch existing code when adding new functionality.
dplyr group operations — group_by() followed by summarise() is the core pattern for computing per-group statistics. In a multi-site validation context this means computing RMSE or bias grouped by site_id, by biome, by PFT, or by time period using the same function call — just changing the group_by argument.
NetCDF output format — PEcAn stores model ensemble outputs in NetCDF files. Reading these requires the ncdf4 package. The key operations are nc_open(), ncvar_get() to extract a variable array, and nc_close(). Ensemble outputs have an extra dimension for ensemble member number, which is why collapsing to median before metric computation loses uncertainty information.
Probabilistic metrics vs. point-estimate metrics — RMSE and MAE work on single values per timestep (e.g., ensemble median). CRPS (Continuous Ranked Probability Score) works on the full ensemble distribution and rewards both accuracy and calibrated uncertainty. If your model is uncertain and honest about it, CRPS rewards that. RMSE on the median throws away everything the ensemble distribution tells you about uncertainty. This distinction is central to validating ensemble model outputs correctly.

What documentation actually is
The last thing, and the one I keep coming back to: documentation is not describing what the code does. Anyone can read the code. Documentation is describing what the user would not expect — the silent fallback, the edge case that changes behavior, the argument that interacts with another argument in a non-obvious way. If you would have been surprised by a behavior when you first read the code, document it. That surprise is the signal.
