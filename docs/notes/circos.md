Demo code to make circos plots from (meta)Vaults


Load the vaults and make a metavault

```
devtools::load_all('./rlibs/carat/')
l <- list.files('./vaults', full.names=T)
vaults <- lapply(l[5:10], readRDS)
mv <- metaVault(vaults)
```

Make and save a single plot and save pdf
```
#one vault, saving to <vault$meta$id>.pdf
circosPlot(vaults[[1]],savef=TRUE) # saves 1 pdf
```

![image](circos_demo.png)

Setup to filter 

```
genes <- 'grch38.108.clean.gtf'
settings <- system.file(sprintf("extdata/settings/%s.R", 'default'), package="carat")
.args <- list( annotation=list( gene_model=genes ) )
cfg <- caratConfig(settings, .args)
anno <- vaultAnno(cfg)
```


Filter a single vault
```
v <- triageVault(vaults[[1]], anno)
v <- filterVault(v)
```

Save one filtered vault
```
#one filtered vault
circosPlot(v,savef=TRUE) # saves 1 pdf
```

Save all from the metavault to individual pdfs
```
circosPlot(mv,savef=TRUE) # saves 5 pdfs
```



Save all from the filtered metavault to individual pdfs
```
#filtered metavault
mvf <- triageVault(mv, anno)
mvf <- filterVault(mvf)
circosPlot(mvf,savef=TRUE) # saves 5 pdfs
```

# Cohort Level Circos Plots

The `cohortCircosPlot` function provides the ability to visualize a single cohort on one plot to look for recurrent events. 

```
Description:

     make circos plots of the CNV, SV and Fusion data, summarizing
     recurrent events

Usage:

     cohortCicosPlot(
       mv,
       bin_size = "cytoband",
       file = NULL,
       title = NULL,
       cnv = TRUE,
       sv = TRUE,
       fusion = TRUE,
       gnm = "hg38"
     )

Arguments:

      mv: a TPO dtVault

bin_size: currently only supports 'cytoband'

    file: NULL pdf to save, if NULL will use default graphics device

cnv, sv, fusion: booleans for which data types to display Default: TRUE
```

The function operates on a `dtVault` like `circosPlot`: 

```
dtvt <- readRDS('triaged_dtVault.rds')
cohortCicosPlot(dtvt, title="My Cohort",file='./my_cohort_luad.pdf')
```

The plot shows all SV in black and all fusions in gold, with the thickness of the lines proportional to the recurrence of the event in the cohort. The innermost ring shows genomic density of translocations (purple), the next ring shows deletions (blue), duplications (red) and inversions (green). The next ring shows the average copy of each cytoband in all samples (the y axis is `C`, the x or radial axis is genomic location, the colors are standard CNV colors, LOH: green, Gain: red, Loss: blue, Neutral: black). All cases are plotted with some `alpha`, so darker colors means more cases have that value (i.e. every cytoband has `n` points plotted where `n` is the number of cases in the cohort).     

![image2](cohort_circos_demo.png)