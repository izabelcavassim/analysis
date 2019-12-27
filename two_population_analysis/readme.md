# Readme for two population analysis pipeline

This directory contains the code to run analysis of
demographic inference using dadi, fastsimcoal2, and smc++ programs,
on the multi-population demographic models found in `stdpopsim`.
Currently, this pipeline is only set up to run on the Gutenkunst 2009
model for humans and Li & Stephan 2006 model for Drosophila.

The Snakemake workflow includes the necessary pipeline
for simulation, analyses, and plotting in an efficient manner.
Simply choose your parameters in a config file,
and let snakemake handle the rest.
(For large runs, the use of a cluster is highly encouraged)

_NOTE_: This requires a bleeding edge install of `msprime` at the
moment.


## Dependencies
Dadi needs to be installed before running this pipeline: https://bitbucket.org/gutenkunstlab/dadi/wiki/Installation  

```
git clone https://bitbucket.org/gutenkunstlab/dadi
cd dadi
python setup.py install
cd ..
```
To install fastsimcoal2, download the source code into the two population directory and unzip:
```
wget http://cmpg.unibe.ch/software/fastsimcoal2/downloads/fsc26_linux64.zip
unzip fsc26_linux64.zip && rm fsc26_linux64.zip && mv fsc26_linux64 fsc26
```

## Workflow

The analysis includes three programs for inferring population size, split times,
and migration rates from two populations:
[dadi](https://bitbucket.org/gutenkunstlab/dadi/src/master/),
[fastsimcoal2](http://cmpg.unibe.ch/software/fastsimcoal2/), and
[smc++](https://github.com/popgenmethods/smcpp).

To run an analysis, create a directory (wherever you want)
where all results, and intermediate files will be stored.   
Next, create and place a file named `config.json` in it.
The json file must contain key : value combos described below. An example
might look like this:

```json
{
    "seed" : 12345,
    "num_samples_per_population" : [20, 20, 0],
    "replicates" : 1,
    "species" : "HomSap",
    "model" : "OutOfAfrica_3G09",
    "genetic_map" : "HapmapII_GRCh37",
    "chrm_list" : "chr22",
    "mask_file" : "masks/HapmapII_GRCh37.mask.bed"
}
```

Once you have creates a directory which contains the config file
simply run snakemake from _this_ directory (two_population_analysis), and point it to your analysis run
directory, like so

`snakemake -j 40 --config config="/projects/kernlab/jgallowa/homo_sapiens_Gutenkunst_0"`

where `-j` is the number cores available to run jobs in parallel, and
`--config` points to the _directory_ that contains the config file.


### Cluster environments
Our workflow can also be run on a cluster. To do so requires
the setup of a `.json` configuration file that lets `snakemake`
know about your cluster. We have provided an example of
such a file in `cluster_talapas.json` that is for use with a
University of Oregon SLURM cluster.

```json
{
    "__default__" :
    {
        "time" : "02:30:00",
        "n" : 1,
        "partition" : "kern,preempt",
        "mem" : "16G",
        "cores" : "4",
    },
    "run_msmc" :
    {
        "time" : "05:00:00",
        "mem" : "32G",
        "cores" : "4",
    }

}
```

At a minimum, you should
edit the names of the partition to match those on your own HPC.
The workflow can then be launched with the call

`snakemake -j 999 --config config="/projects/kernlab/jgallowa/homo_sapiens_Gutenkunst_0" --cluster-config cluster_talapas.json --cluster "sbatch -p {cluster.partition} -n {cluster.n} -t {cluster.time} --mem-per-cpu {cluster.mem} -c {cluster.cores}"`

(it may prove useful to simply put this command in a bash script)

and jobs will be automatically farmed out to the cluster

### Final output
The current final output is are three plots: one comparing population size  
and divergence time estimates, one comparing migration rate estimates, and  
one showing the population size trajectories inferred from smc++, i.e.,  
`homo_sapiens_Gutenkunst_0/Results/estimates_mig_dadi_fsc.png`  
`homo_sapiens_Gutenkunst_0/Results/estimates_N_tdiv_dadi_fsc_smcpp.png`  
`homo_sapiens_Gutenkunst_0/Results/smcpp_estimated_Ne.png`

## Parameter Description

`seed` : `<class 'int'>`
This sets the seed such that any anaysis configuration
and run can be replicated exactly.

`num_samples_per_population` : `<class 'int'>`
This is the number of samples to simulate for within each population  
for each replicate.

`replicates` : `<class 'int'>` The number of replicate simulations to run and
analyze.

`output_dir` : `<class 'str'>` This where all the intermediate files and results
will be stored. This includes all plots preduced, as well as all input/output files
from the simulations and software runs organized into subdirectories by
replicate seed.

`species` : `<class 'module'>` from `stdpopsim` such as `homosap`.
Chromosomes, demographics, and recombination maps should be derived from this.

`model` : `<class childclass Model>`
This is the class that defines the demography associated with a species. All models
should inherit from `<class Model>`

`genetic_map` : `<class childclass GeneticMap>` This will define the genetic map
used for simulations.

`chrm_list` : `<class 'str'>` A string of the chromosome names you would like to simulate,
separated by commas. All chromosomes simulated will be fed
as a single input into each analysis by the inference programs, for each replicate.
Set to "all" to simulate all chromosomes for the genome.
