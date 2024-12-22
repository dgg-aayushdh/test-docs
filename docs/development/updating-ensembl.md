# Updating TPO to a newer Ensembl version

Updating TPO to a newer Ensembl version is relatively straightforward, but requires multiple steps as typically both software (i.e. `VEP`) and reference files need to be updated in lock-step. Both `Docker` and `GCP` images will need to be rebuild to incorporate the changes.

The process consists of the following steps which will result in an update of `VEP`, the `VEP` annotation sources which include Ensembl i.e. [vep_cache](https://useast.ensembl.org/info/docs/tools/vep/script/vep_cache.html), and Ensembl `GTF` reference [files](https://useast.ensembl.org/info/data/ftp/index.html).

1. Updating `VEP` to support the new annotation sources and rebuilding (at least) the `tpocarat` Docker image.
1. Building a new GCP instance image (i.e. `tpoboot`) containing the updated Docker images.
1. Downloading and modifying GTF and FASTA files from Ensembl, downloading VEP cache files.
1. Creating a new `tporefs` reference image containing the updated `GTF`, `FASTA`, and `VEP` cache files.

## 1. Update VEP and build new Docker images

We need to update `VEP` to a version which will support the latest Ensembl `VEP` annotation sources (i.e. `vep_cache`).

1. Identify the required VEP version.
   - Find latest VEP on [GitHub](https://github.com/Ensembl/ensembl-vep), it is good to wait for a first bug-fix release e.g. `106.1`.
   - Check if latest VEP version is also available on [Bioconda](https://anaconda.org/bioconda/ensembl-vep), if not it is better to wait.
   - In TPO code edit `base/images/tpocarat/Dockerfile` to reflect the desired VEP version from Bioconda.
2. Create updated Docker images with new VEP. For consistency it is reasonable to update all images not only `tpocarat`.
   - In the TPO config file bump `BOOT_VER` to not override the old images.
   - If necessary do a `refs_pull` on the instance, as some files (e.g. installers) are required to build Docker images.
   - Do a full `boot_build`, ideally without reusing the Docker cache.

We now have a set of new Docker images locally.

## 2. Build new GCP instance image

1. Push updated images to the GCR using `boot_push`
2. Create updated GCP instance image using `gcp_boot_build`

We now have a set GCP instance image with the updated Docker images.

## 3. Create Updated reference files

1. Download and Ungzip the following files, replacing 106 with the desired version
 - http://ftp.ensembl.org/pub/release-106/variation/vep/homo_sapiens_merged_vep_106_GRCh38.tar.gz
 - http://ftp.ensembl.org/pub/release-106/fasta/homo_sapiens/cdna/Homo_sapiens.GRCh38.cdna.all.fa.gz
 - http://ftp.ensembl.org/pub/release-106/gtf/homo_sapiens/Homo_sapiens.GRCh38.106.gtf.gz
2. Fix GTF file by combining multiple `tag` features into one `tags` feature
```
cat Homo_sapiens.GRCh38.108.gtf | scripts/make_gtf_tags.py fix_tags > Homo_sapiens.GRCh38.108-tags.gtf
```
3. Create `clean` GTF and FASTA file,
In an interactive R-session, walk through the steps in `scripts/make_ensembl_rna_gtf.R`. This will create a cleaned up GTF file, and a corresponding FASTA file.
4. Test if a new CODAC configuration file can be created from the new GTF file. 
```
makeAnnotations("Homo_sapiens.GRCh38.108.clean.gtf", "hg38")
```
If it fails or gives warnings it means the structure of the GTF file changed, do not ignore investigate.

## 4. Create a new `tporoot`

We will use the new `tpoboot` instance image to build a new `tporoot` with the updated reference files

1. Copy all new and updated files in the right places.
```
# original GTF file
cp Homo_sapiens.GRCh38.108.gtf $XXX/v2/refs/refs/grch38/ensembl/grch38.108.all.gtf
cp Homo_sapiens.GRCh38.cdna.all.fa $XXX/v2/refs/refs/grch38/ensembl/grch38.108.all.cdna.fa
# clean GTF and FASTA file
cp Homo_sapiens.GRCh38.108.clean.gtf $XXX/v2/refs/refs/grch38/ensembl/grch38.108.clean.gtf
cp Homo_sapiens.GRCh38.108.cdna.clean.fa $XXX/v2/refs/refs/grch38/ensembl/grch38.108.all.cdna.fa
## VEP file
cp homo_sapiens_merged_vep_108_GRCh38.tar.gz $XXX/v2/refs/refs/grch38/ensembl/grch38.108.merged.vep.tar
``
2. Push updated reference files to the cloud using `refs_push`
3. Build updated `tporoot` image.
    - Update `ROOT_VER` to newer version reflecting update of both `BOOT_VER` and bump in Ensemble version, e.g. 
      ```
      ROOT_VER = 2.5-0-grch38-tpo-108
      ```
    - Trigger build of new `tporoot` GCP image using `gcp_root_build`

