# inslurmtion

run Slurm inside Slurm as an unprivileged user!

*wait, but why?* 
When working on a large HPC system where allocation is done node wise,
and we have workloads which use cores as "consumable resources", it's
easier to organize work as a huge array of single core jobs, and just
ask the HPC system for a block of nodes, and internally run some workload
management to distribute smaller jobs.  Why not just reuse Slurm with
a `CR_core` config inside a large allocation?

## check it out

Create files for both
```bash
./setup outer 0
./setup inner 10
```
then start outer cluster
```bash
export SLURM_CONF=$PWD/outer/conf
slurmctld
slurmd
```
and check it out,
```bash
$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
outer*       up   infinite      1   idle localhost
```
then start inner cluster
```bash
$ sbatch ./doargs `which slurmctld` -D -f inner/conf
Submitted batch job 2

$ sbatch ./doargs `which slurmd` -D -f inner/conf
Submitted batch job 3

$ grep started *.out
slurm-2.out:slurmctld: slurmctld version 15.08.7 started on cluster inner
slurm-3.out:slurmd: slurmd version 15.08.7 started
slurm-3.out:slurmd: slurmd started on Tue, 25 Dec 2018 21:00:31 +0100

$ SLURM_CONF=inner/conf sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
inner*       up   infinite      1   idle localhost
```
while on the outer cluster we can see
```bash
$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
outer*       up   infinite      1    mix localhost
```
