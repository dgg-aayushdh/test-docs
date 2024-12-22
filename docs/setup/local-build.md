## Overview

This section will detail the steps necessary to make TPO operational in the GCP cloud and locally. The purpose of this process is to create two GCP images: one bootable instance image `tpoboot` and one reference image `tporoot`. Together these two GCP images are used to launch GCP instances which will execute a TPO pipeline on the cloud.

Important variables:
- `BOOT_VER` is the version of the TPO docker images and the GCP instance image
- `ROOT_VER` is the version of the GCP image containing the TPO static and dynamic references
- `TPO_ROOT` is the root directory of the TPO source tree
- `config.ini` is a config file based on a copy of a template in `$TPO_ROOT/config` 

## References

Not all files necessary for the execution of a genomic pipeline can be kept in a git repository, such as reference genomes, gene models, large annotation files etc, custom software etc., due to their size. TPO stores such reference files in a GCP bucket. These files are used in two scenarios: when building Docker images, and when building the GCP instance image `tporoot` (see below).

The first step in the building of TPO is to pull the reference files locally. To do this, update the `config.ini` file:

```
[RUNTIME]
WORK = <directory>
REFS = <directory>
```

The `WORK` directory is required. If `WORK` is provided but `REFS` is blank, `REFS` defaults to `$WORK/refs`, otherwise `REFS` should be a directory both the `user` and `root` have read-write access to.

Next, execute the following command to pull references and indexes from GCP bucket to local directory:

```
$TPO_ROOT/tpo.py -project config/my_config.ini refs_pull
```

Depending on the internet connection this may take some time (10+ minutes). A total of 50-100GB will be downloaded.

See [Development](Development#References) for instructions on how to add custom files to the references.

## Docker images

Once the reference files have been pulled it is possible to build the required Docker images.

First, the `config.ini` file needs to be edited, to specify `BOOT_VER` a version number which will be used to tag the docker images e.g. `tpobase:$BOOT_VER`. The same version number will be later used to tag the GCP boot image `tpoboot:$BOOT_VER` (see below).

```
[TPO]
BOOT_VER = <version number>
```
The version number above can be anything e.g. `0.0.1-rel1`. Once it is set we can build the Docker images:

```
$TPO_ROOT/tpo.py -project config/my_config.ini boot_build
```

This command takes two optional parameters 
- `--nocache` to build the images from scratch and drop the docker build cache. Equivalent to the `--no-cache` argument in docker [build](https://docs.docker.com/engine/reference/commandline/build).
- `-image <image_name>` to specify which image to build (among `base`, `cords`, `crisp`, `carat`) default: all

Depending on the local machine performance building the images will take up to 30min. The process will result in the creation of four images versioned by `$BOOT_VER`.

```
$ docker images
REPOSITORY                 TAG         IMAGE ID       CREATED        SIZE
gcr.io/mctp-gce/tpocarat   <$BOOT_VER>   8d135b36df60   0 days ago     12GB
gcr.io/mctp-gce/tpocrisp   <$BOOT_VER>   603db1bf58f6   0 days ago     11.7GB
gcr.io/mctp-gce/tpocords   <$BOOT_VER>   1a0efa23bc63   0 days ago     11.9GB
gcr.io/mctp-gce/tpobase    <$BOOT_VER>   f19a24294dc7   0 days ago     10.7GB
```

Note: the IMAGE IDs and SIZEs will differ based on the time point the images are created.

## Google Cloud Platform (instance) image

When TPO is being run on the GCP, the pipeline code is executed within a docker container which is running one of the docker images created above. Since these images are quite bulky it is not efficient to pull them (i.e. `docker pull <image>`) prior to running each execution. Therefore, we will prepare a custom GCP Instance [image](https://cloud.google.com/compute/docs/images#custom_images) with all of the images pre-loaded.

The whole process is comprised of three steps:

1. Push local docker images to the Google Container Registry (GCR)
2. Pull docker images from the GCR to a GCP instance
3. Save the updated GCP instance as a GCP image

The above steps translate into two fully-automated commands:

```
$TPO_ROOT/tpo.py -project config/my_config.ini boot_push
```

This command will push the local images that were just built. You can go to https://console.cloud.google.com/gcr/images, then select your project from the drop-down list in the upper left corner, and see if your local docker images were successfully uploaded to GCR.

```
$TPO_ROOT/tpo.py -project config/my_config.ini gcp_boot_build
```

The second command will automatically perform a series of steps: start a GCP instance, pull docker images, create GCP image from instance. By default, the `gcp_boot_build` command will wait (issuing periodic updates) until all of those steps have been completed. This command should not be interrupted, as it will leave the build-process in a broken state. The process should not take more than 15 minutes.

We can check for success by searching for the instance image on GCP: 

```
$ gcloud compute images list | grep tpoboot
tpoboot-<$BOOT_VER>                          mctp-gce                                                           READY
```

There are three uses for this GCP instance image:
- building TPO references (covered next)
- launching TPO pipelines on GCP (see: [Usage](Usage))
- starting a TPO instance for debugging and development (see: [Development](Development#gcp-instance))

## Building TPO work disc

Running instances require working disk space to store input, temporary, and output files. These disks are created from the `work` image, which is a formatted (ext4) GCP image that is automatically resized when an instance is launched. Having a pre-formatted empty image saves time when launching instances, as initializing and formatting large images takes a lot of time.

```
$TPO_ROOT/tpo.py -project config/my_config.ini gcp_work_build
```

## Building TPO references

Running TPO pipelines both locally and on the GCP requires access to a large number of reference files (e.g. reference gnomes, annotations), but even more importantly large indexes for read alignment and quantification. These indexes take many hours to build and require significant compute resources. Typically it is not possible to build those references on a standard desktop, therefore we need to build the references on the GCP. Hence unlike docker images which we build locally and push to the cloud, we build the references locally and optionally pull them locally.

Building TPO references is carried out by one command `gcp_refs_build`, which automates the following steps:

1. Launch a GCP instance based on the `tpoboot-$BOOT_VER` image (analogous to `gcp_instance`)
2. Download the static reference files (analogous to `refs_pull`)
3. Build (indexing, compilation) dynamic reference files
4. Save the updated references as a GCP image

```
$TPO_ROOT/tpo.py -project config/my_config.ini gcp_refs_build
```

The above command will build a GCP instance image versioned by `$ROOT_VER` i.e. `tporoot-$ROOT_VER`. The process takes 4h or more hours to complete.

We can check for success by searching for the instance image on GCP: 

```
$ gcloud compute images list | grep tporoot
tporoot-<$ROOT_VER>                         mctp-gce                                                           READY
```

TPO is fully operational on the GCP once `tporoot-$ROOT_VER` (and previously `tpoboot-$BOOT_VER`) has been successfully created. To operate TPO locally the contents of `tporoot-$ROOT_VER` have to be downloaded locally. The process involves:

1. creating a tar-archive of the contents of the `tporoot-$ROOT_VER` image
2. pushing the tar-archive from a GCP instance to a GCP bucket
3. pulling the tar-archive from a GCP bucket to a local instance
4. unpacking the contents of the tar-archive

The first two steps happen on the GCP cloud i.e. an instance is launched which mounts `tporoot-$ROOT_VER` and creates a tar-archive of its contents, this happens as part of `gcp_root_push`, the next two steps happen on the local instance as `gcp_root_pull`.

```
$TPO_ROOT/tpo.py -project config/my_config.ini gcp_root_push
$TPO_ROOT/tpo.py -project config/my_config.ini gcp_root_pull
```

Warning: because the static and dynamic references stored in `tporoot-$ROOT_VER` are large, a significant amount of storage (250Gb or more) is required locally for `gcp_root_pull` to complete. Locally, the contents of the `tporoot-$ROOT_VER` image will be stored in a directory as specified by the configuration file:

```
[RUNTIME]
ROOT = <path>
```

Here `ROOT` should be a directory where the user (and the UNIX `root`account) have both read-write access to. The whole process of pushing and pulling `tporoot-$ROOT_VER` locally can take several hours and will largely depend on the local network bandwidth and disk storage performance.