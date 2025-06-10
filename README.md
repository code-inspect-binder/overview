# OSF Project Execution Results Summary

The reproducibility study was conducted using the [StatCodeSearch](https://huggingface.co/datasets/drndr/statcodesearch) dataset, part of the GenCodeSearchNet benchmark suite. This dataset consists of code-comment pairs extracted from R scripts hosted on the Open Science Framework (OSF), with a particular focus on research in the social sciences and psychology involving statistical analysis.

From this dataset, we extracted the OSF project identifiers associated with the R scripts to retrieve and process complete research materials from OSF. In total, **296 unique OSF projects** were identified and analyzed through our reproducibility pipeline.

The complete list of project identifiers is available in [**project_ids**](https://github.com/code-inspect-binder/overview/blob/main/metadata/project_ids.csv).

This CSV contains:

| Column         | Description                              |
|----------------|------------------------------------------|
| **Project ID** | OSF project identifier for each study    |

## Reproducibility Pipeline
The reproducibility pipeline used in this study is implemented in the companion repository [**osf-to-binder**](https://github.com/Code-Inspect/osf-to-binder).

It automates the end-to-end process of:

- Downloading project materials from OSF using project identifiers
- Extracting R code dependencies
- Building reproducible Binder containers
- Executing scripts within these environments
- Collecting and classifying the resulting logs

---

## 1. Project Processing Failures

This includes a summary of OSF-hosted R projects that failed at various stages of a reproducibility pipeline. The results are consolidated in [project_processing_failures](https://github.com/code-inspect-binder/overview/blob/main/results/project_processing_failures.csv), which contains:

| Column            | Description                                                                 |
|-------------------|-----------------------------------------------------------------------------|
| **Project ID**     | OSF project identifier                                                     |
| **Failure Stage**  | Stage where the failure occurred: `File Not Found`, `Container Build Fail`, or `Git Push Fail` |
| **Reason**         | Explanation of the failure                                                 |

### Failure Types

- **File Not Found – 32 projects**  
  Files were missing or inaccessible on OSF—often due to deletion, renaming, or lack of version control.

- **Container Build Fail - 15 projects**  
  The container build process failed due to issues like malformed `DESCRIPTION` files, unresolved dependencies, or invalid package references.

- **Git Push Fail – 23 projects**  
  Projects exceeded GitHub's 100MB file limit, preventing the repository from being pushed.

---

## 2. Project Processing Successes

The final set of **226 projects** were:

- Successfully containerized  
- Docker images were built and pushed to Docker Hub

Docker images are available on [Docker Hub](https://hub.docker.com/u/meet261).

In addition to the standard repositories, we also created **flowR-enabled variant** for each project. These include an `.Rprofile` and `postBuild` script that ensure each container launches directly into RStudio with the [flowR](https://github.com/flowr-analysis/rstudio-addin-flowr) package preinstalled and available as an RStudio addin for interactive dependency exploration.

Repositories follow this naming pattern:
- Standard: `code-inspect-binder/osf_projectid`
- flowR-enabled: `code-inspect-binder/osf_projectid-f`

**Example:**  
- Without flowR: [`code-inspect-binder/osf_3kem6`](https://github.com/code-inspect-binder/osf_3kem6)
- With flowR: [`code-inspect-binder/osf_3kem6-f`](https://github.com/code-inspect-binder/osf_3kem6-f)

---

## 3. Execution Results Overview

The execution results of these 226 projects are summarized in [execution_results_with_binder](https://github.com/code-inspect-binder/overview/blob/main/results/execution_results_with_binder.csv)

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

## 4. Error Classification

To help interpret failures, a two-level classification approach was used:

- **Level 1 – Error Message Extraction**:  
  Regular expressions were used to detect and extract recurring error patterns from logs.

- **Level 2 – Semantic Grouping**:  
  Extracted messages were grouped into broader categories to better understand the nature of the failure.

---

## 5. Failure Types

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

## Citation

If you use this repository or refer to this work, please cite:

Saju, L., Holtdirk, T., Mangroliya, M. P., & Bleier, A. (2025).  
*Computational Reproducibility of R Code Supplements on OSF*.  
arXiv preprint arXiv:2505.21590. https://arxiv.org/abs/2505.21590

This work was funded by the German Research Foundation (DFG) under project No. 504226141.
