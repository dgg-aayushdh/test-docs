### Introduction

TPO Silos and Vaults solve two orthogonal problems in the storage, integration, and analysis of genomic data. By necessity genomic pipeline results are produced by a variety of tasks each saving results in a distinct file format. The `CARAT` packages (located in `rlibs/carat`), can be thought of as TPOs ETL (Extract Transform Load) solution. It first extracts and collects all the relevant data, this data is stored in a binary `Silo` object. The following step creates a `Vault` object represents the transformed data which can be then loaded into a columnar or SQL database.

### Motivation

The idea for `dxVaults` is two-fold. 

First, the outputs of `carat-vault` tend to evolve as new functionality is added and bugs being fixed. For example tables are updated with new columns (e.g. the `pid`, `ccf`, or support for normal quasr output). Vaults created at different times are not compatible for `metaVault`. The `dtVaultUpdate` function updates tables in old formats to the newest format.

Second, the `dtVault` fixes some design decisions around legacty `vaults` / `metaVaults` allowing us to store all the data - lossless - in an SQL database, such as DuckDB, PostgreSQL and/or SQLite. This enables the use datasets that would not fit otherwise into memory, and the development of web-applications.

Depending on how the data is accessed and stored (but not what data is stored) there are three types of vaults:

1. `dtVault` stored using `data.table` and accessed either using fast `data.table` or slow `dplyr` APIs.
2. `dpVault` stored using `data.table` and accessed using fast `dtplyr` APIs (i.e. `dplyr` for data.tables)
3. `dbVault` stored using `SQL` and access using `dbplyr` APIs (i.e. `dplyr` for data bases)

The three interfaces `dplyr`, `dtplyr` and `dbplyr` are very similar. In the current code base we use a mix of `data.table` and `dplyr` APIs when using legacy `vaults` and `metaVaults`. Universal functions that can work with all vaults have `dx` prefix.

### Quick Start

**Where is the code**

```
# dxVault* functions (these functions work on all dt/dp/db vaults)
<TPO_GIT_ROOT>/rlibs/carat/R/dtVault.R
# dpVault* functions
<TPO_GIT_ROOT>/rlibs/carat/R/dpVault.R
# dtVault* function
<TPO_GIT_ROOT>/rlibs/carat/R/dtVault.R
# dbVault* functions
<TPO_GIT_ROOT>/rlibs/carat/R/dbVault.R
# DuckDB backend
<TPO_GIT_ROOT>/rlibs/carat/R/duckdb.R
```

**Create `dtVaultAnno` object**

A `dtVault` object is stored in a `dtVault` as `$anno` it is used in triaging.

```
dtanno <- dtVaultAnno("/tpo/refs/grch38/ensembl/grch38.108.clean.gtf")
```

This will create a table with gene-level annotations. This is `data.table` which links gene_ids, gene_names, chromosomal positional information and columns indicating whether a gene is of interest or an artifact. This function allows providing optional germline and somatic genes-of-interest (GOI), and artifacts: `somatic_goi`, `germline_goi`, `somatic_art`. By default those lists are from corresponding files in the `carat/inst/goi` directory.

**Create a `dtVault` from a legacy vault**

```
vault.fn <- ".../xxx/xxx-vault.rds"
v <- readRDS(vault.fn)
dtv <- dtVaultUpdate(v, dtanno)
```

**Create a multi-sample `dtVault` from a list of legacy vaults**

```
vault.fns <- ...
vs <- lapply(vault.fns, readRDS)
dtvs <- lapply(vs, dtVaultUpdate, dtanno)
dtv <- dtVaultStack(dtvs)
```

**Create a `dbVault` from a multi-sample `dtVault`**

```
createDuckDBFromVault("test-vault-db.duckdb", dtv)
con <- dbConnect(duckdb::duckdb(), dbdir = "test-vault-db.duckdb", read_only = FALSE)
dbv <- dbVault(con)

```

**Create a `dbVault` directly from multiple legacy vault files**

```
vault.fns <- ...
createDuckDBFromVaultFiles("test-vault-db2.duckdb", vault.fns, dtanno)
```

**Convert a `dbVault` into a `dtVault`**

```
dtv <- dtVaultCollect(dbv)
```

### Variant Triaging and Filtering

Triaging and filtering of `dxVaults` is a two step process, giving the user the flexibility to define which tables to triage, and which how to filter variants based on the triaging result. 


1. `vaultTriage` applies triaging functions to the provided `dxVault`. This updates (optionally in-place) the `triage` column with a string representing the triage result i.e. `PASS` or as a semi-colon delimited string `problemX;problemY`.
2. `vaultFilter` select variants that do not match a set of filtering rules. This returns a new `dxVault`

default rules:
```
#'   list(
#'     somatic="annotation|consequence|evidence|artifact",
#'     germline="annotation|consequence|evidence|goi",
#'     structural="evidence",
#'     fusion="evidence"
#'   )
```

If the above rule-set is provided, `vaultFilter` will remove `germline` variants that with the `annotation`, `consequence`, `evidence` and `goi` problems, but `structural` variants will be only filtered based on `evidence`.

```
## subset dbv to only have variants related to RBM10 and convert into a dtVault
dtv.rbm10 <- dbv |> dxVaultSubsetByGene("RBM10") |> dbVaultCollect()
## triage the subsetted dtVault (but triaging can also be done on the complete dbVault)
## by default all tables are filtered and the dbVault or dtVault is modified in-place
dtv.rbm10.t <- vaultTriage(dtv.rbm10)
## filter variants based on the default rules
dtv.rbm10.f <- vaultFilter(dtv.rbm10.t)
```

### Versioning

If it is helpful to keep track of changes to the structure of the dtVault object by versioning it. Each vault has a `$version` and only vaults with compatible versions can be stacked. The structure of a vault with a given version is stored in `inst/extdata/dtvault-{ver}.schema`.

A new schema can be created with the following commands. the resulting schema file will be used to ensure the consistency of the object by some functions operating on `dtVault` objects. Here `dbv` is any vault.

```
schema <- .dtVaultSchema(dtv)
writeLines("<TPO_DIR>/rlibs/carat/inst/extdata/schema-{ver}.schema")
```

A schema is essentially a vault with no rows. It can be loaded as follows

```
dtv0 <- eval(parse("<TPO_DIR>/rlibs/carat/inst/extdata/schema/vault-v2.schema"))
```




