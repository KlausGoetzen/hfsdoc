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
* [Tools](Tools.md)
* [Demos](Demos.md)
* [Appendix](Appendix.md)
   + [TTree Branch Names](#ttree-branch-names)
   + [HepParMap](#hepparmap)

# Appendix

## TTree Branch Names
[Back to TOC](#table-of-contents)

The command `store` in configuration lines with `REC` statement leads to an automatic creation of a `TTree` with many branches, depending on the local storage switches (`evt`, `cand`, `fit4c`, `shape`) and the global storage setting `storeopt` with the switches (`cms, 2body, dalitz, pid, micro, truth, pos, fit, mult, max, sum, shape`). The chosen branch names are quiet compact in order to efficiently work with the resultant trees interactively, the complete list of name prefixes is listed below. Since in general a complete decay tree of a composite particle is stored recursively, the structure of the branch-names is

```
x<postfix>          : mother particle, i.e. root of the decay tree 
xd<i><postfix>      : daughter <i> of mother
xd<i>d<j><postfix>  : daughter <j> of daugher <i> of mother
...
tx<postfix>         : mother particle, i.e. root of the decay tree (MC truth)
txd<i><postfix>     : daughter <i> of mother (MC truth)
txd<i>d<j><postfix> : daughter <j> of daugher <i> of mother (MC truth)
...
fx<postfix>         : mother particle, i.e. root of the decay tree (4C and/or vertex fitted)
fxd<i><postfix>     : daughter <i> of mother (4C and/or vertex fitted)
fxd<i>d<j><postfix> : daughter <j> of daugher <i> of mother (4C and/or vertex fitted)
...

es<postfix>        : event related quantities (a.k.a. "event shape variables")

```

where `<postfix>` encodes the information, e.g. the momentum `px` or muon pid `pidmu` etc. There are also some general information like event number, channel code, etc. which are as well listed below.

 cms, 2body, dalitz, pid, micro, truth, pos, fit, mult, max, sum, shape

| Name (event info) | Meaning                            | Local switch |
| ----------------- | ---------------------------------- | ------------ |
| ev                | running event number               | evt          |
| cand              | running candidate numbner in event | evt          |
| mode              | arbitrary mode number              | evt          |
| chan              | channel number                     | evt          |
| ncand             | number of candidates in event      | evt          |
| nmc               | number of MC tracks in event       | evt          |
| beam...           | 4-vector of beams (see 4-vector)   | evt          |

| Postfix (composites)           | Meaning                                 | Local + global switch |
| ------------------------------ | --------------------------------------- | --------------------- |
| uid                            | unique ID of candidate                  | cand                  |
| mct                            | MC truth match (flag)                   | cand                  |
| trpdg, pdg                     | true and assigned PDG code              | cand                  |
| chg                            | electrical charge                       | cand                  |
| m                              | mass of particle                        | cand                  |
| mm, mm2                        | missing mass (squared) to beams         | cand                  |
| mdif                           | mass difference to first daughter       | cand                  |
| px, py, pz, e                  | 4-vector components                     | cand                  |
| p, pt                          | (transverse) momentum                   | cand                  |
| tht, phi                       | polar theta and phi angle               | cand                  |
| ctht                           | cosine theta                            | cand                  |
| pxcm, pycm, pzcm, ecm          | 4-vector components (cms)               | cand + cms            |
| pcm, ptcm                      | (transverse) momentum (cms)             | cand + cms            |
| thtcm, phicm                   | polar theta and phi angle (cms)         | cand + cms            |
| x, y, z                        | origin/decay vertex of candidate        | cand + pos            |
| len                            | distance vtx-IP (poca vertex)           | cand + vtx            |
| ctau                           | c·tau (decay length)                    | cand + vtx            |
| pang                           | angle between p and (vertex - IP)       | cand + vtx            |
| ldv                            | displacment in units vertex resolution  | cand + vtx            |
| vqa                            | sum distances of tracks to vertex       | cand + vtx            |
| pide, pidmu, pidpi, pidk, pidp | PID probabilities (e, mu, pi, K, p)     | cand + pid            |
| \<PID detector names>          | raw PID detector responses (micro info) | cand + micro          |
| t..m                           | true mass of particle                   | cand + truth          |
| t..px, t..py, t..pz, t..e      | true 4-vector components                | cand + truth          |
| t..p, t..pt                    | true (transverse) momentum              | cand + truth          |
| t..tht, t..phi                 | true polar theta and phi angle          | cand + truth          |
| t..ctht                        | true cosine theta                       | cand + truth          |
| t..x, t..y, t..z               | true origin/decay vertex of candidate   | cand + truth          |
| f..m                           | fitted mass of particle                 | cand + fit            |
| f..px, f..py, f..pz, f..e      | fitted 4-vector components              | cand + fit            |
| f..p, f..pt                    | fitted (transverse) momentum            | cand + fit            |
| f..tht, f..phi                 | fitted polar theta and phi angle        | cand + fit            |
| f..ctht                        | fitted cosine theta                     | cand + fit            |
| oang                           | opening angle                           | cand + 2body          |
| cdecang, decang                | (cosine of) decay angle                 | cand + 2body          |
| m01, m12, m02                  | combined masses within 3-body decays    | cand + dal            |
| dal01, dal12, dal02            | squared masses within 3-body decays     | cand + dal            |
| fchi2, fprob                   | chi2 and probability of 4C fit          | fit4c                 |
| fbest                          | flag for best 4C fitted candidate       | fit4c                 |

| Name (event shape) | Meaning                             | Local + global switch |
| ------------------ | ----------------------------------- | --------------------- |
| esmult             | multiplicity                        | shape + mult          |
| esmultc            | charged multiplicity                | shape + mult          |
| esmultn            | neutral multiplicity                | shape + mult          |
| esmulte            | electron multiplicity               | shape + mult          |
| esmultmu           | muon multiplicity                   | shape + mult          |
| esmultpi           | pion multiplicity                   | shape + mult          |
| esmultk            | kaon multiplicity                   | shape + mult          |
| esmultp            | proton multiplicity                 | shape + mult          |
| espmax             | maximum momentum                    | shape + max           |
| espcmax            | maximum charged momentum            | shape + max           |
| espnmax            | maximum neutral momentum            | shape + max           |
| esptmax            | maximum transverse momentum         | shape + max           |
| esptcmax           | maximum transverse charged momentum | shape + max           |
| esptnmax           | maximum transverse neutral momentum | shape + max           |
| espsum             | sum momentum                        | shape + sum           |
| espcsum            | sum charged momentum                | shape + sum           |
| espnsum            | sum neutral momentum                | shape + sum           |
| esptsum            | sum transverse momentum             | shape + sum           |
| esptcsum           | sum transverse charged momentum     | shape + sum           |
| esptnsum           | sum transverse neutral momentum     | shape + sum           |
| esspher            | sphericity                          | shape + shape         |
| esaplan            | aplanarity                          | shape + shape         |
| esplan             | planarity                           | shape + shape         |
| escirc             | circularity                         | shape + shape         |
| esthrust           | thrust                              | shape + shape         |
| esfw0              | fox wolfram moment H0               | shape + shape         |
| esfw1              | fox wolfram moment H1               | shape + shape         |
| esfw2              | fox wolfram moment H2               | shape + shape         |
| esfw3              | fox wolfram moment H3               | shape + shape         |
| esfw4              | fox wolfram moment H4               | shape + shape         |
| esfw5              | fox wolfram moment H5               | shape + shape         |

## HepParMap
[Back to TOC](#table-of-contents)

The string-based parameter manager `HepParMap` (class located in `src/HepFastAux.C`) is a quite versatile and handy tool to easily handle ROOT macros or functions with many parameters. Instead of letting function headers get longer and longer for more steering options, all parameters are parsed from one single `TString` input parameter. From the users point of view the call of the function then looks like

```
MyFunction(parameter_string)

```

rather than

```c++
MyFunction(par1, par2, par3, par4, ..., par_17, ...)

```

This has the particular advantage, that one just needs to explicitly specify parameters deviating from default, instead of specify _all_ parameters until reaching the one of interest, and it is obvious what parameter is set since it is referred to by name. If we as an example consider a function 

```c++
DrawBW(double mass=1.5, double width=0.05, double range_min=1.0, double range_max=2.0, double spin=1)

```

to draw a relativistic Breit-Wigner, then usually we need to call it

```c++
DrawBW(1.5, 0.05, 1.0, 2.0, 0)

```

to try it with `spin=0`. In addition you need to know all the defaults for the other parameters, and that `spin` is the fifth one. Using `HepParMap` with the defaults set internally properly, the call just looks like

```c++
DrawBW("spin=0")

```

Another advantage of this way of parameter handling is, that you can add new parameters inside the code without the need to change the interface of the function `DrawBW`. 

### Interface

The important part of the interface of the class looks like

```c++
  HepParMap(<opt>, <print>=false, <delim>=":", <assign>="=", <strip> = 3); 
  
  double          GetDbl(<parname>, <default>=0.0);
  int             GetInt(<parname>, <default>=0);
  TString         GetStr(<parname>, <default>="");
  vector<double>  GetVDbl(<parname>, <default>={}, <fill>=false, <delim>=",");
  vector<int>     GetVInt(<parname>, <default>={}, <fill>=false, <delim>=",");
  vector<TString> GetVStr(<parname>, <default>={}, <fill>=false, <delim>=",");
  
  vector<TString> GetParList();
  bool            Exists(<parname>);
  vector<TString> CheckRecall(<verbose>=false, <txt>="Unknown parameters: ");  

```

In the constructor and the method `SetMap`, `<opt>` is the single string parameter, `<print>` the verbose flag, `<delim>` the delimiter between parameter settings, `<assign>` the assignment symbole, and `<strip>` the setting if and how to strip white-space when parsing (3 = strip on both sides). In the accessor methods, `<parname>` is the name of the parameter, `<default>` the corresponding default value, and `<fill>` the flag to fill the current vector of parameters from the default vector, if less values are given, and `<delim>` the delimiter between list values. Internally all parameter values (also lists) are represented as one single string value, and the accessor methods `GetDbl`, ... , `GetVStr` parse this string to bring it to the requested format. The method `GetParList` returns a vector with all parameter names, `Exists` checks whether a parameter was explicitly set, and `CheckRecall` allows to check, whether there were parameters specified not being requested by the macro code. This is helpful to identify typos in parameter names.

In case of multiple appearances of parameter names in `<opt>` the latest setting overrides the former ones, so that e.g.  `<opt>="mass=1.5:width=0.05:mass=2.0"` results in the assignment `mass=2.0`.

### Example Usage

The usage for the above example could look like this:

```c++
void DrawBW(TString opt)
{
  HepParMap pm(opt);

  double mass          = pm.GetDbl("mass",   1.5);
  double width         = pm.GetDbl("width",  0.05);
  vector<double> range = pm.GetVDbl("range", {1.0, 2,0}, true);
  int spin             = pm.GetInt("spin",   1);

  pm.CheckRecall(true);
  ...
}

```

and a call setting these parameters explicitly like

```c++
DrawBW("mass=1.5 : width=0.05 : range=1,2 : spin=0")

```

where the white-space is just here added for readibility but can also be left away. By setting the flag `<fill>` for the parameter `range`, we make sure that this vector has at least two elements. The call of `CheckRecall` reports about all unknown parameters, i.e. a call `DrawBW("masss=1.7:spin=0")` results in the output

```c++
root [1]  DrawBW("masss=1.7:spin=0")
Unknown parameters: masss  

```

and identifies the typo in the intended parameter name `mass`, being very helpful for debugging when operating with many parameter names.


