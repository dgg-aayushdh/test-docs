# CURVE Portal

The `curve` R library included in TPO contains the code needed to create a case/patient-centered R Shiny portal for viewing TPO pipeline results. It enables users to deploy a Google Cloud instance with PostgeSQL and the appropriate schema, as well as the R Shiny application front end.  

## Initialization
To launch a Google Cloud instance with the appropriate schema, first ensure that `CRISP_QUANT_GTF` is set to the correct gene model used during the analysis (regardless of whether `crisp-quant` or `crisp-quasr` was used for the RNA analysis). This GTF will be used to create the schema for the database to store gene expression data. Additionally, set `SECRETS_CURVE_USER` and `SECRETS_CURVE_PW` to a username and password of your choice. Next, run 

```
tpo.py -config <my-config.ini> curve_db_init <name>

```

When the task completes, a GCP instance with the `<name>` chosen above will be visible with 

```
gcloud compute instances list
```

Next, copy the internal IP of the new instance and use it to create the `CURVE_DB` environment variable like 

```
export CURVE_DB=<ip address>:5432:curvedb:<SECRETS_CURVE_USER>:<SECRETS_CURVE_PW>
```

Import a vault (the output of `carat_vault`) into the newly created database with 

```
Rscript <TPO_ROOT>/rlibs/curve/scripts/import_vault.R -v <path to vault.rds> -g <path to gtf>
```

After the desired vaults are imported, you must refresh some of the database views (this will happen automatically via `cron` `@daily`, but to proceed they must be updated once). 

```
<TPO_ROOT>/rlibs/curve/scripts/connect.sh
# run the following SQL commands
REFRESH MATERIALIZED VIEW somatic_recur;
REFRESH MATERIALIZED VIEW germline_recur;
REFRESH MATERIALIZED VIEW fusion_recur;
```

Finally, to run the application in R, first set the env variable (if it is not already set)
```
devtools::load_all('/mnt/share/code/tpo/rlibs/curve')
Sys.setenv('CURVE_DB'='<ip address>:5432:curvedb:<SECRETS_CURVE_USER>:<SECRETS_CURVE_PW>')
shiny::runApp('./rlibs/curve/shiny')
```

To add additional databases to the existing GCP instance (e.g. for other projects), first set the `CURVE_DB` environment variable to the existing database, then modify the `mokefile.ini` with a new username and password (if desired), then run 

```
tpo.py -config <my-config.ini> curve_db_add <new name>

```

Then update your `CURVE_DB` environment variable to the new database credentials. 

## Adding metadata

Metadata from a csv file can be added to the database and displayed in the sidebar. This metadata can either be per case (it will be applied to all analyses in the DB which match that case) or per tumor sample. The csv must have a column called `case` and optionally can have a column called `alignid_t`. If your `CURVE_DB` variable is set, run 

```
Rscript ./rlibs/curve/scripts/sideload_metadata.R -m ./curve_ready_metadata.csv
```

or you can provide a database if your choosing with 

```
Rscript ./rlibs/curve/scripts/sideload_metadata.R -m ./curve_ready_metadata.csv -d "<ip address>:5432:curvedb:<SECRETS_CURVE_USER>:<SECRETS_CURVE_PW>"
```

## Adding feature data

Feature-level data (i.e. proteomics) can be added by running 
```
Rscript  ./rlibs/curve/scripts/sideload_featuredata.R -f tumor_proteomics_data.csv -g grch38.108.clean.gtf -n 'Tumor Proteomics'
```
The `-n|--name` argument specifies how it will be displayed in the database, this can have an unlimited number of values (i.e. "Tumor Proteomics", "Normal Proteomics" etc.)

The feature data should have a column called `gene_id` which maps to the gtf, and additional column names should be the tumor libraries in the `groupings.alignid_t` column in the database: 

```
gene_id,SI_31726,SI_31727,SI_31728,SI_31729,SI_31730,SI_31731
ENSG00000000003,25.658270426101506,25.177941588030407,26.259224491207846,25.08829842251876,25.35722453735322,24.19243418767442
ENSG00000000419,27.157933578782927,27.02590571202042,27.28431746818307,26.382172053762027,26.563799134992284,26.24792074908906
ENSG00000000457,22.107181257530307,22.377517486832268,22.786689254413982,22.247178863137258,22.519943787643896,22.711983638005982
ENSG00000000938,20.987427212477797,21.571249076966673,21.52211488625682,21.05133550391971,21.13084974148148,21.514292520849526
ENSG00000000971,29.78160643892207,29.22701562690786,29.530658892692795,29.835090234851148,30.87703665066274,30.37104840417782
ENSG00000001036,27.589627669217133,27.48563431228603,27.20853714345757,27.165062596101397,26.891624930214046,26.93188615596623
ENSG00000001084,27.564997110778258,27.621554193888702,27.385591592442797,27.116940514726977,27.752192110344744,27.97982524429814
ENSG00000001167,20.23906629527079,20.637911880139654,21.439503936347002,20.984022166587522,22.313036507648828,22.54092939525347
ENSG00000001461,NA,NA,NA,NA,NA,NA
```