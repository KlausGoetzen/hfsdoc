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
   + [Parameter Assignment Syntax](#parameter-assignment-syntax)
   + [Particle Naming Scheme](#particle-naming-scheme)
   + [Configuration File Keywords](#configuration-file-keywords)
   + [Running the Simulation](#running-the-simulation)
   + [Simulation Output](#simulation-output)
     - [Generator Tree](#generator-tree)
     - [Reconstruction Trees](#reconstruction-tree)
     - [Histograms](#histograms)
* [Configuration Setup](ConfigurationSetup.md)
* [Running on Virgo](Virgo.md)
* [Tools](Tools.md)
* [Demos](Demos.md)
* [Appendix](Appendix.md)

# Getting Started

While it is in principle possible to run and steer the simulation with a generic ROOT macro, it is strongly recommended to make use of the main simulation macro `HepFastSim.C` in combination with an appropriate configuration file.

## Parameter Assignment Syntax
[Back to TOC](#table-of-contents)

Since the all in the configuration is about setting and assigning parameters, let us briefly discuss the syntax. A parameter assignment always is given by an expression of the form

```
<parameter_name> = <parameter_value(s)>
```

Multiple parameter settings can be done by concatenation of individual expressions with colon (`:`) inbetween. The parameter parsing is based on the class `HepParMap` declared inside `HepFastAux.C`. More details about this very useful parameter handler can be found at [Appendix / HepParMap](Appendix.md#hepparmap)..

### Syntax rules

* Any whitespace around `<parameter_name>` is trimmed and not part of the name
* When specified multiple times, the last setting of a parameter supersedes all previous ones (`a=1 : a=42` results in `a=42`)
* Parameters sometimes can/must carry specifiers in parenthesis (e.g. `store(J/psi, ntp1) = evt : pidmu(mu+) = 0.1`)
* Whitespace in comma/semicolon separated lists around the separator character is ignored (only relevant for string values)
* The value `<parameter_value(s)>` can be
  * an integer, float, or string 
  * a comma separated list of integers, floats, or strings, and sometimes mixtures of the three types 
  * a semicolon separated list, used for decay chain definitions
  * a white-space separated list, used for single decay definitions
  * omitted (this has default value of 1 like a flag, e.g. `nosave` means `nosave=1`)
  * omitted, and `<parameter_name>` prefixed with `!` (setting it to 0, e.g. `!nmc` means `nmc=0`)
  * containing LaTeX-like code e.g. for histogram titles (the character `\` will be replaced by `#` for ROOT)
* Depending on the context, default values will be padded in lists that omit parameters in the front or back
* Inside a block protected with `{ ... }`:
  + the string will not be split at `:` and `,` characters
  + protection brackets are removed if being outermost like in `{abcd}`, but not in `a{bcd}`
  + the `{` and `}` must pair-wise match like e.g. in `{ {ad} {bc{de}} }`
* _Examples:_
  * `print=1000 : !nmc : nosave`
  * `f = 0.8 : file = parms/ftf_pbp.dat`
  * `hist=50,10,21,50,0,2.5 : var=xdal01,xdal12`
  * `phsp : ecm = 4.6, 0.00965 : reaction = anti-p-, p+ : fixtarget`
  * `title = fitted m(\mu^{+}\mu^{\minus});m [GeV/c^{2}] : leg = \chi^{2} cut`
  * `dec = pbp -> J/psi pi+ pi- ; J/psi -> mu+ mu-`
  * `pidmu(mu+-) = 0.1,1` can be abbreviated to `pidmu(mu+-) = 0.1` (acts as `>=`)
  * `pidmu(pi+-) = 0,0.1` can be abbreviated to `pidmu(pi+-) = ,0.1` (acts as `<=`)
  * `tht = 20,180` can be abbreviated to `tht=20` if `tht=0,180` is default setting
  * `fcn = { return cos([tht] - TMath::Pi()/2); }` will protect the term `TMath::Pi()` from being split to `TMath` and `Pi()/2`
  * `a={my:name} : b=a,{b,c},d : c={{keep}} : d=my{bra:ces}` will give `a="my:name", b={"a", "b,c", "d"}, c="{keep}", d="my{bra:ces}"` 


## Particle Naming Scheme
[Back to TOC](#table-of-contents)

In the above examples, we already see a couple of particle names used for generation, selection, reconstruction or storage. The names to be used correspond to those used by EvtGen. The file `HepFastParticleTable.txt` defines the names and in principle can be searched. The particle codes follow the code rules by the Particle Data Group (PDG). Since it is sometimes handy to include the charged conjugates, there are two ways to do so:

+ In case of charged particle names ending with `+` or `-`, one can simply use a `+-` instead, i.e. `mu+-` corresponds to `mu+, mu-`. 
+ For neutral particles the expression `cc` can be appended, so that e.g. `Lambda0 cc` corresponds to `Lambda0, anti-Lambda0`.

This is in particular relevant for single particles like composites to be stored to TTree, so that the charged conjugates are stored to the same tree. The other occasion are particle lists specified for the `box` generator or veto and trigger lists (`veto` and `trig`) for the generators `partreader` and `phsp`.

## Configuration File Keywords
[Back to TOC](#table-of-contents)

The details about the configuration are described in the chapter [Configuration Setup](ConfigurationSetup.md). There are four main catagories that need to be configured, which are

* **Event generation** or input
* **Detector** configuration for simulation
* **Reconstruction/analysis** and storage
* **Live histograms** (optional) 

The available **keywords and their meanings** are

| Keyword | Meaning                                                |
| ------- | ------------------------------------------------------ |
| INC     | Include another config file (recursively)              |
| OPT     | General steering options                               |
| ADD     | Add particle/decay to particle database (TDatabasePDG) |
| GEN     | Generator/event reader configuration                   |
| TRK     | Definition of a tracking detector                      |
| EMC     | Definition of an EMC (gamma) detector                  |
| PID     | Definition of a PID detector                           |
| REC     | Reconstruction/storage of composites                   |
| HIST    | Specification of live histograms                       |


The configuration is done line by line, where each line starts with a capital letter keyword followed by a double-semicolon, followed by a colon separated list of parameter and configuration settings. 

```
<KEYWORD>  ;;  <param_1> = <value(s)_1> : <param_2> = <value(s)_2> :   ..   : <param_n> = <value(s)_n>
```

A config line can be split into several physical line by adding three periods (`...`) at the end of a line and continue it in the next line. I.e. the next two configurations are identical:

```
<KEYWORD>  ;;  <param_1> = <value(s)_1> : <param_2> = <value(s)_2> : <param_3> = <value(s)_3>  # some comment

<KEYWORD>  ;;  <param_1> = <value(s)_1> : ...   # some comment 1
               <param_2> = <value(s)_2> : ...   # some comment 2
               <param_3> = <value(s)_3>         # some comment 3
```

_Do not forget the colon between parameter assignments_ (e.g. before `...`) when using this line split option. The available list of parameters depends on the keyword. Empty lines as well as everything after a hash `#` symbol is ignored by the configuration parser, so the hash acts as a comment symbol. 

_Example:_
```txt
GEN  ;; phsp : p1=10.3 : reaction=anti-p-,p+ 
GEN  ;; phsp : dec = pbarpSystem -> J/psi pi+ pi- ; J/psi -> mu+ mu-

TRK  ;; name=trk : tht=5,160 : eff=0.95 : dp=1.5 : dtht=2 : dphi=2
PID  ;; name=pid : tht=5,160 : eff=0.95 : mres = 0.2

REC  ;; dec = J/psi -> mu+ mu- ; pbarpSystem -> J/psi pi+ pi- : ...
        pidmu(mu+-)=0.1 : store(pbarpSystem, ntp0)=cand

HIST ;; tree=ntp0 : title=\bar{p}p mass;m [GeV] : hist=4.2,5.0 
HIST ;; tree=ntp0 : title=J/psi mass;m [GeV]    : hist=2.9,3.3 : var=xd0m
```

## Running the Simulation
[Back to TOC](#table-of-contents)

The main simulation macro is **`HepFastSim.C`**. It steers the simulation process in a comprehensive but easy-to-use way. It should be loaded (ideally compiled, making it run twice as fast) to the ROOT prompt, where multiple simulation runs can be performed in a single ROOT session. In order to run (and compile) the code, the include path has to be set by sourcing the file `set_rootinc.sh`. This adds the `src/` directory to the `ROOT_INCLUDE_PATH` and at the same time sets some handy aliases
```
> . set_rootinc.sh

ROOT_INCLUDE_PATH=/<your place>/HepFastSim/src/
>>>> Load HepFastSim.C+       to ROOT prompt with 'hfs'  <<<<
>>>> Load HepFastBgAnalysis.C to ROOT prompt with 'hfsb' <<<<
>>>> Load HepDrawHistos.C     to ROOT prompt with 'hfsd' <<<<
```

To load and compile the macro to the prompt just run
```
> root -l HepFastSim.C+ 
```
or simpler
```
> hfs 
```
The compilation process takes some time when run for the first time, but loading of the library afterwards is much quicker than loading the code uncompiled. In the ROOT prompt, when called without parameters, usage instructions are printed:
```
root [0] HepFastSim()  // or alternatively HFS()

HepFastSim - Copyright (C) 2025 (GPLv3) - Author: Klaus Goetzen (GSI) - Distributed WITHOUT ANY WARRANTY.

USAGE: HepFastSim(int nev, TString cfg, TString opt="")

  nev  = number of events to run
  cfg  = configuration file name
  opt  = additional ':'-separated options in the form '<key> = <value>' (appended to 'OPT' setting in cfg file)
   - det       = name of detector configuration file; overrides cfg for detector config
   - ana       = name of analysis configuration file; overrides cfg for analysis config
   - gen       = name of generator configuration file; overrides cfg for generator config
   - rndseed   = globale random seed (default: 0)
   - nmc       = generate ntuple with generated MC candidates (default: 0)
   - pidmode   = PID mode (default: 'chi2'; anything else switches to LH mode)
   - mcphot    = number of low energetic additional photons allowed for MC truth match (default: 5)
   - mcethresh = photon threshold energy for MC truth match (default: 0.03 GeV)
   - bzfield   = Bz-field in [T]: 0.0 = do not run poca finder (default: 0)
   - prop2ip   = propagate secondaries close to IP (default: 0)
   - hconf     = padwidth,col of histogram canvas (default: 400,4)
   - hopt      = 2D histogram draw option preset
   - legtxt    = text size of legend preset (default: 0.04)
   - legwid    = width of legend preset (default: 0.25)
   - legmarg   = margin of legend preset (default: 0.3)
   - noleg     = do not draw legend on histograms
   - nostat    = do not draw statistics box on histograms
   - file      = output file name (default: ana_<config-filename>_<tag>.root)
   - mode      = arbitrary mode number stored in the n-tuples (default: 0)
   - tag       = arbitrary tag for file name
   - storeopt  = options what information for composite/event shape to store to the output tree (default: 'all')
                 (available: cms, 2body, dalitz, pid, micro, truth, pos, fit, mult, max, sum, shape; veto with !...)
   - savehist  = flag whether to save histos to file (default: 0)
   - savetree  = list of tree names to save to file (default: 1=all, 0=none)
   - savenmc   = flag whether to save MC tree to file (default: nmc)
   - savefig   = name to save canvas (default: '' = do not save)
   - nosave    = do not save anything (overrides all save settings above!)
   - print     = event print frequency (default: 1000)
   - info      = print current parameter setting (default: 0)
   - errlvl    = suppress ROOT warning/error output below errlvl*1000 (default: 0)
   - verbose   = verbosity level; (default: 0)

Example: HepFastSim(10000, "cfg/demo_jpsi.cfg", "!nmc : nosave")
```

The parameters `nev` and `cfg` are mandatory, while `opt` is an optional string with parameter settings extending/superseding the [general options OPT](ConfigurationSetup.md#opt---general-options) in the configuration file. 

The available options as listed above can be assigned to different categories:

| Categorie          | Options                                                        |
|--------------------|----------------------------------------------------------------|
| Configuration      | ana, det, gen                                                  |
| Simulation control | rndseed, nmc, pidmode, mcphot, mcethresh, bzfield, prop2ip     |
| Histograms         | hconf, legtxt, noleg, nostat                                   |
| File output        | file, mode, tag, storeopt, savehist, savetree, savenmc, nosave |
| Console output     | print, info, errlvl, verbose                                   |

Running the simulation based on the configuration file `cfg/demo_jpsi.cfg` can simply be done by
```
hfs
...
root [0] HFS(10000, "cfg/demo_jpsi.cfg")
```

## Simulation Output
[Back to TOC](#table-of-contents)

After the simulation finished, several objects are/can be stored:
+ A **tree** containing the information **from the generator(s)** (`nmc`),
+ the **trees** generated during **reconstruction** by `REC` statements in the configuration (see [Configuration Setup](ConfigurationSetup.md#rec---reconstructionanalysis) for details),
+ the **live histograms**, 
+ and a image of the complete **histogram panel**. 

If not specified explicitly by setting `file=<filename>`, or switched off by `nosave` (switching off all output), a ROOT output file named **`ana_<cfg-file>.root`** is created, where trees and histograms are stored.

### Generator Tree
If the option `nmc` is set, the tree containing generator information (namend **`nmc`**) is created. This tree is only needed to generate live histogram(s) from from it by `HIST` statements in the configuration file; otherwise the generation of this tree can be disabled with (`!nmc`). Per default, this tree if stored to the output file, which can be suppressed by the option `!savenmc`. 

The tree `nmc` is structured in the way, that for each event and quantity, an array of values is filled corresponding to the list of particles created in this reaction. The quantities are:

| Quantities         | Meaning                                                 |
|--------------------|---------------------------------------------------------|
| `ev, ntrk, chan`   | Number of event, particles in event, simulated channel  |
| `uid, pdg`         | Unique ID and PDG code of particle                      |
| `p, m`             | Momentum and mass of particle                           |
| `px, py, pz, e`    | Lorentz vector components of particle                   |
| `x, y, z`          | Start vertex of particle                                |
| `tht, phi`         | polar and azimuthal angle of particle (lab system)      |
| `moth, ndau`       | mother particle index and number of daughter particles  |
| `dauf, daul`       | index of first and last daughter particle               |

The array indices of the generated particles are given by the order of creation in the recursive generation process, for example:

+ Decay: `beams -> J/psi pi+ pi- pi0; J/psi -> mu+ mu-; pi0 -> gamma gamma`
+ Index: `[0]=beams, [1]=J/psi, [2]=mu+, [3]=mu-, [4]=pi+, [5]=pi-, [6]=pi0, [7]=gamma, [8]=gamma`

_Examples for TTree::Draw:_
+ Generated J/psi mass: `("m[1]")` or `("m", "pdg==333")`
+ Direction of charged pions: `("tht:phi", "abs(pdg)==211")`
+ Invariant pi+ pi- mass: `("sqrt((e[4]+e[5])^2-(px[4]+px[5])^2-(py[4]+py[5])^2-(pz[4]+pz[5])^2)")`


### Reconstruction Trees
In contrast to the MC tree, the reconstruction trees have an entry (row) per particle. The stored quantities depend on the configuration of the `REC` statement and the options in `OPT`, see details in [Configuration Setup](ConfigurationSetup.md#rec---reconstructionanalysis) and the option `storeopt` in [the previous chapter](#running-the-simulation). There are many quantities available, which are all listed in [the Appendix](Appendix.md#ttree-branch-names). Per default the reconstruction trees are all stored to the output file, which can be suppressed by the option `!savetree`.


### Histograms
All live histogram being created during the simulation process can be stored as well to the output file with the switch `savehist` (default: `false`). They are named/enumerated as `h_<num>` in the order of creation, i.e. `h_0`, `h_1`, etc.

In order to save the histogram panel as image to a file, the option `savefig=<fig name>` can be used, with `<fig name>` being the figures' file name, where the file name extension (`jpg`, `png`, `pdf`, `C` for ROOT macro, `root` for ROOT file, etc) controls the output format.


Proceed to the next section: [Configuration Setup](ConfigurationSetup.md)
