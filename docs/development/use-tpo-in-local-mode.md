# Run Cords analysis locally
## step1. Configurate Runtime
Please follow the instructions on page [Installation](Installation).
## step2. Modify runtime setting in config file
Please follow the instructions on page [runtime-local-mode](Configuration#runtime-local-mode).
## Step3. Add `--local` in running command. Examples:
### align
```
./tpo.py -config mokefile_grch38-tpo-latest-v2-e106.ini cords_align
    SE_tumor SI_31729
    gs://mctp-tpo/jhgong-tpo-tiny-true/tiny_mctp_SI_31729_H7MFFDSXY_2_1.fq.gz
    gs://mctp-tpo/jhgong-tpo-tiny-true/tiny_mctp_SI_31729_H7MFFDSXY_2_2.fq.gz
    --local
```
### post
```
./tpo.py -config mokefile_grch38-tpo-latest-v2-e106.ini cords_postalign
    SE_tumor
    gs://mctp-tpo/jhgong-tpo-tiny-true/repo/cords-align/SE_tumor
    --local
```
### somatic
```
./tpo.py -config mokefile_grch38-tpo-latest-v2-e106.ini cords_somatic SE_to_tumor
    -alnt gs://mctp-tpo/jhgong-tpo-tiny-true/repo/cords-postalign/SE_tumor
    --local
```