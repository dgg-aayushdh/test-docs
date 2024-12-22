# Unit Testing

Testing can be accomplished using the built in `unit_test` task: 

```
$ ./tpo.py unit_test -h
usage: tpo.py unit_test [-h] [-test TEST] [-start_from START_FROM] [--skip_r_tests]
                        [--skip_cleanup] [--dry_run]

options:
  -h, --help            show this help message and exit
  -test TEST            (str) [default: HCC2218] what unit test sample to test
  -start_from START_FROM
                        (str) [default: align] which pipeline to start from
  --skip_r_tests        (``bool``) Skip the R test
  --skip_cleanup        (``bool``) Skip removing alignments
  --dry_run             (``bool``) save rather than run commands
```

## Getting started

First set up the necessary fields in the mokefile `TEST` section to specify where on GCP and where locally the results should end up, for example:

```
[TEST]
UNIT_TEST_BUCKET = mctp-hopkinal/unit_test
UNIT_TEST = /home/hopkinal/tpounit/
```

## Running Automated Tests

The first time you run `unit_test` with a given `UNIT_TEST_BUCKET` it will run the complete test from start to finish (ignoring `-start_from`), saving the results to `<UNIT_TEST_BUCKET>/base`. 

```
./tpo.py -config mokefile.ini unit_test
```

Subsequent tests will generate a unique run ID, and can start anywhere in the process, taking results from `$UNIT_TEST_BUCKET/base/` as their starting point. Run a test from the annotation stage:

```
./tpo.py -config mokefile.ini unit_test -start_from anno
```

### What it does

By default, `unit_test` will 
* Run necessary TPO piplines
* Download the final results (vault output) to the CWD
* Run the specified test in `./rlib/carat/inst/extdata/test/`
* Save output to `<runid>_tpo_unit_test.results`

To run the pipelines (and skip syncing results and running R tests), run with `--skip_r_test`. To generate the testing script but not run anything, use `--dry_run`. 

### Output
A typical output (using `-start_from vault`) looks like 
```
$ ./tpo.py -config mokefile-unit_test.ini unit_test -start_from vault
moke    DEFAULT 2023-05-10 16:44:49,336 task: unit_test
moke    DEFAULT 2023-05-10 16:44:49,336 dry_run: False
moke    DEFAULT 2023-05-10 16:44:49,336 mode: tnr
moke    DEFAULT 2023-05-10 16:44:49,336 skip_r_tests: False
moke    DEFAULT 2023-05-10 16:44:49,336 start_from: vault
moke    DEFAULT 2023-05-10 16:52:09,073 shell[0]: /mctp/users/hopkinal/deployment/bleeding/tpo/pipe/unit-test/unit_test.sh
moke    DEFAULT 2023-05-10 16:52:09,073 stdout: runid: 5ee22c96

$ cat 5ee22c96_tpo_unit_test.results
Wed May 10 16:44:51 EDT 2023 TPO_CODE_VER: 2.6-14-hopkinal
Wed May 10 16:44:51 EDT 2023 TPO_ROOT_VER: 2.6-3-grch38-tpo-108
Wed May 10 16:44:51 EDT 2023 TPO_BOOT_VER: 2.6-11
Wed May 10 16:44:51 EDT 2023 TPO_REFS_VER: tpo-refs/v2
Wed May 10 16:44:51 EDT 2023 Running TPO unit test on Tumor-Normal-RNA data from vault
Wed May 10 16:44:51 EDT 2023 Making Vault
Wed May 10 16:51:45 EDT 2023 Running R Unit Tests

== Testing test-vault.R ========================================================

[ FAIL 0 | WARN 0 | SKIP 0 | PASS 0 ]
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 1 ]
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 2 ]
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 3 ]
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 4 ]
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 5 ]
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 6 ]
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 7 ]
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 8 ]
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 9 ]
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 10 ] Done!
Wed May 10 16:52:09 EDT 2023 Finished
```

## Adding unit tests

The `unit_test` TPO task uses the R `testthat` package. Add new tests to `./rlibs/carat/inst/extdata/test/generic.R`. If you are developing functions that operate on any TPO `*vault` objects, you can add tests to this script and then re-run them (on the vault in `<UNIT_TEST_BUCKET>/base`) by running

```
./tpo.py -config mokefile.ini unit_test -start_from R
```

## Adding unit test samples

If you want to add samples other than HCC2218 to test on, for example `HCC1395`, you will need to do a few things
* First, make a new config file for the test in `rlibs/carat/inst/extdata/test/HCC1395.config` (see HCC2218 example)
* Next, place the fastqs into `gs://tpo-refs/v2/test/fastq/`, with file names like `<UNIT_TEST_LIB_T>_unittest_1.fq.gz`
* Run the new test with 

```
./tpo.py -config mokefile.ini unit_test -start_from align -test HCC1395
``` 

## My Test Worked! What does that mean?

If `unit_test` ran with no errors and all R test passed, this means:
* Your mokefile specifies a working combination of `CODE`, `ROOT`, `BOOT`, and `REFS`
* Any changes you made to `CODE` did not seriously break the pipelines
* The version of `carat` installed locally is compatible with the vaults made by the tested version of the pipeline

It does not mean:
* There are no bugs in the new code
* The variant filters are working properly 

## I don't want to deal with this moke task stuff, can I still use these tests?
The tests in `./rlibs/carat/test/generic.R` are reasonably general, and most of them will pass on any vault, so you can in theory run 
```
v <- 'path/to/my/favorite/vault.rds'
testthat::test_file('./rlibs/carat/test/generic.R')
```
