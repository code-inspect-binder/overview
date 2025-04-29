# Overview
# OSF Project Execution Results Summary

This repository documents the execution results of a reproducibility pipeline applied to **296 OSF-hosted projects** containing R code, based on the [StatCodeSearch](https://huggingface.co/datasets/drndr/statcodesearch) dataset.
The reproducibility study was conducted using the **StatCodeSearch** dataset, which is part of the GenCodeSearchNet benchmark suite. This dataset contains code-comment pairs extracted from R scripts hosted on the Open Science Framework (OSF), with a particular focus on projects in the domains of social sciences and psychology involving statistical analysis.

---

## 1. Initial Verification


An initial verification revealed that, among the 296 OSF projects referenced in the dataset, **32 projects** contained missing or inaccessible files (see [File Not Found Projects](results/file_not_found_projects.txt)).  
This finding indicates that a portion of the dataset had become outdated, likely due to file deletions, renaming of files, or modifications made to OSF repositories after the initial data collection.  
The absence of formal version control within many OSF repositories further complicates the recovery or exact reproduction of the original computational environments associated with these projects.

---

## 2. Container Build Status

From the **remaining 264 projects**, containerization failed for **15 projects**, see [Build Failed Projects](results/build_failed_projects.txt).
The failures were primarily due to:

- **Malformed or incomplete `DESCRIPTION` files:** Automatically generated DESCRIPTION files often lacked required fields or had formatting issues.
- **Incorrectly extracted dependencies:** Variables, file paths, or invalid names such as `unknown`, `NULL`, or numbers (`0`, `1`) were mistakenly added as dependencies.
- **Invalid scoped package references:** References like `knitr::opts_chunk` were incorrectly treated as standalone packages.
- **Unavailable or incompatible packages:** Some projects required non-CRAN packages (e.g., `crsh/papaja`, `DF_network`) or packages needing system libraries missing in the container.
- **Failed package installation:** Broken dependencies or compilation failures caused `devtools::install_local(getwd())` to fail during the container build.

---

## 3. GitHub Push Status

From the remaining **249 projects**, **26 projects** could not be pushed to GitHub because they contained files larger than 100 MB, violating GitHub’s size restrictions, see [Git Push Failed Projects](results/git_push_failed_projects.txt)

---

## 4. Successfully Processed Projects

The final set of **226 projects** were:

- Successfully containerized,
- Docker images were built and pushed to Docker Hub.
Docker images are available on [Docker Hub](https://hub.docker.com/u/meet261).

---

## 5. Execution Results Overview

The execution results of these 226 projects are summarized in [execution_results_with_binder.csv](results/execution_results_with_binder.csv)

This CSV file contains the following columns:

| Column | Description |
|:-------|:------------|
| **Project ID** | OSF Project ID |
| **R/Rmd Script** | Name of the R or RMarkdown file executed |
| **Execution Status** | `successful` or `failed` |
| **Reason** | Type of failure (`Code Issue` or `Container Issue`) |
| **Error Message** | Extracted error message indicating the cause of failure |

- **Successful:** The script executed completely inside the container without throwing any error.
- **Failed:** Execution stopped due to an error.  

---

## 6. Error Classification

To help interpret failures, a two-level classification approach was used:

- **Level 1 – Error Message Extraction**:  
  Regular expressions were used to detect and extract recurring error patterns from logs.

- **Level 2 – Semantic Grouping**:  
  Extracted messages were grouped into broader categories to better understand the nature of the failure.

---

## 7. Failure Types

### Code Issues:

Code issues refer to problems within the R or RMarkdown script itself. These errors would likely occur even if the code was executed outside a container. They include:

- **Missing Object or Function**  
  Functions or variables used in the code are undefined or unavailable.  
  _Examples:_ `object not found`, `could not find function`, `unexported object`.

- **Invalid File or Directory Path**  
  File input/output paths are incorrect or unavailable.  
  _Examples:_ `cannot open file`, `no such file or directory`.

- **File Read Error**  
  Code fails when reading from a file, often due to missing data or unsupported formats.

- **Syntax or Argument Errors**  
  Malformed R code or incorrect usage of function arguments.  
  _Examples:_ `unexpected symbol`, `argument is missing`.

- **Data Structure Mismatches**  
  The expected structure (e.g., data frame columns) doesn't match the actual data loaded.

- **Encoding and String Handling Issues**  
  Problems related to reading or writing text with incompatible encodings.

### Container Issues:

Container issues refer to problems where the script fails during execution inside a successfully built containerized environment.  
These failures are **not due to bugs in the script itself**, but because some software dependencies expected at runtime are missing or incorrectly configured.

Although the container is built successfully, errors can still occur during script execution when necessary libraries, system tools, or R packages are missing.

Common examples include:

- **Missing Package**  
  R packages used by the script were not installed or listed properly in the environment.

- **Package Installation Failure**  
  Errors occur during installation of packages inside the container at runtime (e.g., version conflicts, missing compilation tools).

- **Shared Library Load Error**  
  Required system-level shared objects (e.g., `.so`, `.dll` files) were unavailable when an R package tried to load them.

- **RStudio Environment Errors**  
  Scripts depend on features specific to RStudio (e.g., interactive device for plotting) that aren't available inside a Binder container.

- **System-Level Dependency Missing**  
  Scripts require external binaries or OS tools (e.g., `pandoc`, `ImageMagick`) not present inside the container.

Each failure case is classified into one of these categories based on the error message. This classification helps distinguish reproducibility problems caused by the environment from actual bugs in the code.

---
  
## 8. Notes

- Missing projects and file mismatches could be due to a lack of formal version control on OSF.
- Failed builds often highlight real-world challenges in open science reproducibility.
- All Docker images are standardized for long-term use and reproducibility.

---

## 9. Funding

This work was funded by the German Research Foundation (DFG) under project No. 504226141.
