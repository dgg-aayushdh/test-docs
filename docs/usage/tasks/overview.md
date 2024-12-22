## Overview

The TPO commands fall into two main categories `pipeline` commands, and `infrastructure` commands (see list below). The `pipeline` commands execute TPO analysis pipelines given some input data and save the outputs in a GCP bucket. The `infrastructure` commands manage references, docker / GCP images, and used mostly in the [build](Build) and [development](Development) process.

In general, the `pipeline` commands are configured (via a config file) and provided with inputs (as a parameters). The `infrastructure` commands are also configured via the same config file, but typically do not have data inputs.

## Pipeline commands

```
    ## BCL
    bcl                 Execute BCL pipeline
    ## DNA analysis
    cords_align         Execute CORDS ALIGN pipeline
    cords_unalign       Execute CORDS UNALIGN pipeline
    cords_postalign     Execute CORDS POSTALIGN pipeline
    cords_somatic       Execute CORDS SOMATIC pipeline
    cords_germline      Execute CORDS GERMLINE pipeline
    cords_cnvex         Execute CORDS CNVEX pipeline
    cords_structural    Execute CORDS STRUCTURAL pipeline
    cords_misc          Execute CORDS MISC pipeline
    ## RNA analysis 
    crisp_align         Execute CRISP ALIGN pipeline
    crisp_quasr         Execute CRISP QUASR pipeline
    crisp_codac         Execute CRISP CODAC pipeline
    crisp_germline      Execute CRISP GERMLINE pipeline
    crisp_quant         Execute CRISP QUANT pipeline
    ## annotation and reporting
    carat_anno          Execute CARAT ANNO pipeline
    carat_report        Execute CARAT REPORT pipeline   
```

## Infrastructure commands

```
    ## Modifying and updating references
    refs_pull           Pull references from GCP
    refs_push           Push references to GCP
    ## Code development
    code_build          Build TPO Code Images
    code_push           Push code image to GCR
    code_pull           Pull code image from GCR
    ## Docker images build and deployment
    boot_build          Build TPO Boot Images
    boot_pull           Pull TPO Boot images from GCR
    boot_push           Push TPO Boot images to GCR
    ## GCP images build and deployment
    gcp_boot_build      Build boot GCP image/disk
    gcp_root_build      Build root GCP image/disk
    gcp_root_push       Push root/tpo from image to GS
    gcp_root_pull       Pull root/tpo from GS to local
    ## Debug and testing
    template            Execute TEMPLATE pipeline
    gcp_instance        Start GCP instance to develop pipelines
    gcp_configure       Configure needed GCP resources (TODO)

```

## Overview of pipeline structure
![tpo_flowchart](https://user-images.githubusercontent.com/19393296/167164207-afd721ec-a2f1-4234-8991-be6a12fc717b.jpg)

## Modifying pipeline command parameters
Parameters used in the pipeline commands are set within the config files. Instructions for how to set these default parameters are discussed [here](https://github.com/mctp/tpo/wiki/Configuration). However, there are instances where we wish to modify pipeline commands for a subset of samples and do not wish to make an entirely new config file to do this. In this instance we can modify these paramaters by passing them when we call TPO. An axample for how to do this during Crisp-Quasr is shown below.

```
../../mokefile.py -config ../../mokefile_agilentv4_grch38-tpo_1.4.txt crisp_quasr SI_26995 ../../repo/crisp-align/SI_26995-CDMJCANXX-4/ --nowait --preemptible -cargs 'QUASR_MIXCR::false;QUASR_COUNTGTF::/../refs/grch38/ensembl/grch38.97.clean.gtf'
```

In this case, we make two amendments to the config file - the first modifying the QUASR_MIXCR parameter and the second the QUSR_COUNTGTF parameter. These parameters are separated by a semicolon and encolsed with single quotes. '-cargs' is used to indicate the parameters are being passed. 



