## Computational Reproducibility
**Computational reproducibility** refers to the ability to regenerate the original results of a study using the shared code and data. It is essential for ensuring transparency, trust, and long-term impact—particularly in fields like **social science and psychology**, where data-driven research informs public policy, education, health, and other societal outcomes.

To support this effort, we developed an automated pipeline that identifies dependencies, builds containerized environments, and executes code artifacts in a controlled setting. The pipeline is available in the [**osf-to-binder**](https://github.com/Code-Inspect/osf-to-binder) repository, where it supports large-scale reproducibility testing using Binder and Docker.

This GitHub organization and its repositories were created to organize, track, and share the results of this reproducibility analysis after executing and evaluating all code artifacts. Each sub-repository corresponds to a specific OSF project and includes reconstructed environments and execution results.

---

## Execution Summary
Using this reproducibility pipeline, we analyzed a collection of R scripts sourced from the [StatCodeSearch](https://huggingface.co/datasets/drndr/statcodesearch) dataset within the GenCodeSearchNet benchmark. These R scripts, which focus on statistical analyses in social science and psychology, were sourced from the GenCodeSearchNet benchmark and linked to relevant projects on the Open Science Framework (OSF), a platform for sharing research materials. In total, 296 unique OSF projects were identified and examined through our reproducibility pipeline.

The complete list of project identifiers is available in [**project_ids.csv**](https://github.com/code-inspect-binder/overview/blob/main/metadata/project_ids.csv), which contains:

| Column         | Description                              |
|----------------|------------------------------------------|
| **Project ID** | OSF project identifier for each study    |

We found that only **264 projects** were retrievable from OSF. Among them, nearly **99% lacked proper dependency information**, such as missing `DESCRIPTION` files, environment specifications (e.g., `renv.lock`, `sessionInfo()` output), or other metadata needed to reproduce the software environment.

When we attempted to run the scripts, only **25.87%** completed successfully without errors. The remaining scripts encountered issues like missing packages, broken file paths, or coding errors that caused execution to stop.

At the project level:

- **40 projects (16%)** were [fully reproducible](https://github.com/code-inspect-binder/overview/blob/main/results/fully_reproducible_projects.csv) (i.e., all scripts ran successfully),
- **34 projects (13.65%)** showed partial reproducibility (i.e., some scripts ran successfully),
- **175 projects (70%)** had no scripts that ran successfully.

These findings highlight persistent barriers to reproducibility at scale, including missing R packages, broken file paths, and system-level incompatibilities. They also demonstrate how automation and standardized pipelines can help researchers better assess and improve reproducibility in shared research code, especially in domains like social science where reproducible evidence is critical.

---

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

This section summarizes OSF-hosted R projects that failed at various stages of the reproducibility pipeline. The results are consolidated in [project_processing_failures.csv](https://github.com/code-inspect-binder/overview/blob/main/results/project_processing_failures.csv), which contains:

| Column            | Description                                                                 |
|-------------------|-----------------------------------------------------------------------------|
| **Project ID**     | OSF project identifier                                                     |
| **Failure Stage**  | Stage where the failure occurred: `File Not Found` or `Container Build Fail` |
| **Reason**         | Explanation of the failure                                                 |

### Failure Types

- **File Not Found – 32 projects**  
  Files were missing or inaccessible on OSF—often due to deletion, renaming, or lack of version control.

- **Container Build Fail – 15 projects**  
  The container build process failed due to issues like malformed `DESCRIPTION` files, unresolved dependencies, or invalid package references.

---

## 2. Project Processing Successes

The final set of **249 projects** successfully passed earlier stages of the reproducibility pipeline (e.g., data availability, container build).

These projects now fall into two categories:

- **226 projects** were **successfully containerized** and successfully pushed to both [Docker Hub](https://hub.docker.com/u/meet261) and GitHub under the [code-inspect-binder](https://github.com/code-inspect-binder) organization.
- **23 projects** were **successfully containerized** but could **not be pushed to GitHub or Docker Hub** due to file size limitations (e.g., files >100MB exceeding GitHub’s upload limit).

Despite the Git push issues, a few of these 23 projects were successfully containerized, and their corresponding R scripts ran without error. These are considered functionally reproducible, even though their repositories could not be published to GitHub or Docker Hub.

A complete list of [fully reproducible](https://github.com/code-inspect-binder/overview/blob/main/results/fully_reproducible_projects.csv) projects — i.e., projects where all scripts ran successfully — is available at the linked CSV file.

This file also includes a `Git Push Success` column indicating whether the GitHub push step succeeded (`True`) or failed (`False`).

### Repositories and FlowR Integration

For the 226 published projects:

- Standard repositories: `code-inspect-binder/osf_projectid`
- FlowR-enabled variants: `code-inspect-binder/osf_projectid-f`

Each flowR-enabled container launches directly into RStudio with the [flowR](https://github.com/flowr-analysis/rstudio-addin-flowr) package preinstalled as an RStudio addin for interactive dependency exploration.

**Example:**  
- Without flowR: [`code-inspect-binder/osf_6jmke`](https://github.com/code-inspect-binder/osf_6jmke)  
- With flowR: [`code-inspect-binder/osf_6jmke-f`](https://github.com/code-inspect-binder/osf_6jmke-f)

---

## 3. Execution Results

The execution results of these projects are summarized in [execution_results_with_binder](https://github.com/code-inspect-binder/overview/blob/main/results/execution_results_with_binder.csv)

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
```
@inproceedings{Saju2025Computational,
  title     = {Computational Reproducibility of R Code Supplements on {OSF}},
  author    = {Saju, Lorraine and Holtdirk, Tobias and Mangroliya, Meetkumar Pravinbhai and Bleier, Arnim},
  booktitle = {R2CASS 2025: Social Science Meets Web Data: Reproducible and Reusable Computational Approaches, Workshop Proceedings of the 19th International AAAI Conference on Web and Social Media},
  year      = {2025},
  doi       = {10.36190/2025.49},
  publisher = {Association for the Advancement of Artificial Intelligence}
}
```

This work was funded by the German Research Foundation (DFG) under project No. 504226141.
