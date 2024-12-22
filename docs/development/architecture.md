# Architecture

TPO follows as much as possible 

Design principles:

- infrastructure is designed and optimized to run on the cloud
- easy to update and modify
- take advantage of GCP managed services (block storage, object storage, instances, images, container registries)
- pipelines should run locally for development and debugging purposes
- pipelines are implemented as simple shell scripts

## R packages

When adding new dependencies for packages in `rlibs`. The `tpobase` image. 

## References

**Warning**: Always do a `refs_pull` prior to doing a `refs_push` as files absent locally will be deleted remotely in a `refs_push`.

## Images

TPO uses two types of images, Docker images and GCP images. The main use for Docker images is to provide a standarized environment for the execution of pipelines, both locally and remotely on GCP. A secondary use case is to provide the TPO code to the using [volumes](https://docs.docker.com/storage/volumes). The docker images are built locally, and distributed using the Google Container Registry (GCR) to the GCP instances. The GCP images are used to launch standardized GCP instances and to provide those instances with large files which cannot be kept in a git repository, such as genomic references.

Follwing the execution of a TPO command, the following steps happen:

1. A GCP instance is started using the `tpoboot` GCP image. This image has all required TPO Docker images pre-loaded.
2. The GCP instance mounts a read-only disk from the `tporoot` GCP image. This disk provides all required reference files under `/tpo`.
3. The GCP instance downloads a `tpocode` Docker image which provides a version of the TPO codebase as a volume.
4. On the GCP instance a Docker container is started from one of the Docker images (e.g. `tpocords`), with the `tpocode` Docker image mounted as a volume under `/code`.
5. The running Docker container (provided with the volumes and input data) executes the pipeline.

## Code volume Docker image

TPO uses a Docker volume to create snapshots of the TPO codebase (i.e. everything under `$TPO_ROOT`) to deploy on the cloud. The images are versioned using the `CODE_VER` variable in the `[TPO]` section of the config file.

| Command    | Purpose |
|------------|---------|
| code_build | Build the TPO code image locally |
| code_push  | Push the image to the GCR registry |
| code_pull  | Pull the image from the GCR registry |

The resulting image `tpocode` provides a `/code` volume which is mounted on the executable Docker images.

## Executable Docker images

TPO uses 4 different Docker images. A base image, called `tpobase` and 3 derivative images.

| Image    | Description |
| -------- | ----------- |
| tpobase  | An Ubuntu 18.04 image configured to support the compilation and running of all TPO software |
| tpocords | tpobase + custom tools required for the `cords` pipelines |
| tpocrisp | tpobase + custom tools required for the `crisp` pipelines |
| tpocarat | tpobase + custom tools required for the `carat` pipelines |

The `tpobase` image is a complete Ubuntu-based system with commonly required dependencies installed. The derivative images install software specific to certain types of analyses on top of the `tpobase` image.

The following `tpo.py` commands control the building and push/pull of the Docker images. The images are versioned using the `BOOT_VER` variable in the `[TPO]` section of the config file.

| Command    | Purpose |
|------------|---------|
| boot_build | Build the TPO docker images locally |
| boot_push  | Push the images to the GCR registry |
| boot_pull  | Pull the images from the GCR registry |

In general, there is typically no need to modify the images outside of TPO development. So the images are built and pushed once using `boot_build` and `boot_push`, respectively.

## GCP images

Two GCP images are built, one `tpoboot` providing a standardized instance for all TPO cloud runs, and another `tporoot` containing all the files that are necessary for the TPO pipelines to run. The `tpoboot` and `tporoot` images are versioned using the `BOOT_VER` and `ROOT_VER` variable in the `[TPO]` section of the config file, respectively.

| Image   | Version  | Purpose |
|---------|----------|---------|
| tpoboot | BOOT_VER | Minimal Ubuntu-based image to launch GCP instances |
| tporoot | ROOT_VER | Large image containing reference and index files |

The `tpoboot` image is a minimal Ubuntu-based image with the TPO Docker images (`tpocords`, `tpocrisp`, `tpocarat`) pre-loaded. It only needs to be rebuilt if any of the Docker images gets updated, or there is a need to update the Ubuntu OS (e.g. security reasons).

The `tporoot` image is a data volume and cannot be booted. It contains the complete set of assets required to run TPO. This includes genomic reference files, built indexes for various alignment tools, compiled R libraries etc. The contents of this image depend on some settings in the TPO configuration file. For example it contains built alignment indexes only for the genomic references indicated in `ALIGN_FASTA`. Therefore, the `tporoot` image has to be in general changed whenever any of the following variables is modified. 

- `ALIGN_FASTA`
- `ALIGN_NAME`
- `ALIGN_MASK`
- `QUANT_GTF`
- `QUANT_FASTA`
- `QUANT_NAME`
- `ANNO_VEPCACHE`

In practice, `tporoot` is built only once for a given reference genome configuration. TPO provides config template files for GRCh38 following community recommentations in `$TPO_ROOT/config`.

Building the `tporoot` image requires considerable computational resources and cannot be done on a laptop or desktop computer. Currently the only way to create a new image is on the GCP. This process takes approximately 4 hours.

| Command        | Purpose |
|----------------|---------|
| gcp_boot_build | Build the `tpoboot` GCP image |
| gcp_root_build | Build the `tporoot` GCP image |
| gcp_root_push  | Create a tar archive of the `tporoot` image and save it in a GCP bucket |
| gcp_root_pull  | Download and extract the tar archive locally |

If intending to run TPO locally, following `gcp_root_build`, the user needs to execute `gcp_root_push` and `gcp_root_pull` to transfer the contents of `tporoot` into a local directory, as specified by the `ROOT` variable in the `[RUNTIME]` section of the config file.

## GCP instance

A GCP instance based on the `tpoboot-$BOOT_VER` image can be created using the following command:

```
$TPO_ROOT/tpo.py -config config/my_config.ini gcp_instance
```

Warning! Please not this command will start an instance which will be running indefinitely, possibly incurring high costs. The instance should be suspended, stopped, or terminated when not in use. Please see GCP documentation on how to accomplish this from either the CLI or cloud console. 

Add user to docker group in order to use docker:

`sudo usermod -aG docker <userID>`

## Test cases
```
./tpo.py -config mokefile_grch38-tpo-latest-v2-e106.ini cords_align HLGWCDRXX.1 SI_30806 \
gs://tpo-refs/v2/test/fastq/tiny_SI_30806_HLGWCDRXX_0_1.fq.gz \
gs://tpo-refs/v2/test/fastq/tiny_SI_30806_HLGWCDRXX_0_2.fq.gz
```

## Developer Notes

- after Java update, check if bbmap JNI still compiles (i.e. if JAVA_HOME is valid)
- lots of error messages can be lost because we are running docker in -d, disable if debugging
- All hard-coded variables in build_refs need to go in order to support alternate species
- updating Ensembl: downlad fasta, bgzip, faidx, cdna for kallisto, gtf fix tags