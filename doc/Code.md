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

Copyright 2025, Klaus Götzen, GSI Helmholtzzentrum für Schwerionenforschung GmbH
License: Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)
```

# Table of Contents

* [Introduction](../README.md)
* [Quick Start](../README.md)
* [General Concept](GeneralConcept.md)
* [Code](Code.md)
   + [Code Structre](#code-structure)
   + [Classes](#classes)
   + [Steering Macros](#steering-macros)
   + [Tools](#tools)
   + [Parameter Files](#parameter-files)
* [Getting Started](GettingStarted.md)
* [Configuration Setup](ConfigurationSetup.md)
* [Running on Virgo](Virgo.md)
* [Tools](Tools.md)
* [Demos](Demos.md)
* [Appendix](Appendix.md)
* [References](References.md)

# Code

## Code Structure
[Back to TOC](#table-of-contents)

The code and parameter data are arranged in the following directory structure. The directory `src/` contains all the classes needed to run the simulation, while the macros to be actually executed are located in the installation directory. The directory `parms/` contains parameters needed by HepFastSim. The particle database `HepFastParticleTable.txt` is mandatory, while the other three files are used for generic event generation and do not need to be present if this feature is not used. The directory `cfg/` contains example configuration files, example setups for the PANDA, BESIII and pCBM detectors, and the file `pid_templates.cfg` with some example PID detector templates for dE/dx, DIRC, E/p, ToF and RPCs.

```
├── cfg
│   ├── demo_3phi.cfg
│   ├── demo_acc.cfg
│   ├── demo_dispvtx.cfg
│   ├── demo_fitmass.cfg
│   ├── demo_gen.cfg
│   ├── demo_genbg.cfg
│   ├── demo_gluex.cfg
│   ├── demo_jpsi.cfg
│   ├── demo_mini.cfg
│   ├── demo_pid.cfg
│   ├── demo_ppcharm.cfg
│   ├── demo_rhopi.cfg
│   ├── demo_var.cfg
│   ├── demo_vtx.cfg
│   ├── det_besiii.cfg
│   ├── det_panda.cfg
│   ├── det_panda_eff.cfg
│   ├── det_pcbm.cfg
│   ├── hpid_panda.cfg
│   └── pid_templates.cfg
├── doc
│   ├── Appendix.md
│   ├── Code.md
│   ├── ConfigurationSetup.md
│   ├── Demos.md
│   ├── GeneralConcept.md
│   ├── GettingStarted.md
│   ├── Tools.md
│   ├── Virgo.md
│   ├── acc.png
│   ├── demo.png
│   ├── demo_acc.png
│   ├── demo_gen.png
│   ├── demo_jpsi.png
│   ├── demo_pid.png
│   ├── demo_var_1.png
│   ├── demo_var_2.png
│   ├── demo_vtx.png
│   ├── pidexample.png
│   └── pidpdf.png
├── parms
│   ├── dpm.dat
│   ├── ftf_pbp.dat
│   ├── ftf_pp.dat
│   └── HepFastParticleTable.txt
├── src
│   ├── HepFactory.C
│   ├── HepFastAux.C
│   ├── HepFastBoxGen.C
│   ├── HepFastCand.C
│   ├── HepFastCandList.C
│   ├── HepFastDetBase.C
│   ├── HepFastDetector.C
│   ├── HepFastEventGen.C
│   ├── HepFastEventShape.C
│   ├── HepFastEvtReader.C
│   ├── HepFastFit4C.C
│   ├── HepFastGenBase.C
│   ├── HepFastNtp.C
│   ├── HepFastPartReader.C
│   ├── HepFastPhspGen.C
│   ├── HepFastPidDetector.C
│   ├── HepFastQATool.C
│   ├── HepFastSimpleCombiner.C
│   ├── HepFastSimulation.C
│   ├── HepFastVtxPoca.C
│   └── HepSmartH.C
├── HepDrawHistos.C
├── HepFastBgAnalysis.C
├── HepFastSim.C
├── HepFastSimDemo.C
├── HepFastSimDemoMini.C
├── HepPidHistGen.C
├── HepPidPdfGen.C
├── hfslog.sh
├── job_rootmacro.sh
├── README.md
├── set_rootinc.sh
├── set_virgo.sh
└── subhfs.sh
```

## Classes
[Back to TOC](#table-of-contents)

HepFastSim brings the following core classes for running the simulation:

| Class                 | Function                                                          |
| --------------------- | ----------------------------------------------------------------- |
| HepFastAux.C          | _Commonly used global auxilliary tools_                           |
| HepFactory            | Candidate factory                                                 |
| HepFastCand           | Particle candidate                                                |
| HepFastCandList       | List of particle candidates                                       |
| HepFastEventGen       | Steering class for event generators                               |
| HepFastGenBase        | Generator base class                                              |
| HepFastBoxGen         | Box generator                                                     |
| HepFastEvtReader      | Reader for EvtGen input files                                     |
| HepFastPartReader     | Reader for DPMGen/FTFGen input files (TClonesArrays of TParticle) |
| HepFastPhspGen        | PHSP decayer based on TGenPhaseSpace                              |
| HepFastSimulation     | Core class for event simulation                                   |
| HepFastDetBase        | Detector base class                                               |
| HepFastDetector       | Tracking or EMC detector component                                |
| HepFastPidDetector    | PID-detector component                                            |
| HepFastSimpleCombiner | Core class for combinatorics                                      |
| HepFastFit4C          | 4-constraint fitter                                               |
| HepFastVtxPoca        | Tool for vertexing via POCA (no fitting) and track propagation    |
| HepFastEventShape     | Event based variables like multiplicities, event shapes, etc...   |
| HepFastQATool         | Recursive composite candidate dump                                |
| HepSmartH             | Histogram wrapper for easy histogramming from TTrees              |
| HepFastNtp            | Easy-to-use TTree wrapper                                         |

## Steering Macros
[Back to TOC](#table-of-contents)

In order to run the simulation, three steering macros are provided.

| Macro                | Function                                |
| -------------------- | --------------------------------------- |
| HepFastSim.C         | Main simulation macro                   |
| HepFastSimDemo.C     | Example for running without config file |
| HepFastSimMiniDemo.C | Minimalistic example                    |

## Tools
[Back to TOC](#table-of-contents)

There are also some additional tools bundled with the package:

| Macro               | Function                                   |
| ------------------- | ------------------------------------------ |
| HepDrawHistos.C     | Tool for simple histogram generation       |
| HepFastBgAnalysis.C | Tool for background analysis               |
| HepPidHistGen.C     | PID histogram generator from PID function  |
| HepPidPdfGen.C      | PID pdf histogram generator from TTree     |
| hfslog.sh           | Analyse log files from virgo cluster jobs  |
| job_rootmacro.sh    | Submit single ROOT macro to virgo cluster  |
| set_virgo.sh        | Set environment for cluster job submission |
| subhfs.sh           | Submit jobs to virgo cluster               |

## Parameter Files
[Back to TOC](#table-of-contents)

There are (for the time being) four parameter files bundled with the software:

| File                     | Function                                                                                |
| ------------------------ | --------------------------------------------------------------------------------------- |
| HepFastParticleTable.txt | Particle/decay definitions for particle TDatabasePDG (ROOT); Naming scheme from EvtGen  |
| dpm.dat                  | Parametrizations of anti-p p generic reactions from DPG event generator (1639 channels) |
| ftf_pbp.dat              | Parametrization of anti-p p generic reactions from FTF event generator (5264 channels)  |
| ftf_pp.dat               | Parametrization of pp generic reactions from FTF event generator (3031 channels)        |

Proceed to the next section: [Getting Started](GettingStarted.md)
