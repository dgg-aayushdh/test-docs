# Prerequisites and dependencies for installation

TPO is designed ground-up to work on the Google Cloud and uses [Sentieon](https://www.sentieon.com/products/) tools to accelerate and improve some analyses. Optionally, TPO can be run locally, which is recommended for development and debugging purposes, and utilize only open-source tools.

This section describes the process of instaling the TPO CLI on a local machine. Additional steps are required to enable the local execution
of TPO pipelines either remotely (default) or [Cloud Deployment](Cloud-Deployment), or locally [Local Deployment](Local-Deployment). 

## Required software

The following software need to be installed by the user or systems administrator:

1. Linux (TPO is tested on Ubuntu 22.04 and RHEL8), but any modern Linux distribution should work.
2. Python 3.6+
3. [Moke](https://pypi.org/project/moke) 1.2.5+
4. Docker (recommended) or Podman (less well tested)
5. [Google Cloud SDK](https://cloud.google.com/sdk/docs/install)
6. rsync, pigz

All of the tools above are required for both local and remote execution of TPO. The documentation only covers installation of Moke, and basic test to veryify that Docker and the cloud SDK are correctly configured.

### Moke

Moke provides a Python library to facilitate the writing of command-line tools, it transforms a Python module into a command-line script. Every function in the module can become a sub-command, with options inferred from the argument list and the optional doc string.

The TPO CLI (Command Line Interface) is implemented as Python modules transformed at runtime by moke into a CLI. Moke can be installed from PIP.

```
$ pip3 install --user moke
```

If `moke` was installed previously it is necessary to force an upgrade.

```
$ pip3 install --user moke -U
```

To test the installation, try importing moke in the default Python interpreter.
```
$ python3
>>> import moke
```

If this succeeded please proceed to the next step. If not the following test may help identifying the problem. Moke will bind to the Python executable used during its installation. However it is helpful to check whether the `site-packages` directory of the default `python3`, as found using the following command, matches the one for `pip3`.

```
# site-packages directory of python3
$ /usr/bin/env python3
>>> import site; site.getsitepackages()
```

```
# site-packages directory of pip3
$ pip3 --version
```

### Testing the installation

Docker installation and permissions can be typically verified using the following command:

```
$ docker run hello-world
| Unable to find image 'hello-world:latest' locally
| latest: Pulling from library/hello-world
| 2db29710123e: Pull complete 
| Digest: sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
| Status: Downloaded newer image for hello-world:latest
| 
| Hello from Docker!
| This message shows that your installation appears to be working correctly.
```

Python installation can be tested as follows:

```
$ /usr/bin/env python
| Python 3.6.9 (default, Jan 26 2021, 15:33:00) 
| [GCC 8.4.0] on linux
| Type "help", "copyright", "credits" or "license" for more information.
```

The `moke` command run in any directory (other than the TPO codebase), should produce the following output:

```
$ moke
| moke: *** No mokefile.py found. Try 'moke new'. Stop.
```


### Google Cloud SDK

Please refer to the [official document](https://cloud.google.com/sdk/docs/install) of Google Cloud SDk for its installation

After installation, Google Cloud SDK needs initiation with GCP account which is discussed in the next GCP account part.

```
gcloud init
```

If executed locally, the `gcloud` tool provided by the Google Cloud SDK will not be used, and `gsutil` will only be used to download TPO reference files and input sequencing data.

## Required accounts and licenses

GCP account is required for TPO to run in the cloud, and Sentieon license is essential if TPO is expected to run in Sentieon mode.

### GCP account

* If intending to use TPO on the Google Cloud Platform (GCP), the user will need a GCP account with standard permissions:
    - creating instances, images, and disks using the [gcloud](https://cloud.google.com/sdk/gcloud) tool.
    - reading from and writing to GCP buckets using the [gsutil](https://cloud.google.com/storage/docs/gsutil) tool.
    - (optionally) creating an GCP [File Store](https://cloud.google.com/filestore) NFS share.

The custom GCP information and settings should be provided to TPO via the configuration file `<config.ini>`. Please refer to [Configuration](Configuration#GCP) for details on how to configure TPO and a GCP project.

### Sention license

Secure a license file from [Sentieon](https://www.sentieon.com/). A [free-trial](https://www.sentieon.com/home/free-trial/) is available.

The Sentieon software license file (or license server) should be provided to TPO via the configuration file `<config.ini>`. Please refer [here](QuickStart#Sentieon) for details on how to configure TPO to use Sentieon license server or license file.

Downloading the Sentieon tools is not required as these are bundled within TPO, but Sentieon is not functional without a license. 

## Installing TPO

TPO is ready directly after git clone:

```
$ git clone https://github.com/mctp/tpo
```

This should create a directory `tpo`, referred to as `$TPO_ROOT` throughout the documentation with the following contents

```
$ tree --dirsfirst -L 1 tpo
tpo
| ├── base
| ├── config
| ├── mods
| ├── pipe
| ├── rlibs
| ├── scripts
| ├── mokefile.py
| ├── README.md
| └── tpo.py -> mokefile.py
```

To check if the installation is functional (mostly if the default python version is the same as the one used by `pip3`):

```
$ tpo/tpo.py
| usage: tpo.py [-h] [-config CONFIG] [-cargs CARGS] [-ls LS]
|               [-ll {debug,info,subdefault,default,warn,error}] [-lf {nltm}]
| {gcp_root_build,cords_align,refs_pull,code_pull,gcp_root_push,cords_structural,gcp_root_pull,crisp_quasr,crisp_codac,crisp_germline,crisp_quant,crisp_align,template,code_push,code_build,gcp_configure,bcl,boot_pull,gcp_instance,gcp_boot_build,cords_misc,cords_germline,carat_report,cords_somatic,refs_push,boot_push,cords_unalign,cords_postalign,carat_anno,boot_build,cords_cnvex}
|              ...
| tpo.py: error: too few arguments
```



