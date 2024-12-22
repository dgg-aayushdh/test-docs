### Purpose: 
The purpose of this page is to introduce a reliable and reproducible means of QC'ing variants and features for metavaults. Specifically this page will outline how:
1.  Variants can be manually examined and re-triaged following use of the default triage functions.
2.  Samples can be classified on the feature level (i.e. SPOP-mutant, WNT-altered, etc.)

The functions that the code below leverages are stored primarily in `/tpo/rlibs/carat/R/review.R` and `/tpo/rlibs/carat/R/triage.R`




### Variant-level review


1.  Load and run default triaging
```
# Load/triage
#########
ddtmv <- qs::qread('/path/to/metavault/dtmv.rds')
ddtmvf <- vaultTriage(dxv = ddtmv)
#########
```


2.  Export pertinent data types for feature of interest for manual review.
```
# Export for review
#########
## choose gene
g <- 'SPOP'
## compute
som <- getReviewTableSomatic(ddtmvf, g)
grm <- getReviewTableGermline(ddtmvf, g)
str <- getReviewTableStructural(ddtmvf, g)
fus <- getReviewTableFusion(ddtmvf, g)
## save
write.table(som, '/save/path/spop_som.txt', row.names = F, col.names = T, quote = F, sep = '\t')
write.table(grm, '/save/path/spop_grm.txt', row.names = F, col.names = T, quote = F, sep = '\t')
write.table(str, '/save/path/TEMP/spop_str.txt', row.names = F, col.names = T, quote = F, sep = '\t')
write.table(fus, '/save/path/spop_fus.txt', row.names = F, col.names = T, quote = F, sep = '\t')
#########
```


3.  Manually review externally (google sheets/excel). 
** NOTE: if/when exporting structural table to google sheets, you *MUST* change the default format of all the cells to text. The 'default' format changes the var_ids of the structural table and mucks everything up.


4.  Load the manually review tables and combine into stacked table   
```
# Modify
#########
## load annotated tables 
som <- fread('/save/path/spop_reviewed_som_2023-08-10.tsv')
grm <- fread('/save/path/spop_reviewed_grm_2023-08-10.tsv')
str <- fread('/save/path/spop_reviewed_str_2023-08-10.tsv')
fus <- fread('/save/path/spop_reviewed_fus_2023-08-10.tsv')
## make
som <- formatReviewTable(som, FIELD='somatic')
grm <- formatReviewTable(grm, FIELD='germline')
str <- formatReviewTable(str, FIELD='structural')
fus <- formatReviewTable(fus, FIELD='fusion')
## combine
stacked <- rbindlist(list(som,grm,str,fus))
## save
write.table(stacked, '/save/path/spop_stacked.txt', row.names = F, col.names = T, quote = F, sep = '\t')
#########
```


5.  Append the stacked table to any existing stacked tables you have. I recommend keeping a single table for all the variants you've modified.  It's easy to copy paste the tables onto a single stacked table stored in google docs/excel. This can readily be used in the following step to change the vault. Then, once you've manually reviewed all the genes you want, you'll have a single stacked table which you can use to easily re-triage the variants in your vault based on your careful manual review. 
```
# Update
#########
## load stacked table
TBL = fread('/save/path/stacked_variants.tsv')
## re-triage
ddtmvf <- dxRescueVariants(ddtmvf, TBL)
#########
```



### Feature-level review

1.  Manually review on the feature level. This is where you can review the evidence across data types with your manually reviewed variants to make feature level calls (i.e. APC-mutant, HRD-mutant).  
```
# export
#########
g_spop <- reviewFeatures(ddtmvf, g)
write.table(g_spop, '/save/path/spop_gl.txt', row.names = F, col.names = T, quote = F, sep = '\t')
#########
```


2.  Manually review the created table.


3.  Merge this table as you see fit with the metadata of your metavault.