# mcmc-date in an Apptainer container

## How to run mcmc-date through the container

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

Run mcmc-date analysis:
```
./run -a -r "$(pwd)/mcmc-date-container/mcmcdate.sif" -f analysis.conf -c -k ug f r
```

Analyze the results:
```
./analyze
```

## How to build the container image or modify it

For development:
```
APPTAINER_TMPDIR="/local/tmp/dir" apptainer build --sandbox mcmcdate_debian_global_v2 mcmcdate_debian_global_v2.def
```

Now you can enter the image and modify it to your needs via:
```
apptainer shell --writable mcmcdate_debian_global_v2
```

For distribution use the multistage build that removes Cabal, GHC, etc and only keeps the mcmc-date binaries and their library dependencies:
```
APPTAINER_TMPDIR="/local/tmp/dir" apptainer build mcmcdate.sif mcmcdate_debian_global_v2_multistage.def
```
