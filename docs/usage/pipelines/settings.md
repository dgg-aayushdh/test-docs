# Pipeline sections



```
[BCL]
# BCL settings and parameters 

[CORDS]
# Settings related to DNA analyses including
# - Alignment
# - Somatic variant calling
# - Germline variant calling
# - Structural variant calling
# - Copy-number variant calling
# - Miscellaneous analyses

[CRISP]
# Settings related to RNA analyses
# - Alignment
# - Germline variant calling
# - Alignment-free quantification
# - Alignment-based quantification
# - Fusion detection

[CARAT]
# Settings related to variant annotation

[TEST]
# Settings useful during debugging and testing
```
Most of the settings are required, there fore a user should generally start with a template and modify. Templates are provided in the [config](../tree/master/config) directory in the main repository.
Configuration files are neither forward nor backward-compatible i.e. an upgrade of TPO may require and update to the configuration file, but those changes are typically very minor.

# Settings descriptions

