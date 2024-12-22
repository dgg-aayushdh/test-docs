# Miscelaneous

#### Somatic/germline variable meaning
* ref - Reference allele; only 1 returned per line 
* alt - alternative allele only 1 returned per line
* context - trinucleotide context of mutation
* sbs96 - strand converted mutation format
* TRANSCRIPT - assigned based on external information; prioritizes assignment to transcript with maximal potential harm
* EXON - assigned based on external information; prioritizes assignment to transcript with maximal potential harm
* IMPACT - can be low, moderate, high; protein changing variants moderate/high; typically only interested in moderate/high variants. Not fool proof as some synonymous variants are high impact but not shown as so. TERT/TP53 impacted post-hoc
* SIFT - algorithm output that predicts what should happen to a protein. "mostly useless" per marcin
* Warnings - not currently used
* HGVS_Equivalent - lists all transcripts for which mutation does the same thing
* AFT - allele frequency in the tumor (ADT/DFT); calculation not perfect as ADT weighted based on read quality; 2% cutoff ~minimum 
* DPT - depth in tumor ( total coverage )
* AFN - allele frequency in normal
* DPN - depth in normal
* ADT_FWD - in paired end sequencing, Forward read support; should be in equal proportion to ADT_REV; can be skewed in low quality input but in general expect at least 1 supporting read in ADT_REV
* ADT_REV - in paired end sequencing, Reverse read support; should be in equal proportion to ADT_FWD ; can be skewed in low quality input but in general expect at least 1 supporting read in ADT_FWD
* sor - strand odds ratio; sophisticated measure of ADT_FWD/ADT_REV; ideal is ~1
* str - indel mutation occurs in a tandem repeat and furthers repeat
* str_ru - the repeating unit
* str_len - how many repeating units start
* str_diff - how many repeating units were added by variant
* ecnt - number of different variants considered w/in a region (exon?); measure of noisiness of variant region; ecnt>10 likely junk while ecnt ~3 likely true
* tlod - evidence that variant is in the tumor; IMPORTANT;  core output of TNscope; higher the better
* nlod - evidence that variant is in ABSENT in normal; IMPORTANT;  core output of TNscope; higher the better
* flod - (nlodf) - evidence variant is not in the normal assuming if it was it would be at same frequency in tumor
* xfet - fischer's exact test using ADT/ADN; probability different between tumor and normal
* sor - strand odds ratio
* mqrs - map quality rank sum; GATK parameter; compares quality of tumor reads to normal reads
* mqs - map quality score; average or median quality of mapped read; above 30 is good, above 27 is generally fine
* oxog - calculates oxidized guanines; TCGA legacy
* dbsnp - whether variant in dbsnp; contains germline and somatic variants both inherited and de novo. Also contains sequencing errors. Generally skewed towards germline (~95%)
* clinid- id of variant in clinvar; clinvar contains information on clinical impact of variants; can be germline or somatic. If pathogenic/likely pathogenic the variant should not be ignored
* clinvar - clinvar annotation
* cosmic_cnt - annotates variants relative to cosmic database; in how many cancer samples was this mutation seen; contains artifacts
* gnomad_af - database of healthy individuals rigorously filtered; allele frequency; high quality; higher frequency lower odds pathogenic
* kg_af - same thing as gnomad but based on 1000 samples
* pon - from genomic data commons; annotated from GDC as whether present in normals. If not blank then found by GDC
* rmsk_hit - repeat masker hit; is variant in repetitive region of genome; in general a nonzero value here indicates a questionable variant d/t challenge of aligning in repetitive regions
* Tn - proportion of tumors that have this variant; based on mioncoseq samples; identify low quality FFPE samples
* Nn - proportion of normals that has this variant; based on mioncoseq samples; identify low quality FFPE samples
* Taf90 -  90%ile allele frequency in the tumor; based on mioncoseq samples
* Naf90 - 90%ile allele frequency in the normal; based on mioncoseq samples