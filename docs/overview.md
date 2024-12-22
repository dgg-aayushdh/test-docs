# Overview

## Glossary

- **`Task`** is the unit of TPO execution implementing a data processing or analysis function.
- **`Pipeline`** is a series of interdependent tasks together implementing an NGS analysis workflow.
- **`Local`** refers to the machine on which the user runs the TPO CLI.
- **`Cloud`** refers to the Google Cloud Platform (GCP).
- **`Remote`** refers generally to infrastructure (compute, storage) on the GCP.
- **`Docker Image`** TPO uses docker images to manage the environment and software in which all tasks are executed.
- **`GCP Image`** TPO uses GCP images to create disk volumes to boot configured instances or provide reference files.
- **`GCP Instance`** Virtual machine hosted on Google's infrastructure running a TPO `GCP Image`.

## Basic example

TPO enables users to run genomics analyses on the cloud (or locally) using simple command line invocations. For example, the following command would run somatic variant calling in a TPO GCP instance.

```
tpotask.sh -project <project.ini> cords_somatic \
    run1 -tsample tumor -nsample normal -alnt <tumor alignment> -alnn <normal alignment>
```

Breaking the command into parts we have:

- `$TPO_ROOT` is the location of the TPO root directory as cloned from git, typically added to `$PATH`.
- `tpotask.sh` is the main CLI application to launch TPO tasks.
- `<project.ini>` is a configuration file that controls all aspects of how TPO works locally or in the cloud.
- `cords_somatic` is the name of the TPO somatic variant calling task that will be launched.
- `run1` is the user-specified ID of this pipeline execution (run) and `-tsample tumor -nsample normal -alnt <tumor alignment> -alnn <normal alignment>` are the other parameters controlling the inputs and outputs for the chosen task `cords_somatic`.

At the very high level, what happens after a TPO task is executed is simple:

1. An instance is launched on GCP.
2. The instance downloads input files from a cloud storage bucket on GCP, runs genomic analyses, uploads output results to a bucket.
3. The instance terminates.

All the necessary images are created in local using the configuration files which are then pushed to the GCP. Hence, all the task executions are actually being performed in the cloud and not in our local. One of the reasons behind it is because these tasks are very resource heavy which require powerful machines if we are to run in out local computers. The application uses docker images for tasks that need to be performed. These images are pushed to the GCR (Container Registry) which will be installed in the new instance that gets created. Once the task is completed, the output is uploaded in the GCS(Google Cloud Storage). 
## Configuration

TPO is configured through one or multiple configuration files using the `INI` format. Since every aspect of TPO can be configured it is helpful to use multiple configuration files each controlling a specific aspect of TPO function. Defaults are provided for a number of
reference genomes.
