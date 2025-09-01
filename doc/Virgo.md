```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@                                                                                                       @
@    @@     @@ @@@@@@@@ @@@@@@@@     @@@@@@@@    @@@     @@@@@@  @@@@@@@@     @@@@@@  @@@@ @@     @@    @
@    @@     @@ @@       @@     @@    @@         @@ @@   @@    @@    @@       @@    @@  @@  @@@   @@@    @
@    @@     @@ @@       @@     @@    @@        @@   @@  @@          @@       @@        @@  @@@@ @@@@    @
@    @@@@@@@@@ @@@@@@   @@@@@@@@     @@@@@@   @@     @@  @@@@@@     @@        @@@@@@   @@  @@ @@@ @@    @
@    @@     @@ @@       @@           @@       @@@@@@@@@       @@    @@             @@  @@  @@     @@    @
@    @@     @@ @@       @@           @@       @@     @@ @@    @@    @@       @@    @@  @@  @@     @@    @
@    @@     @@ @@@@@@@@ @@           @@       @@     @@  @@@@@@     @@        @@@@@@  @@@@ @@     @@    @
@                                                                                                       @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

# Table of Contents

* [Introduction](../README.md)
* [Quick Start](../README.md)
* [General Concept](GeneralConcept.md)
* [Code](Code.md)
* [Getting Started](GettingStarted.md)
* [Configuration Setup](ConfigurationSetup.md)
* [Running on Virgo](Virgo.md)
  + [Preparations](#preparations)
  + [Submitting Jobs](#submitting-jobs)
  + [Output Handling](#output-handling)
* [Tools](Tools.md)
* [Demos](Demos.md)
* [Appendix](Appendix.md)

# Running on Virgo

A major advantage of `HepFastSim` obviously is the running speed, so that a couple of kHz as event simulation rate is to be expected. However, running only a single job at a time nevertheless limits the amount of events. Therefore it is highly desirable to run many jobs in parallel and add the output later on, e.g. on a high performance computing cluster as the **virgo cluster at GSI**.

In the following it is described to install `HepFastSim` on virgo and submit many jobs in parallel to the cluster.

## Preparations
[Back to TOC](#table-of-contents)

First the `HepFastSim` package needs to be installed on a machine with access to the [virgo cluster](https://virgo-docs.hpc.gsi.de/user-guide/overview/software.html) at GSI. When logged in to a local `lxlogin` or `lxpool` machine at GSI, another `ssh` brings you to a virgo submit node (**vae** stands for **Virtual Application Environment**):
```
ssh vae25.hpc.gsi.de
```
There you have access to the `/lustre` file system. Change directory to your personal `/lustre` space, e.g. (for name `johndoe`)
```
> cd /lustre/panda/johndoe
```
Here you need to install a copy of `HepFastSim`, so either clone it again from the git repository or simply copy your other local installation. After doing so, you need to set an appropriate ROOT version and recompile. Then the environment for job submission has to be set properly.
```
> git clone https://git.gsi.de/k.goetzen/hepfastsim.git hfs
...
> cd hfs
hfs > . /cvmfs/eel.gsi.de/debian12-x86_64-linux/root/636-00-vae/bin/thisroot.sh
hfs > . set_rootinc.sh

ROOT_INCLUDE_PATH=/lustre/panda/johndoe/hfs/src/
>>>> Load HepFastSim.C+       to ROOT prompt with 'hfs'  <<<<
>>>> Load HepFastBgAnalysis.C to ROOT prompt with 'hfsb' <<<<
>>>> Load HepDrawHistos.C     to ROOT prompt with 'hfsd' <<<<

hfs > hfs -q               #  => compiles HepFastSim and quits afterwards
...
hfs > . set_virgo.sh main  #  => use '. set_virgo.sh debug' for testing

SLURM_JOB_PARTITION = main
>>>> LIST   all jobs with 'sq'   <<<<
>>>> CANCEL all jobs with 'srm'  <<<<
```
The setting **main** of the last command selects the slurm partition for job submission of run times up to **4 hours**. An alternative is the partition **debug** that has a time limit of **30 min**, which is recommended for testing. The script also creates local subfolders **data, log**, and **slurmlog** in your hfs directory. Now your environment is properly configured to submit jobs.

## Submitting Jobs
[Back to TOC](#table-of-contents)

In order to sumit `HepFastSim` simulation jobs, use the submission script **`subhfs.sh`**:
```
> ./subhfs.sh

Submit array of HepFastSim jobs on VIRGO.

USAGE: ./subhfs.sh <cfg> <arr> <nev> <opt>

 <cfg> : Prefix of config file; must be located in dir cfg/. Output file = 'data/ana_<cfg>_RUNID.root'; log file = 'log/log_<cfg>_RUNID.log'.
 <arr> : Job array range, e.g. '1-100' or '100-199'; number RUNID will be used as rndseed for HepFastSim (default = 1-3).
 <nev> : Number of events per job (default = 10000).
 <opt> : Options for HepFastSim (default = "")

./subhfs.sh demo 100-109 20000 'savehist'
```
The submission scripts uses the script **`job_rootmacro.sh`** to submit single jobs to the virgo cluster. It takes care about **proper random number handling** by adding a `rndseed=RUNID` to the running options, so that each jobs runs on a well defined random number, that is identical to the array number `RUNID` of the job.

As a test you can submit a couple of jobs to the `debug` partition by
```
> . set_virgo.sh debug
> ./subhfs.sh demo_jpsi 1-3 10000 'savehist'

cfg = cfg/demo_jpsi.cfg
arr = 1-3
nev = 10000
opt = savehist

sbatch -a1-3 -- /lustre/panda/johndoe/hfs/job_rootmacro.sh HepFastSim.C+(10000,"cfg/demo_jpsi.cfg","rndseed=RUNID:file=data/ana_demo_jpsi_RUNID.root:savehist") log/log_demo_jpsi_RUNID.log

Submitted batch job 40169269
```
With the command `sq` (alias for `squeue -u $USER`) you can check the job status:
```
      JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
  40169269_1     debug    macro johndoe  R       0:15      1 lxbk1134
  40169269_2     debug    macro johndoe  R       0:15      1 lxbk1134
  40169269_3     debug    macro johndoe  R       0:15      1 lxbk1134
```

## Output Handling
[Back to TOC](#table-of-contents)

After finished, you can find three log-files in `log/log_demo_jpsi_[123].log`, and the three ROOT output files `data/ana_demo_jpsi_[123].root`. First we can analyse how many events in total at which speed were simulated, and what total time it took, by using the script `hfslog.sh`
```
> ./hfslog.sh "log/log_demo_jpsi_*.log"

Evaluation for log/log_demo_jpsi_*.log
Num Events : 30.00 k
CPU Time   : 12 s
Speed      : 2604 Hz
```
You can do the math yourself, but with this speed you could produce about **1 billion events per hour** when running 100 jobs in parallel, i.e. about **10 billion events overnight**! 

In order to merge the ROOT output, use the tool `hadd` bundled with ROOT:
```
> hadd data/mrg_demo_jpsi_30k.root data/ana_demo_jpsi_*.root

Info in <hadd>: target file: data/mrg_demo_jpsi_30k.root
Info in <hadd>: compression setting for all output: 101
hadd Source file 1: data/ana_demo_jpsi_1.root
hadd Source file 2: data/ana_demo_jpsi_2.root
hadd Source file 3: data/ana_demo_jpsi_3.root
hadd Target path: data/mrg_demo_jpsi_30k.root:/

> rl data/mrg_demo_jpsi_30k.root
root [0] 
Attaching file data/mrg_demo_jpsi_30k.root as _file0...
(TFile *) 0x55bbb07fb380
root [1] .ls
TFile**         data/mrg_demo_jpsi_30k.root
 TFile*         data/mrg_demo_jpsi_30k.root
  KEY: TTree    nmc;1   MC truth list
  KEY: TTree    ntp0;1  trk [REC: ]
  KEY: TTree    ntp2;1  J/psi [REC: J/psi -> mu+ mu-]
  KEY: TTree    ntp3;1  pbp [REC: pbp -> J/psi pi+ pi-]
  KEY: TH1F     h_0;1   p track (cms)
  KEY: TH1F     h_1;1   p track (cms) (muons)
  KEY: TH1F     h_2;1   m(#mu^{+}#mu^{#minus})
  KEY: TH1F     h_3;1   m(#mu^{+}#mu^{#minus}) (truth)
  KEY: TH1F     h_4;1   m(#mu^{+}#mu^{#minus}) (combinatoric)
  KEY: TH1F     h_5;1   m(#mu^{+}#mu^{#minus}) (#rho^{0}#mu^{+}#mu^{-})
  KEY: TH1F     h_6;1   m(#mu^{+}#mu^{#minus}) (FTF BG)
  KEY: TH1F     h_7;1   m(J/#psi#pi^{+}#pi^{#minus})
  KEY: TH1F     h_8;1   m(J/#psi#pi^{+}#pi^{#minus}) (xmct)
  KEY: TH1F     h_9;1   m(J/#psi#pi^{+}#pi^{#minus}) (!xmct)
  KEY: TH1F     h_10;1  fitted m(#mu^{+}#mu^{#minus})
  KEY: TH1F     h_11;1  fitted m(#mu^{+}#mu^{#minus}) (raw)
  KEY: TH2F     h_12;1  m^{2}(#pi^{+}#pi^{-}) vs m^{2}(J/#psi#pi^{+})
  KEY: TH1F     h_13;1  PID(#mu^{#pm})
  KEY: TH1F     h_14;1  PID(#mu^{#pm}) (muons)
root [2] nmc->GetEntries()
(long long) 30000
```

