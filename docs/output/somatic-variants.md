## Somatic Variants
The somatic variants appear in the vault under `v$tables$somatic` and contain the following fields

| Field | Class | Description|
|-------|-------|------------|
triage|character | ?? |
id|character | name of analysis (from cords-somatic) |
var_id|character | chr:pos_ref/alt |
chr|character | chromosome |
pos|integer | genomic position |
ref|character | reference allele |
alt|character | alternate allele |
context|character | 3 base context for SNV |
sbs96|character | 3 base context for SNV - sbs format |
gene_name|character | SYMBOL of gene |
gene_id|character | ENSG Gene ID |
TRANSCRIPT|character | ENST (or other) transcript ID |
EXON|character | exon #/#of exons |
HGVSc|character | DNA change |
HGVSp|character | protein change |
Consequence|character | from VEP?  |
IMPACT|character | from VEP |
SIFT|character | SIFT score |
Warnings|character | ?? |
HGVS_Equivalent|character | other transcripts for which the HGVSp would be the same |
AFT|numeric | Allele Frequency Tumor |
AFN|numeric | Allele Frequence Normal |
ADT|numeric | Alt Dept Tumor |
DPT|numeric | Depth Tumor |
ADN|numeric | Alt Depth Normal |
DPN|numeric | Depth Normal |
ADT_FWD|integer | Alt depth in FWD direction |
ADT_REV|integer | Alt depth in REV direction |
str|logical | short tandem repeat? |
str_ru|character | repeat unit |
str_len|integer | repeat length |
str_diff|integer | ?? |
ecnt|integer | # of mutations in region ? |
tlod|numeric | log odds variant is in the tumor |
nlod|numeric | log odds the variant is not in the normal |
flod|numeric | log odd the variant is not in the normal given the AF in the tumor |
xfet|numeric | Fisher's Exact Test p-value ref/alt tumor/normal |
sor|numeric | Strand Odds Ratio |
mqrs|numeric | Map Quality Rank Sum |
mqs|numeric | Map Quality Sum |
oxog|numeric | likelihood of oxoG  |
dbsnp|integer | dbSNP ID |
clinid|integer | CLinVar ID |
clinvar|character | ClinVar count |
cosmic_cnt|integer | COSMIC count |
gnomad_af|numeric | gnomAD Pop Allele Frequency |
kg_af|numeric | 1000G Pop Allele Frequency |
pon|character | ? |
rmsk_hit|character | RepeatMasker Hit |
is.indel|logical | is it an InDel? |
sblr|numeric | ?? |
all.pon|integer | ?? |
pid|character | ?? |
vcf_filter|character | ?? |
ccf|numeric| Percentage of cancer cells harboring this SNV |
m|integer| Number of Alleles in each tumor cell harboring this SNV |

