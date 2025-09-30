# mcmc-date in an Apptainer container

This repository provides a containerized version of [McmcDate](https://github.com/dschrempf/mcmc-date), developed by Dominik Schrempf ([@dschrempf](https://github.com/dschrempf)), that can date a phylogenetic tree with constraints.

By using this container image, you can omit the necessity of installing [Haskell](https://www.haskell.org/), the computer language `mcmc-date` was written in, the [Glasgow Haskell Compiler](https://www.haskell.org/ghc/), and [Cabal](https://www.haskell.org/cabal/) on your system and just run `mcmc-date` directly instead.

Currently an [Apptainer](https://apptainer.org/) container is provided which is supported by most HPC clusters.

## Running `mcmc-date` through the Apptainer container on your dataset

Go to the directory with your dataset (rooted tree, treelist, calibrations and constraints, etc).
```
cd workdir
```

Let's assume that the data under `workdir/` looks as the following:
```
- workdir/data/rooted.tree
- workdir/data/trees.list
- workdir/data/calibrations.csv
- workdir/data/constraints.csv
- workdir/analysis.conf
```

Clone this repository into the `workdir`:
```
git clone https://github.com/oist/mcmc-date-container.git
```

Create a symlink to the `run` and `analyze` scripts:
```
ln -s mcmc-date-container/run
ln -s mcmc-date-container/analyze
```

Prepare your dataset for the mcmc-date analysis:
```
./run -a -r "$(pwd)/mcmc-date-container/mcmcdate.sif" -f analysis.conf -c -k ug f p
```
where 
- with the `-a` option we ask for `Apptainer` instead of a local Haskell installation,
- with `-r` we specify the absolute path to the Apptainer image, 
- with `-f` we specify the analysis configuration file, 
- with `-c` we activate calibrations, 
- with `-k` we activate constraints, 
- with `ug` we ask for uncorrelated gamma molecular clock model, 
- with `f` we ask for a full covariant likelihood matrix 
- and finally with `p` we run the preparation step of `mcmc-date`.

Run mcmc-date analysis:
```
./run -a -r "$(pwd)/mcmc-date-container/mcmcdate.sif" -f analysis.conf -c -k ug f r
```
where all the options are the same as above except replacing `p` (prepare) with `r` (run), thus running the tree dating analysis.

Analyze the results:
```
./analyze
```

## Files in this repository

- [analyze](./analyze) this wrapper script is a modified version of [McmcDate](https://github.com/dschrempf/mcmc-date)'s `analyze` script including the option (`-a -r PATH`) to call an Apptainer container instead of running it directly in a local Haskell environment. The script is backwards compatible with the original `analyze` script.
- [run](./run) this wrapper script is a modified version of [McmcDate](https://github.com/dschrempf/mcmc-date)'s `run` script including the option (`-a -r PATH`) to call an Apptainer container instead of running it directly in a local Haskell environment. The script is backwards compatible with the original `run` script.
- [mcmcdate.sif](./mcmcdate.sif) Apptainer's sif container image that runs a slim version of Debian Trixie (13.1) and contains the `mcmc-date` and its helper scripts in binary format
- [mcmcdate_debian_global_v2.def](./mcmcdate_debian_global_v2.def) Apptainer container definition file to create a fully functional Haskell environment able to compile `mcmc-date` and its helper scripts
- [mcmcdate_debian_global_v2_multistage.def](./mcmcdate_debian_global_v2_multistage.def) Apptainer multistage container definition file that after creating the `mcmc-date` binaries, it copies them into a fresh Debian installation getting rid of the Haskell environment and reducing final image size
- [example/analysis.conf](example/analysis.conf) Example analysis.conf file for `mcmc-date`. Please note, that even if calibrations and constraints are specified here, they will be only used if you activate them with the corresponding switches for the `run` script (`-c` and `-k`)
- [example/calibrations.csv](example/calibrations.csv) Example time calibrations file (the header line is mandatory). Two leaf names pinpoint their most recent common ancestor node the calibration is to be set on.
- [example/constraints.csv](example/constraints.csv) Example relative constraints file (the header line is mandatory). Two leaf names pinpoint their most recent common ancestor node the constraint is to be set on.

## Usage

### Usage of the `run` script
```
Usage: run [OPTIONS] RELAXED_MOLECULAR_CLOCK_MODEL LIKELIHOOD_SPECIFICATION COMMANDS

Auxiliary data options:
-b Activate braces
-c Activate calibrations
-k Activate constraints

Algorithm related options:
-i NAME  Initialize state and cycle from previous analysis with NAME
-H       Activate Hamiltonian proposal (slow, but great convergence)
-m       Use Mc3 algorithm insteahd of Mhg

Other options:
-f FILE    Use a different analysis configuration file (relative path)
-n SUFFIX  Use an analysis suffix
-p         Activate profiling
-s         Use Haskell stack instead of cabal-install
-a         Use Apptainer image instead of cabal-install or Haskell stack
-r FILE    Absolute path to the mcmcdate Apptainer SIF file, if not set, by default: <script's directory>/mcmcdate.sif is used

Relaxed molecular clock model:
ug  Uncorrelated gamma model
ul  Uncorrelated log normal model
uw  Uncorrelated white noise model
al  Autocorrelated log normal model

Likelihood specification:
f  Full covariance matrix
s  Sparse covariance matrix
u  Univariate approach
n  No likelihood; use prior and auxiliary data only

Available commands:
p  Prepare analysis
r  Run dating analysis
c  Continue dating analysis
m  Compute marginal likelihood

A configuration file "analysis.conf" is required.
For reference, see the sample configuration file.
```

### Usage of the `analyze` script
```
Usage: analyze [options]

Options:
 -h      Display help and exit
 -s      Skip files with previous analysis
 -a      Use Apptainer image instead of cabal-install or Haskell stack
 -r FILE Absolute path to the mcmcdate Apptainer SIF file, if not set, by default: <script's directory>/mcmcdate.sif is used
```


## How to build the container image or modify it

For development:
```
APPTAINER_TMPDIR="/your/local/tmp/dir" apptainer build --sandbox mcmcdate_debian_global_v2 mcmcdate_debian_global_v2.def
```

Now you can enter the image and modify it to your needs via:
```
apptainer shell --writable mcmcdate_debian_global_v2
```

For distribution use the multistage build that removes Cabal, GHC, etc and only keeps the `mcmc-date` binaries and their library dependencies:
```
APPTAINER_TMPDIR="/your/local/tmp/dir" apptainer build mcmcdate.sif mcmcdate_debian_global_v2_multistage.def
```

