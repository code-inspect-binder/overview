# Overview
# OSF Project Execution Results Summary

This repository documents the execution results of a reproducibility pipeline applied to **296 OSF-hosted projects** containing R code.

---

## 1. Initial Verification: File Not Found Analysis

An initial verification step revealed that, out of the 296 OSF projects, **32 projects** had missing files, see [File Not Found Projects](results/file_not_found_projects.txt) for the list. 
This outcome suggests that a portion of the dataset had become outdated, likely due to file deletions, file renaming, or the projects no longer being available on OSF. 

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
  In these cases, the **Reason** field provides a high-level classification of the issue.

---

## 6. Error Analysis

To analyze and interpret script execution failures, a two-level classification approach was implemented:

- **Level 1:** Regular expressions were used to extract common error messages.
- **Level 2:** Errors were grouped into broader semantic categories where possible.

The main error categories were:

- **Missing Object or Function:** 
  - "object not found," "could not find function," or "unexported object access" errors.
- **Invalid File or Directory Path:** 
  - "cannot open file," "failed to search directory," or "directory not found."
- **Missing Package:**
  - Missing explicit or implicit library dependencies.
- **Package Installation Failure:**
  - Installation errors such as "lazy loading failed" or "package namespace load failed."
- **Shared Library Load Error:**
  - Failures related to missing system-level shared objects.
- **File Read Error:**
  - Failures when attempting to read datasets or compressed files.

Some specific errors occurred infrequently, but were still recorded individually. These included:

- **RStudio Environment Errors**  
- **Compressed File Not Found**
- **Syntax or Argument Errors**  
- **Encoding and String Handling Issues**  
- **Missing Arguments in `setwd()`**  
- **Data Structure Mismatches**  

---

## 7. Notes

- Missing projects and file mismatches could be due to a lack of formal version control on OSF.
- Failed builds often highlight real-world challenges in open science reproducibility.
- All Docker images are standardized for long-term use and reproducibility.

---
