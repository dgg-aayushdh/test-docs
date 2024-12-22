Here is a ~complete TPO run example: 

```
# Running DNA Tumor Alignment
tpo.py -config mokefile-unit_test.ini  cords_align SI_30806 SI_30806 gs://tpo-refs/v2/test/fastq/SI_30806_unittest_1.fq.gz  gs://tpo-refs/v2/test/fastq/SI_30806_unittest_2.fq.gz &

# Running DNA Normal Alignment
tpo.py -config mokefile-unit_test.ini  cords_align SI_30807 SI_30807 gs://tpo-refs/v2/test/fastq/SI_30807_unittest_1.fq.gz  gs://tpo-refs/v2/test/fastq/SI_30807_unittest_2.fq.gz &

# Running RNA Tumor Alignment
tpo.py -config mokefile-unit_test.ini  crisp_align SI_9706 SI_9706 gs://tpo-refs/v2/test/fastq/SI_9706_unittest_1.fq.gz  gs://tpo-refs/v2/test/fastq/SI_9706_unittest_2.fq.gz &

wait

# Running DNA Tumor Postalignment
tpo.py -config mokefile-unit_test.ini  cords_postalign SI_30806 gs://mctp-hopkinal/unit_test/base/repo/cords-align/SI_30806 &

# Running DNA Normal Postalignment
tpo.py -config mokefile-unit_test.ini  cords_postalign SI_30807 gs://mctp-hopkinal/unit_test/base/repo/cords-align/SI_30807 & 

wait

# Running DNA Tumor QC
tpo.py -config mokefile-unit_test.ini  cords_misc SI_30806 gs://mctp-hopkinal/unit_test/base/repo/cords-postalign/SI_30806 & 

# Running DNA Normal QC
tpo.py -config mokefile-unit_test.ini  cords_misc SI_30807 gs://mctp-hopkinal/unit_test/base/repo/cords-postalign/SI_30807 & 

# Running RNA Quantification
tpo.py -config mokefile-unit_test.ini  crisp_quasr SI_9706 gs://mctp-hopkinal/unit_test/base/repo/crisp-align/SI_9706 &

# Running RNA Fusion
tpo.py -config mokefile-unit_test.ini  crisp_codac SI_9706 gs://mctp-hopkinal/unit_test/base/repo/crisp-align/SI_9706 &

wait

# Running Variants
tpo.py -config mokefile-unit_test.ini  cords_somatic SI_30806-SI_30807 \
  -alnt gs://mctp-hopkinal/unit_test/base/repo/cords-postalign/SI_30806 \
  -alnn gs://mctp-hopkinal/unit_test/base/repo/cords-postalign/SI_30807 &

# Running Structural
tpo.py -config mokefile-unit_test.ini  cords_structural SI_30806-SI_30807 \
  -alnt gs://mctp-hopkinal/unit_test/base/repo/cords-postalign/SI_30806 \
  -alnn gs://mctp-hopkinal/unit_test/base/repo/cords-postalign/SI_30807 &

wait

# Annotating Variants
tpo.py -config mokefile-unit_test.ini  carat_anno SI_30806-SI_30807  \
  -somatic gs://mctp-hopkinal/unit_test/base/repo/cords-somatic/SI_30806-SI_30807 \
  -structural gs://mctp-hopkinal/unit_test/base/repo/cords-structural/SI_30806-SI_30807 

# Running CNV (with an override)
tpo.py -config mokefile-unit_test.ini \
  -cargs 'CORDS_CNVEX_CAPTURE::/tpo/refs/grch38/capture/onco1500-v6a-targets-ucsc.bed.gz;CORDS_CNVEX_POOL::/tpo/refs/grch38/pools/cnvex-pool-onco1500.v6a-v1.pool.rds;' \
  cords_cnvex SI_30806-SI_30807  \
  -tvar gs://mctp-hopkinal/unit_test/base/repo/carat-anno/SI_30806-SI_30807 \
  -alnt gs://mctp-hopkinal/unit_test/base/repo/cords-postalign/SI_30806 \
  -alnn gs://mctp-hopkinal/unit_test/base/repo/cords-postalign/SI_30807 

# Making Vault
tpo.py -config mokefile-unit_test.ini  carat_vault SI_30806-SI_30807-SI_9706  \
  -anno gs://mctp-hopkinal/unit_test/base/repo/carat-anno/SI_30806-SI_30807 \
  -misc_t gs://mctp-hopkinal/unit_test/base/repo/cords-misc/SI_30806 \
  -misc_n gs://mctp-hopkinal/unit_test/base/repo/cords-misc/SI_30807 \
  -cnvex gs://mctp-hopkinal/unit_test/base/repo/cords-cnvex/SI_30806-SI_30807 \
  -tquasr gs://mctp-hopkinal/unit_test/base/repo/crisp-quasr/SI_9706 \
  -codac gs://mctp-hopkinal/unit_test/base/repo/crisp-codac/SI_9706


```