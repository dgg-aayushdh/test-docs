# Two-stage CNVEX analysis

We will run `cords-cnvex` twice, first in automated mode starting from `BAM` and `VCF` files to make the CNVEX output files (data, model, segmentation) and the results of model search i.e. `somatic-model` / `germline-model`. This output can be investigated in `cnvex-viewer` once an appropriate model is chosen the output can be updated by running cnvex a second time by using the output of the first run as input.


First stage run CNVEX find candidate models

Relevant contents of `full.ini`:
```
## CNVEX
CNVEX_ASSEMBLY = hg38
CNVEX_SETTINGS = exome-pair
CNVEX_CAPTURE = agilent-v4-targets-ucsc
CNVEX_POOL = /tpo/refs/grch38/custom/cnvex-pool-agilent.v4-v3.pool.rds
CNVEX_POPAF = GNOMAD_AF
CNVEX_PROCESS = true
CNVEX_SOMATIC = true
CNVEX_SOMATICSEARCH =
CNVEX_SOMATICPICK = top4
CNVEX_GERMLINE = true
CNVEX_GERMLINESEARCH = --nogrid --nofine
CNVEX_GERMLINEPICK = top1
```
First run, starting from BAM and VCF

```
$ /mnt/share/code/tpo/tpo.py -config full.ini cords_cnvex  --nowait \
    -alnt gs://mctp-ryan/mioncoseq/mCRPC/repo/cords-postalign/SI_24250_CCF6VANXX \
    -alnn gs://mctp-ryan/mioncoseq/mCRPC/repo/cords-postalign/SI_24251_CCF6VANXX \
    -tvar gs://mctp-ryan/mioncoseq/mCRPC/repo/carat-anno/MO_2438-SI_24250.SI_24251 \
    MO_2438-SI_24250.SI_24251
```

Second stage just produce a digest with a chosen model. We disable any process (no need this has already been done). Model can be chosen in cnvex-viewer or elswhere.

```
## CNVEX
CNVEX_PROCESS = false
CNVEX_SOMATIC = false
CNVEX_GERMLINE = false
```

And run CNVEX with the chosen model

```
/mnt/share/code/tpo/tpo.py -config test_quick.ini cords_cnvex --nowait --overwrite \
    -cnvx gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251 \
    -gpick '0.98:1.98' \
    -spick '0.448:2.66' \
    MO_2438-SI_24250.SI_24251
```

Output will look like this:

```
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-config.txt
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-docker.txt
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-germline-digest-00.rds <---- gpick
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-germline-digest-01.rds
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-germline-err
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-germline-model.rds
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-germline-segment.rds
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-opts.rds
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-somatic-digest-00.rds   <---- spick
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-somatic-digest-01.rds   | auto picks
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-somatic-digest-02.rds   |
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-somatic-digest-03.rds
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-somatic-digest-04.rds
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-somatic-err
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-somatic-model.rds
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251-somatic-segment.rds
gs://mctp-mcieslik-data/misc/test/repo/cords-cnvex/MO_2438-SI_24250.SI_24251/MO_2438-SI_24250.SI_24251.rds
```
