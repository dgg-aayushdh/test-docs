# Introduction

Turnkey Precision Oncology (TPO) is a scalable and integrated platform for executing precision oncology analysis workflows. TPO implements a number of NGS data analysis [tasks](Tasks) for both DNA and RNA sequencing data, such as variant calling ([cords_somatic](Tasks#cords_somatic)) or fusion detection ([crisp_codac](Tasks#crisp_codac)). Those tasks can be composed into [pipelines](Pipelines) to implement end-to-end workflows. TPO provides functionality to integrate the multi-modal results, a schema to store data in an SQL database, and a patient-level web application to interactively explore the data. 

The tasks are natively executed in a standardized running environment (a docker image) on the Google Cloud Platform [GCP](https://cloud.google.com/), but can also be run locally.

## Structure of the documentation

The documentation is broken into multiple sections covering each element listed above.

1. [Overview](Overview) describes how TPO works on a high level.
1. [Setup](Setup) provides instruction on installation and configuration.
1. [Usage](Usage) provides detailed documentation on how to use TPO.
1. [Output](Output) documentation of TPO outputs and file formats.
1. [Development](Development) instructions on updating and extending TPO.






