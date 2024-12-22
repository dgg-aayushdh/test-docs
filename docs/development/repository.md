# Repository 


# Top-Level Directories
```
$TPO_ROOT
├── base
├── config
├── examples
├── cli
├── pipe
├── rlibs
├── scripts
├── task
├── test
```

## `cli`
Implementation (python modules) of the main TPO CLI interface for all TPO tasks. Each TPO task is implemented as a python function for which a commandline interface is created using the `moke` python package. Tasks are grouped by type of analysis (e.g. DNA analyses are in `cli/cords.py`) and housekeeping / development tasks `cli/devel.py`. Internal functions setting up the CLI (e.g. defaults) are in `cli/__init__.py`.
## `task`
Implementation (Shell scripts) of the actual TPO tasks. Each directory in `task` implements a single analysis task. For example, `task/cords-align` implements the the DNA alignment pipeline. Every task is comprised of at least three shell scripts. The actual analysis code `*_docker.sh` the local launcher script `*_local.sh` and the remote (GCP) launcher `*_gcp.sh`. 
## `pipe`
Implementation (Shell scripts) of TPO pipelines (wrapper of TPO tasks).
## `rlibs`
R-libraries implementing fundamental TPO functionality, used by TPO tasks.
## `base`
Contains shell scripts and Dockerfiles required to bootstrap the TPO infrastructure of docker and GCP images.
## `config`
Configuration (ini) file templates and documentation.
## `examples`
Short scripts files covering TPO use-cases.
## `scripts`
Misc. scripts.
## `test`
TPO test scripts.
