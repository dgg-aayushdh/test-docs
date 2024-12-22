# Overview

Following [installation](Installation), a series of steps are required to make TPO operational. These steps involve the downloading of reference files and the creation of Docker and GCP images. Some configuration is necessary to tell TPO how to perform these steps, like where to save files (references, temporary, results), which GCP project to use, and how to authorize the Sentieon tools.

In addition, the computational pipelines can be fully customized and configured, how to change their settings is described later in the [Pipelines Settings](Settings) section.

# TPO config files

TPO is configured through only a single configuration file. The format of the configuration file follows the Python `configparser` standard, which is based on the Microsoft Windows INI format see: [configparser module](https://docs.python.org/3/library/configparser.html). The config file, typically using the `.ini` or `.config` extension is divided into several sections. 

Since most of the settings are required but will rarely be modified by the user, the general strategy is to start with one of the provided templates and modify it as needed.

```
$ cp config/mokefile_grch38-tpo.template config/my_config.ini
```

However, a few settings, denoted `~CUSTOM~` in the template files will need to be adjusted by all users.
More examples of config files can be found on https://github.com/mctp/tpo-mctp-config, different versions of config files are applied in different scenarios. See how the field '~CUSTOM' are filled in those examples.

This config file should now be provided to the TPO CLI using the `-config` parameter in order to run TPO:

```
$ $TPO_ROOT/tpo.py -config config/my_config.ini
```

The file sections in the config file are separated into two parts: `[TPO]`, `[SECRETS]`, `[RUNTIME]`, `[GCP]` are controlling general settings such as the details of the cloud environment, which reference files should be used; the rest is configuring computational pipelines. A single file can support both local and GCP use-cases.

# General settings

| Section   | Purpose |
|-----------|---------|
| [TPO]     | settings specify TPO versions |
| [SECRETS] | secrets such as passwords or security tokens |
| [RUNTIME] | runtime settings and locations |
| [GCP]     | Google Cloud Platform Settings |

## [TPO] Asset versioning

First we need to edit the `~CUSTOM~` fields in the `[TPO]` section. The three parameters `CODE_VER`, `BOOT_VER`, and `ROOT_VER` are arbitrary strings to version GCP disk images and Docker images created and used by TPO. For example, `0.0.1-rel1` could be used and incremented whenever an update of the images is needed.

- `ROOT_VER` controls the version of the `tporoot` disk image on GCP. This non-executable image contains reference files and indexes to run TPO on the cloud. When running locally those contents are stored in the `[RUNTIME]` `ROOT` directory.
- `BOOT_VER` controls the version of the 4 executable TPO Docker images.
- `CODE_VER` controls the version of the TPO code Docker images. The contents of this image (i.e. the TPO code embedded within the image) are used when running any TPO pipelines on the cloud. `CODE_VER` serves no function when running TPO locally, in that instance the `$TPO_ROOT` directory cloned from git is used.

An example `[TPO]` section could look as follows:

```
[TPO]
CODE_VER = 0.0.1-rel1
BOOT_VER = 0.0.1-rel1
ROOT_VER = 0.0.1-rel1
````

`REFS_VER` is the location on a bucket with all the reference files which will be put into ROOT_VER

## [SECRETS] Sentieon and AWS

The `SENTIEON_LICENSE` variable controls the `ip-address:port` of the Sentieon license server or a license file. Please refer to the Sentieon documentation for details on how to set up a [license server](https://support.sentieon.com/quick_start/index.html#appendix-set-up-license). If you choose to use a license file, please refer to the [installation instructions](Installation#LicenseFile).

```
[SECRETS]
SENTIEON_LICENSE = ~CUSTOM~
```

TPO understands the following secret variables in addition to the `SENTIEON_LICENSE`. While both variables can be stored in the config file (in the [SECRETS] section) this is not recommended as this exposes them in plain text. It is recommended to provide (in a secure way) at runtime via the `-cargs` argument to `./tpo.py`.

```
[SECRETS]
AWS_ACCESS_KEY_ID = <key id>
AWS_SECRET_ACCESS_KEY = <secret access key>
```

The two variables allow TPO to access files stored in AWS buckets. If provided TPO can access `s3://` read from directories like it would from `gs://`` buckets. Writing results to AWS buckets is not supported.

## [RUNTIME] Local mode

The majority of settings in the `[RUNTIME]` config file section control how `TPO` uses various locations on the local computer to store various files and assets. Importantly, the user (and root) must have read-write access to all directories in this section. The optional settings `TEMP`, `RUNS`, `REFS` should be left blank to use defaults.

```
ROOT = required directory where TPO will store a bundle of references, tools, libraries necessary for the execution of pipelines.
WORK = required directory to store TPO results including TEMP, RUNS, REFS below. 
TEMP = optional (default: $WORK/tmp), directory to store temporary files (best to use fast SSD-backed storage)
RUNS = optional (default: $WORK/runs), directory to store result files 
REFS = optional (default: $WORK/refs), directory to store required reference files and assets (can be slower NFS storage)
```

## [GCP] Remote mode

Settings in the `[GCP]` and `[RUNTIME]`config section controls how TPO interacts with the GCP.

Settings which control how GCP should be used:

```
[GCP]
PROJECT = <name of GCP project user has access to>
LOCATION = <which location the computation will take place e.g. us-central1>
ZONE = <which zone the computation will take place e.g. us-central1-f>
NETWORK = <name of the network the TPO GCP instances will be started in>
SUBNET = <name of the subnet the TPO GCP instances will be started in>
TAG = <TODO>
SERVICE_ACCOUNT = <name of the service account to run the TPO GCP instances typically xxxx-compute@developer.gserviceaccount.com>
```

Settings which control what happens at runtime (pipeline execution):

```
[RUNTIME]
WORK_BUCKET = mctp-tpo/test-tpo
WORK_DONE = <fail or skip> what should happen if running TPO would result in overwriting, to overwrite `--overwrite` needs to be provided.
```
 
# Pipeline configuration

```
[BCL] BCL settings and parameters 

[CORDS] Settings related to DNA analyses including
- Alignment
- Somatic variant calling
- Germline variant calling
- Structural variant calling
- Copy-number variant calling
- Miscellaneous analyses

[CRISP] Settings related to RNA analyses
- Alignment
- Germline variant calling
- Alignment-free quantification
- Alignment-based quantification
- Fusion detection

[CARAT] Settings related to variant annotation

[TEST] Settings useful during debugging and testing
```

| Version    | Purpose |
|------------|---------|
| [CODE_VER] | Code docker image version (default: commit id) |
| [REFS_VER] | Refs GCP image version |
| [BOOT_VER] | Boot GCP image version |


