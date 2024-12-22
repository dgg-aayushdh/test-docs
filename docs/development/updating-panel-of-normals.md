Create a multi-sample cohort BCF file `<COHORT_BCF>` out of multiple unfiltered VCFs from `cords-somatic`. The purpose of the panel is to quantify noisy positions as detected by `TNScope` therefore the input VCF files should be `-somatic-tnscope.vcf.gz`. Using multiple threads `<T>` will significantly speed-up this process.

```
bcftools merge --threads <T> -Ob -o <COHORT_BCF> -m both <TNSCOPE SOMATIC VCFs>
bcftools index --threads <T> <COHORT_BCF>
```

The `-m both` tells `bcftools` to create separate multi-allelic records for indels and SNPs, which will be useful later as their error profiles are very different. No filtering should be applied to this file.

Cross-tabulate tumor-normal alt-allele allele-depth `AD` counts. This creates a new `INFO/ADX` field. Which represents and `n+1 * n+1` matrix tabulating, where `n` is the maximal tabulated `AD` in the tumor and normal samples. The script is relatively slow, but can be executed in parallel.

```
parallel -j <THREADS> ad-crosstab.py -chrom {} <COHORT_BCF> ADX_{}.vcf ::: chr{1..22} chrX chrY
```

Compress using `bgzip` and index using `bcftools`

```
parallel -j <THREADS> bgzip {} ::: PON_*vcf
parallel -j <THREADS> bcftools index {} ::: PON_*vcf.gz
```

Concatenate VCF files into a single PON

```
bcftools concat --threads <THREADS> -o PON.bcf -Ob PON_chr{{1..22},X,Y}.vcf.gz
bcftools index--threads <THREADS>  PON.bcf
```
