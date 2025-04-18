# Introduction

Turnkey Precision Oncology (TPO) is a scalable and integrated platform for executing precision oncology analysis workflows. TPO implements a number of NGS data analysis [tasks](usage/tasks/overview) for both DNA and RNA sequencing data, such as variant calling ([cords_somatic](usage/tasks/overview)) or fusion detection ([crisp_codac](usage/tasks/overview)). Those tasks can be composed into [pipelines](usage/pipelines/overview) to implement end-to-end workflows. TPO provides functionality to integrate the multi-modal results, a schema to store data in an SQL database, and a patient-level web application to interactively explore the data. 

The tasks are natively executed in a standardized running environment (a docker image) on the Google Cloud Platform [GCP](https://cloud.google.com/), but can also be run locally.

## Structure of the documentation

The documentation is broken into multiple sections covering each element listed above.

1. [Overview](overview) describes how TPO works on a high level.
1. [Setup](setup/summary) provides instruction on installation and configuration.
1. [Usage](usage/cli) provides detailed documentation on how to use TPO.
1. [Output](output/somatic-variants) documentation of TPO outputs and file formats.
1. [Development](development/architecture) instructions on updating and extending TPO.






