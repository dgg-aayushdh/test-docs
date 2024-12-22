# Getting help from the CLI

The task CLI is implemented using `moke` and is self-documenting. The following command can be used obtain up-to-date help for each of the tasks:

```
tpotask.sh <task_name> -h
```



# `bcl`
### Purpose

Runs Illumina `bcl2fastq` to generate `fastq` files.

### Inputs
* `id` Unique run identifier (e.g. "CDMHAANXX")
* `tar` A tar archive of the instrument's output.
* `lib` A tab delimited text file containing the Flowcell / Library / Lane / Barcode information, e.g.
```
Flowcell	Library	Lane	Barcode
HCKNVDRXX	SI_25872	1	C1[Usage](Usage)
HCKNVDRXX	SI_25873	1	C2
HCKNVDRXX	SI_25874	1	C3
HCKNVDRXX	SI_25875	1	C4
HCKNVDRXX	SI_25876	1	C5
HCKNVDRXX	SI_25877	1	C6
```
### Output
In `$WORK_BUCKET/repo/bcl/<id>`:
* Renamed `fastq` files
* `config.txt` containing the run parameters 

# `cords`
## `cords_align`
### Purpose

Align short-read DNA data to a reference genome.

### Input
* `id` Unique run identifier 
* `sample` What to put in the `SM` of read group, important for variant calling.
* `fq1` and `fq2` The `fastq` files, possibly from `bcl` output
### Output
In `$WORK_BUCKET/repo/cords_tnscope/<id>`:
* `<id>-aligstat.txt` The alignment statistics (like Picard `AlignmentSummary`)
* `<id>.cram` The aligned file
* `<id>-genotype.csv` and `<id>-genotype.csv` Results of the genotyping
* other summary files (open-source version outputs four additional figures to sentieon version, i.e. `meanqual.pdf`, `qualdist.pdf`, `gcbias.pdf` and `isize.pdf`)
### Note for open-source version
* Indel realignment is no longer supported by GATK4 since 2016 when GATK was versioned 3.6. This step has not been required by HaplotypeCaller or Mutect2 because they implement a more sophisticated and effective form of realignment. See the [post](https://github.com/broadinstitute/gatk-docs/blob/master/blog-2012-to-2019/2016-06-21-Changing_workflows_around_calling_SNPs_and_indels.md?id=7847).
* Only calculate BQSR table, but may not plot it.
### Example
```
/mctp/users/rebernrj/projects/tpo/mokefile.py -config /mctp/users/rebernrj/projects/tpo-mctp-config/projects/mCRPC/v1.4/mokefile_onco1500v6a_grch38-tpo_1.4.txt cords_align SI_31707_H5L3CDRXY_1 SI_31707 gs://mctp-fastq/H5L3CDRXY/mctp_SI_31707_H5L3CDRXY_1_1.fq.gz gs://mctp-fastq/H5L3CDRXY/mctp_SI_31707_H5L3CDRXY_1_2.fq.gz --nowait --preemptible -work_disk 500G
```
## `cords_unalign`
### Purpose

Create `fastq` files from a BAM file (lossless).

### Input
* `id` Unique run identifier
* `aln` Input BAM file
### Output

## `cords_postalign`
### Purpose

Deduplicate, post-process and optionally combine multiple `cords-align` inputs into one `bam` file ready for variant calling and other downstream analyses.

### Input
* `id` Unique run identifier
* `aln` Input `cords-align` folders.
### Output
### Example
```
# single aligned file
/mctp/users/rebernrj/projects/tpo/mokefile.py -config /mctp/users/rebernrj/projects/tpo-mctp-config/projects/mCRPC/v1.4/mokefile_onco1500v6a_grch38-tpo_1.4.txt cords_postalign SI_31707_H5L3CDRXY gs://mctp-ryan/mioncoseq/mCRPC/repo/cords-align/SI_31707_H5L3CDRXY_1/ --nowait --preemptible

# multiple aligned files
/mctp/users/rebernrj/projects/tpo/mokefile.py -config /mctp/users/rebernrj/projects/tpo-mctp-config/projects/mCRPC/v1.4/mokefile_agilentv4_grch38-tpo_1.4.txt cords_postalign SI_30539 'gs://mctp-data/internal/repo/cords-align/SI_30539-HT2KGDMXX-1/;gs://mctp-data/internal/repo/cords-align/SI_30539-HT2KGDMXX-2/' --nowait --preemptible

```
## `cords_misc`

Miscellaneous analyses for quality control and evaluation of DNA post-alignments.

### Purpose 
### Input
### Output

## `cords_germline`
### Purpose 
Calls variants from a tumor and optional normal sample pair.
### Input
### Output

## `cords_somatic`
### Purpose 
Calls variants from a tumor and optional normal sample pair.
### Input
### Output

## `cords_structural`
### Purpose 
Calls structural variants from a tumor and optional normal sample pair.
### Input
### Output

## `cords_cnvex`
### Purpose

Identify copy number alterations from whole-genome, whole-exome or targeted panel sequencing.

### Input
* `id`  Unique run identifier
* `alnn` and `alnt` The outputs of `cords-postalign` for normal and tumor
* `tvar` or `gvar` The output of `cords-somatic`
* `cnvx` An existing `cords-cnvex` run (alternative)

### Output
In `$WORK_BUCKET/repo/cords_cnvex/<id>`:
* A `model` object which is a list of 4 objects with a complex structure.

### Usage

The operation of `cords_cnvex` is controlled by multiple parameters in the TPO `config` file, the CNVEX settings file, and which of the inputs were provided. In general, CNVEX tries to do as much as possible with the data provided. CNVEX can do somatic (tumor) and/or germline (normal) copy-number analysis. For germline analysis a germline BAM file needs to be provided as `alnn`. VCF variant files can originate either from targeted sequencing (e.g. whole-exome, `tvar`) or genomic sequencing (`gvar`) and can contain either one sample in tumor-only mode or two samples paired (tumor-normal) analysis mode. In tumor-only mode CNVEX can take advantage of a pool of unrelated normal samples (which needs to be constructed separately), can also be used for the analysis of the normal sample.

The CNVEX algorithm consists of three stages:

1. Input data processing. At this step the files provided in through `alnt`, `alnn`, `tvar`, and `gvar` are transformed into a `cnvx` object. Only the `alnt` input is required all other are optional. However, if no variants are provided (i.e. `tvar` or `gvar`) only a limited analysis will be performed. Alternatively, if none of the input `BAM` and `VCF` files are provided, but a previous `cnvx` run is provided it will be re-processed. This allows previous CNVEX results to be updated using a newer code version or new settings.
2. Data segmentation. At this stage the genome is being partitioned into segments of (hopefully) constant copy-number (`C`) and B-allele frequency (`BAF`).
3. Copy-number purity/ploidy model search. This step comprises of data segmentation and model search. This process proceeds separately for the somatic (tumor) and germline (normal) component of the data. It results in multiple files being created.
 
# `crisp`

CRISP stands for 'Clinical RNA Sequencing Interpretation Platform' it comprises of a comprehensive set of pipelines for the analysis of short-read paired-end RNA-seq data.

CRISP performs three separate alignments to maximize the number of aligned chimeric reads used for fusion calling by CODAC. The first alignment, resulting in "*alig.cram" disables chimeric alignment all together, as a result this file only contains linearly-aligned reads. This file is used for gene expression quantification, QC, etc. The two other files disable linear alignment and perform chimeric alignment. Two chimeric files are created because STAR is run separately on overlapping read pairs (SE - long single end reads derived from overlapping paired-end reads), and non-overlapping read pairs (PE).
IMPORTANT: none of the CRISP produced files contains all the reads (they are split into alig, chim-pe, chim-se, unaligned), I strongly recommend using FASTQ for long-term storage and exchange

## `crisp_align`
### Purpose 

Align paired-end short RNA-seq data to a reference genome.

### Input
### Output

Outputs include:

* <FILE_ROOT_NAME>-alig.cram - Linearly aligned reads. This output results from a process that disables chimeric alignment all together, as a result this file only contains linearly-aligned reads. Index files are also created as *bai and *crai as well as an associated log file as *log.
* <FILE_ROOT_NAME>-chim-pe.cram - Chimeric alignment results from non-overlapping read pairs. Index files are also created as *bai and *crai. Index files are also created as *bai and *crai as well as an associated log file as *log.
* <FILE_ROOT_NAME>-chim-se.cram - Chimeric alignment results from overlapping read pairs. Index files are also created as *bai and *crai as well as an associated log file as *log.
* <FILE_ROOT_NAME>-unmapped_1.fq.gz - Unmapped FASTQ from direction 1.
* <FILE_ROOT_NAME>-unmapped_2.fq.gz- Unmapped FASTQ from direction 2.

## `crisp_codac`
### Purpose

Identify Fusions from aligned RNA-seq data.

### Input
### Output

## `crisp_quant`
### Purpose

Quantify RNA abundance using alignment-free techniques.

### Input
### Output

## `crisp_quasr`
### Purpose

Quantify RNA abundances from short-reads using standard alignment techniques.

### Input
### Output

## `crisp_germline`
### Purpose

Call variants in aligned RNA-seq data implementing the GATK4 strategy.

### Input
### Output


# `carat` 

CARAT stands for 'Comprehensive Annotation and Reporting of Aberrations in Tumors' it comprises pipelines which interpret, annotate, filter, summarize and aggregate genomic results as standardized outputs.

## `carat_anno`
### Purpose

Annotation and interpretation of variants detected by any of the `cords` pipelines.

### Input
### Output

## `carat_report`
### Purpose

Aggregation and standardized reporting of results from `cords`, `carat`, and `crisp` pipelines.

### Input
### Output

