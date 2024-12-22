To add a capture panel, use the `scripts/add_capture_panel.R` script. You will need:
* The targets bed file of the panel to add
* A (optional) bed file containing regions to exclude (i.e. genotyping positions that you don't want to annotate against) 
* The reference genome fasta file (you have this if you have done `refs_pull`)
* The existing annotation regions bed file (you have this if you have done `refs_pull`)
* A working `picard.jar`


The man page provides instructions on the workflow:
```
$ Rscript scripts/add_capture_panel.R -h
Usage: Rscript add_capture_panel.R [options]
Generate necessary region files to add a new capture panel to TPO. Returns a MISC.interval_list for cords-misc,
 a CNV.bed for cords-cnvex and a ANNO.bed for cords-anno (annotation bed + padded [targets - excluded]).
 All output files are sorted and reduced.


Instructions

- Run this script
- Run 'moke refs_pull'
- copy MISC.interval_list -> refs/<build>/capture/
- copy ANNO.bed -> refs/<build>/custom/
- Bump the ROOT_VER in the mokefile and run 'moke refs_push' and 'moke gcp_root_build'
- CNV.bed -> tpo/rlibs/cnvex/inst/extdata/capture/
- Run 'moke code_build' (version bumping if needed)


Options:
        --targets=TARGETS
                Targets bed file

        --genome=GENOME
                Genome fasta (for assigning contigs)

        --exclude=EXCLUDE
                optional bed file of regions to exclude from annotation (genotyping or backbone SNPs etc.)

        --annotation=ANNOTATION
                existing TPO annotated_regions bed file

        --padding=PADDING
                padding added to both sides of target bed file for annotation (default:50)

        --picard=PICARD
                path to picard.jar

        -h, --help
                Show this help message and exit

Michigan Center for Translational Pathology (c) 2021

```