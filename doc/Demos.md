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

* [Introduction](README.md)
* [Quick Start](README.md)
* [General Concept](doc/GeneralConcept.md)
* [Code](doc/Code.md)
* [Getting Started](doc/GettingStarted.md)
* [Configuration Setup](doc/ConfigurationSetup.md)
* [Running on Virgo](doc/Virgo.md)
* [Tools](doc/Tools.md)
* [Demos](doc/Demos.md)
   + [Generator Only](#generator-only)
   + [Geometric Acceptance](#geometric-acceptance)
* [Appendix](doc/Appendix.md)

# Demos

In the folder [`cfg`](cfg) are several example configuration files addressing different aspect and possibilities to run **`HepFastSim`**. In the following some of the demos are discussed a bit more detailed.

## Generator only
[Back to TOC](#table-of-contents)

It is possible to completely skip the simulation part (acceptance cuts, efficiency, smearing, combinatorics, ...) and use `HepFastSim` only as a generator tool for decay patterns. This is demonstrated in the configuration file [`cfg/demo_gen.cfg`](cfg/demo_gen.cfg).
```
OPT  ;; nmc : print=5000 : hconf=450,3

GEN  ;; phsp : ecm=5,0.01 : reaction=anti-p-,p+ : fixtarget 
GEN  ;; phsp : c=1 : f=0.5 : dec = pbarpSystem -> omega pi+ pi- ; omega -> pi+ pi-  
GEN  ;; box  : c=2 : f=0.5 : p=1,2 : tht=22,140 : pdg=pi+- : mult=1

# simple p vs theta plot from box generator (chan==2)
HIST ;; tree=nmc : cut=chan==2 : hist=0,3,0,180 : var=p,tht*57.3 : opt=box : title=box generator;p [GeV/c];\theta [deg]

# generated omega mass from ppbar -> omega (-> pi+ pi-) pi+ pi-
HIST ;; tree=nmc : cut=chan==1 : hist=0.70,0.86 : var=m[1] : divx=505 : title=generated omega mass;m(\omega) [GeV/c^{2}]

# Dalitz plot of omega pi+ pi- (the needed masses can be computed from the 4-vector informations)
HIST ;; tree=nmc : cut=chan==1&&abs(m[1]-0.782)<0.3 : hist=0,26,0,26 : ...
        var=((e[1]+e[4])^2-(px[1]+px[4])^2-(py[1]+py[4])^2-(pz[1]+pz[4])^2),((e[1]+e[5])^2-(px[1]+px[5])^2-(py[1]+py[5])^2-(pz[1]+pz[5])^2) : ...
        title=Dalitz plot;m^{2}_{\omega\pi^{+}} [GeV^{2}/c^{4}];m^{2}_{\omega\pi^{\minus}} [GeV^{2}/c^{4}] 
```
_Explanation:_
* In the line `OPT` we need to **enable the MC tree** generation by setting **`nmc`**, from which we want to generate histograms
* The three `GEN` statements configure **two generators**:
   + the **`phsp`** generator with a initial reaction and the decay **pbar p -> omega pi+ pi-** (channel `c=1`)
   + the **`box`** generator generating single pions (`mult=1`) with **p = 1 .. 2 GeV/c** and **theta = 22° .. 140°** (channel `c=2`)
* Three histograms are created:
   + **p vs theta** coverage of the box generator
   + the **generated mass of omega(782)** from decays pbar p -> omega (-> pi+ pi-) pi+ pi-
   + the **omega pi+ pi- Dalitz plot** (the expressions for the square masses need to be computed from the 4-vectors)

![(Generator only demo plot)](doc/demo_gen.png)

## Geometric Acceptance
[Back to TOC](#table-of-contents)

Since it is possible to setup detector spatial coverage in different ways, the file `cfg/demo_acc.cfg` demonstrates and visualizes some possibilities.
```
# ----- generator ------
GEN ;; box : p=2,2 : tht=0,180 : costht : pdg=pi+  : mult=1

# ----- detectors ------
TRK  ;;  name = trk1 : dist=100  : wall=70,50,20,15   # forward wall center (140x100 cm with hole of 40x30cm) 
TRK  ;;  name = trk2 : dist=100  : wall=70,10,,,,70   # forward wall top    (140x20 cm, shifted up by 70cm)
TRK  ;;  name = trk3 : dist=100  : wall=70,10,,,,-70  # forward wall bottom (140x20 cm, shifted down by 70cm)
TRK  ;;  name = trk4 : dist=-100 : tht=140,170        # backward disc (hole for 170 deg < theta < 180 deg)
TRK  ;;  name = trk5 : dist=65   : tht=60,120         # barrel

# ----- reconstruction ------
REC  ;;  store(trk, ntp0) = evt,cand

# ----- plots ------
HIST ;; tree=ntp0 : hist=100,-100,100,100,-100,100 : opt=box : cut=xmct&&xtht<1.57 : var=(100-xz)*tan(xtht)*cos(xphi)+xx,(100-xz)*tan(xtht)*sin(xphi)+xy : title=forward wall;x_{hit} [cm];y_{hit} [cm] 
HIST ;; tree=ntp0 : hist=100,-100,100,100,-100,100 : opt=box : cut=xmct&&xtht>1.57 : var=(100-xz)*tan(xtht)*cos(xphi)+xx,(100-xz)*tan(xtht)*sin(xphi)+xy : title=backward disc;x_{hit} [cm];y_{hit} [cm]
HIST ;; tree=ntp0 : hist=90,0,180,90,-180,180      : opt=box : var=xtht*57.3,xphi*57.3 : title=\theta-\phi-coverage;\theta [deg];\phi [deg]
```
_Explanation:_
* Setup of a very **simple box generator** with mono-energetic single pions with **p = 2 GeV/c in full theta range**
* Define **five detector** components:
   + `trk1`: **central foward wall** of size 140x100cm **with a hole** 40x30cm at z = 100cm
   + `trk2`: **upper forward wall** of size 140x20cm **shifted up** by 70cm at z = 100cm 
   + `trk3`: **lower forward wall** of same size **shifted down** by 70cm at z = 100cm 
   + `trk4`: **backward disc** defined by angular range theta = 140° .. 170° at z = -100cm 
   + `trk5`: **barrel** parallel to z-axis defined by angular range theta = 60° .. 120° with radius r = 65cm
* Store tree with **tracks candidates only**
* Generate three plots:
   + Hit map of **all three foward walls** by computing track positions at z = 100cm from theta, phi 
   + Hit map of **backward disc** by computing track positions at z = -100cm from theta, phi
   + Hit map **theta vs phi** for all detectors

![(Acceptance demo plot)](doc/demo_acc.png)
