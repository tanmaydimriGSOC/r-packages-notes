Notes on R Packages

This repository contains my personal notes and experiments while learning, building, and contributing to R packages within the open-source ecosystem. It serves as both a learning log and a reference as I explore real-world R package development practices.

Purpose of This Repository

This repository is created to practice building and understanding the structure of R packages. It focuses on learning documentation standards, function organization,testing, and core package development workflows using GitHub and CRAN guidelines.

I am preparing for open-source contributions and programs such as Google Summer of Code (GSoC). Maintaining this repository helps me systematically document what I learn while navigating R package internals, contribution workflows, and community-driven development.

Topics Covered

1.Understanding CRAN package structure and standards

2.Writing, reviewing, and maintaining vignettes

3.Managing backward compatibility in R APIs

4.The importance of not breaking existing behavior

5.Learning from code reviews, issues, and rejected PRs

6.Versioning, NEWS files, and release practices

7.Basic testing workflows using testthat

8.Documentation using roxygen2

9.Package checks and CRAN submission considerations

Why R Packages?

R packages are the backbone of the R ecosystem. Learning how they are structured and maintained provides insight into:

Writing reusable and maintainable code

Collaborating effectively in open-source projects

Following community conventions and best practices

Designing APIs that are stable and user-friendly over time

This repository reflects my ongoing effort to move from being a package user to a meaningful package contributor.

CRAN Package Structure

CRAN packages follow a well-defined directory structure that ensures consistency, reliability, and ease of maintenance. Understanding this structure is essential for contributing effectively and for writing packages that meet CRAN standards.

Common components include:

DESCRIPTION – Metadata about the package (name, version, dependencies, author, license)

NAMESPACE – Controls which functions are exported and imported

R/ – Core R source files containing functions

man/ – Automatically generated documentation files (.Rd)

vignettes/ – Long-form documentation and tutorials

tests/ – Unit tests, usually written with testthat

inst/ – Additional files installed with the package

NEWS.md – User-facing changes across versions

README.md – Overview and usage information

Through this repository, I document what each of these components does, how they interact, and common mistakes encountered while working with them.

Learning Approach

Read and analyze existing CRAN and GitHub-hosted packages

Reproduce package structures from scratch

Submit small PRs and learn from feedback

Study CRAN policies and check results

Treat rejections and reviews as learning opportunities

Long-Term Goal

The long-term goal of this repository is to build a strong foundation in R package development and open-source collaboration, enabling me to confidently contribute to established projects and participate in programs like GSoC.
