   
# Reference commands

## `refs_pull`
### Purpose

Pull TPO static reference files from GCP bucket to local directory.

### Input
### Output

## `refs_push`
### Purpose

Push TPO static reference files from local directory to GCP bucket.

### Input
### Output

# Code commands

## `code_build`
### Purpose

Install all the necessary R libraries required for R scripts in TPO, and create a snapshot of the TPO code base including bash scripts and R scripts as a docker volume(tpocode) locally.
In config file, `ROOT` must be set to the local code base.

### Input
### Output

## `code_push`
### Purpose

Push code docker volume(tpocode) to the Google Container Registry (GCR) in the cloud.

### Input
### Output

## `code_pull`
### Purpose

Pull code docker volume from the GCR.

### Input
### Output

# Docker images build and deployment

## `boot_build`
### Purpose

Build TPO docker images locally.

### Input
### Output

## `boot_push`
### Purpose

Push TPO
### Input
### Output

## `boot_pull`
### Purpose
### Input
### Output



# GCP images build and deployment

## `gcp_boot_build`
### Purpose
### Input
### Output

## `gcp_root_build`
### Purpose
### Input
### Output

## `gcp_root_push`
### Purpose
### Input
### Output

## `gcp_root_pull`
### Purpose
### Input
### Output

# Debugging and testing

## `gcp_instance`
### Purpose
### Input
### Output

## `gcp_configure`
### Purpose
### Input
### Output

## `template`
### Purpose
### Input
### Output
