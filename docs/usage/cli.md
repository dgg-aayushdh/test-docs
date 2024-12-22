# TPO Command Line Interface (CLI)

TPO is controlled through a single executable shell script `$TPO_ROOT/tpo.sh`. This executable provide a large number of sub-commands for all major tasks of using TPO. Everything that can be done in TPO is done through this interface, including:

1. Running all analysis tasks and pipelines.
2. Developing, debugging, and updating TPO resources and references.
3. Deploying and configuration locally and on the cloud.

## TPO CLI entrypoint

The `$TPO_ROOT/tpo.sh` entrypoint, referred from now on as `tpo.sh` is, is a simple wrapper around a Python CLI `$TPO_ROOT/tpo.py` implemented using `moke`, see: [Local Installation](Local-Installation). The invocation is the same for all commands.

```
$TPO_ROOT/tpo.sh [log_options] [tpo_options] command [command_options] [arguments...]
```

- **[log_options]** setting passed to `moke` controlling logging functions.
- **[tpo_options]** presets and ini files configuring TPO.
- **[command_options]** named parameters controlling a specific command
- **[arguments]** positional arguments to the command

**log_options**
```
    -ls LS                (file_a) [default: <stderr>] logging stream
    -ll {debug,info,subdefault,default,warn,error}
                          (str) [default: default] logging level
    -lf {nltm}            (str) [default: nltm] logging format`
```

**tpo_options**
```
    - kv (``str``) list of key-value pairs `k::v` separated by `;` in single `'` quotes
    - custom (``str``) Custom ini file, overrides all other ini files
    - project (``str``) Project ini file, provides runtime and GCP settings
    - capture (``str``) Pick capture preset, provides capture baits and targets
    - settings (``str``) Pick settings preset, configures task settings
    - genome (``str``) Pick genome preset, configures reference genome
```
