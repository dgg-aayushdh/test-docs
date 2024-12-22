Adding new pipelines to TPO involves two steps: 
* Adding the pipeline scripts
* Adding software (binaries, etc.) needed for the pipelines to run

## Adding pipeline scripts
TPO pipelines consist of (at least) two components:
* A gcp script (`*_gcp.sh`) which launches the Google Cloud instance and handles input and output of files to the cloud buckets
* A docker script (`*_docker.sh`) which is called by a docker container inside of the GCP instance launched above. This script does the "work" of whatever bioinformatics pipeline is in use (i.e. a call to `samtools` will be inside of this script).

Updates to the scripts done on your local machine need to be made available to the gcp instances where they are needed. This is accomplished via the `code` docker image. After changes are made, consider a version bump, then `moke images_build -image "code"` to update the docker locally and `moke images_push` to copy the code image to GCR. By default the `images_build` task will tar up the local `tpo` directory as is, so any changes (even those not staged, committed, etc.) will be made live. 

## Adding new software  
Software can be made available to TPO scripts in several ways. Deciding which to use is somewhat arbitrary, but should depend on whether the software will be used by multiple pipelines, or just one. The options are 
* Add to `root`
  * This will be available to the gcp instance and any docker images which mount `/tpo:/tpo`
* Add to `base` docker image
  * This will be available to `base` and all docker images that inherit from it
* Add to a specific docker image (i.e. `cords`)
  * This will only be available to that image

The procedure for adding new files to `root` involves the following 
* `moke artifacts_pull` to download the artifacts to a local directory (specified in mokefile)
* wait for pull
* copy binary to artifacts/input
* update appropriate build script (i.e. `base/root/build_tools.sh`)
* Add references if needed
* `moke artifacts_push` to rsync changes to the bucket
* Bump version (`ROOT_VER`) (**_check that no image with a conflicting name/version is present on GCP before proceeding_**)
* `moke gcp_root_build` to launch a gcp instance to build the `ROOT` directory with the new version



