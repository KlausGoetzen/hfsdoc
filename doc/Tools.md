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
* [General Concept](../doc/GeneralConcept.md)
* [Code](../doc/Code.md)
* [Getting Started](../doc/GettingStarted.md)
* [Configuration Setup](../doc/ConfigurationSetup.md)
* [Running on Virgo](../doc/Virgo.md)
* [Tools](../doc/Tools.md)
   + [Background Analysis](#background-analysis---hepfastbganalysisc)
   + [PID Function Cache](#pid-function-cache---heppidhistgenc)
   + [PID PDF Generator](#pid-pdf-generator---heppidpdfgenc)
   + [Histogram Drawing](#histogram-drawing---hepdrawhistosc)
* [Demos](../doc/Demos.md)
* [Appendix](../doc/Appendix.md)

# Tools

## Background Analysis - HepFastBgAnalysis.C
[Back to TOC](#table-of-contents)

`HepFastBgAnalysis.C` is a background channel analyzer based on an analysis output file and a channel list. Since all background (BG) channels generated with the `phsp` generator have a unique channel number stored in the branch `chan` (if storage option `evt` is set), the tool uses this information to identify and print the channels sorted by frequence of appearance. The macro `HepFastBgAnalysis.C(<opt>)` expects one parameter string `opt` with the parameters

* `fana`: name of analysis output file
* `tree`: TTree name inside this file
* `fpar`: name of file containing the BG channel information of the form `id=<number> : fs=<decay channel>`
* `cut`: cut to be applied to the TTree for the BG analysis

We can demonstrate the function by running the demo-analysis `cfg/demo_jpsi.cfg` containing a background source for a sufficiently large number of events with

```c++
root -l 'HepFastSim.C+(50000,"cfg/demo_jpsi.cfg")'
```
and afterwards run `HepFastBgAnalysis.C` with
```c++
root -l 'HepFastBgAnalysis.C("fana=ana_demo_jpsi.root : tree=ntp2 : fpar=parms/ftf_pbp.dat : cut=xm>2.8")'
```
This results in the following output:
```
Processing HepFastBgAnalysis.C("fana=ana_demo_jpsi.root : tree=ntp2 : fpar=parms/ftf_pbp.dat : cut=xm>2.8")...
id=1017 : (  8)  anti-p- pi+ n0 
id=1018 : (  6)  anti-n0 pi- p+ 
id=1022 : (  6)  anti-p- pi- pi+ p+ 
id=1006 : (  3)  pi- pi- pi+ pi+ 
id=1001 : (  2)  pi- pi- pi0 pi+ pi+ 
id=1002 : (  1)  pi- pi0 pi0 pi+ 
id=1003 : (  1)  pi- pi- pi0 pi0 pi+ pi+ 
id=1011 : (  1)  anti-p- pi0 p+ 
id=1012 : (  1)  anti-n0 pi- pi0 p+ 
id=1013 : (  1)  anti-p- pi0 pi+ n0 
id=1016 : (  1)  pi- pi0 pi0 pi0 pi0 pi+ 
id=1020 : (  1)  pi- pi- pi- pi0 pi+ pi+ pi+ 
id=1023 : (  1)  anti-p- pi0 pi0 p+ 
id=1045 : (  1)  pi- pi+ eta 
```
We e.g. see that the channel `anti-p- pi+ n0` is the most dominant background channel in the selected mass region `m > 2.8`.

## PID Function Cache - HepPidHistGen.C
[Back to TOC](#table-of-contents)

The macro `HepPidHistGen.C` offers the functionallity to generate a PID histogram map (see parameter `pidmap` in `PID` [detector setup](../doc/ConfigurationSetup.md#pid-particle-identification-detectors) from input functions in ROOT's `TFormula` format. These functions should depend on mass `m` and momentum `p` and might depend on angles theta (`tht`) and `phi`. The code will generate a set of `TH3F`, one for each particle type (e, mu, pi, K, p), with the axis x=p, y=theta, and z=phi, with the bin contents and bin-errors being the expectation values and resolutions, respectively, of the PID quantity for the corresponding phase-space location (p, theta, phi). The macro `HepFastBgAnalysis.C(<opt>)` expects one parameter string `opt` with the parameters

* `fexp`: (m, p, theta, phi)-dependent formula for expectation value
* `fres`: (m, p, theta, phi)-dependent formula for resolution of expectation value
* `fthr`: mass-dependent formula for momentum threshold of PID quantity (e.g. Cherenkov radiation); default: `0.0` (no momentum threshold)
* `part`: set of particle names to generate histograms for; default: `e+,mu+,pi+,K+,p+`
* `masspar`: name of mass parameter; default: `m`
* `fout`: file name to store histograms to; default: `histpid.root`
* `name`: histogram name prefix to be stored to file; default: `hpid`
* `fopt`: file storage option; default: `update` (adds to existing file); `recreate` will overwrite file
* `pmax`: maximum momentum (can be set instead of p-range); default: `6.0`
* `hp`: parameters for momentum (x) axis; default: `100,0.0,pmax` (number of bins, p\_min, p\_max)
* `htht`: parameters for theta (y) axis: default: `100,0,180` (number of bins, theta\_min, theta\_max)
* `hphi`: parameters for phi (z) axis: default: `1,-180,180` (number of bins, phi\_min, phi\_max); assuming no phi dependence -> only 1 bin

The functions `fexp`, `fres`, and `fthr` can make use of an arbitrary set of additional parameters, which need to be defined in `opt`. Depending on the complexities of the functions, caching by corresponding histograms could speed up the simulation a little, but at the expense of granularity (binning). The generated histograms have the names `<name>_<particle>` for all particles in the list `part`, where `<particle>` is the particle name without the charge symbol, i.e. `e`, `mu`, `pi`, `K`, and `p`. The macro allows to add histograms to an existing files, so that in histogram maps for multiple detectors can directly by stored in one file. 

Let us for example create histograms for a Time-of-flight detector with the corresponding PID quantity being the velocity beta = v/c = p/E. The macro call could look like this:

```c++
root -l -b -q 'HepPidHistGen.C("fexp=[p]/sqrt([p]^2+[m]^2) : fres=[p]^2*[dt]*30*sin([tht])/(([p]^2+[m]^2)*[dist]) : dist=100 : dt=0.1 : name=tofb : htht=100,20,140")'
```

which generates the histograms `tof_e`, `tof_mu`, `tof_pi`, `tof_K`, and `tof_p` with axis parameters x=(100,0,6), y=(100,20,140), and z=(1,-180,180) inside the file `histpid.root` for a barrel shaped ToF detector. The additional parameters `dist` (radial distance of the ToF system to the interaction point in cm) and `dt` (time resolution of the detector in ns) are needed for the resolution function `fres`. The factor `30` is the speed of light in units cm/ns. The generated histogram maps can be used in the simulation configuration with

```
PID ;; name=tof_barrel : pidmap=histpid.root, tofb 
```

like pointed out in [this section](../doc/ConfigurationSetup.md#pid-particle-identification-detectors).

## PID Pdf Generator - HepPidPdfGen.C
[Back to TOC](#table-of-contents)

In case the probability density (PDF) for a certain PID quantity is not Gaussian distributed (see figure in the [PID section](../doc/ConfigurationSetup.md#pid-particle-identification-detectors) for examples), there is the possibility to provide the PDF in form of a `TH3F` histogram, where the (x,y) bin corresponds to the phase-space location (p, theta), and the distribution along the z-coordinate represents the normalized PDF for the quantity. The macro `HepPidPdfGen.C` helps to generate these kind of histograms based on an input `TTree` inside a ROOT file. 

In addition it offers an interactive display of histograms already generated (just now or read from the input ROOT file) to inspect the PDF for the various particle types. The macro `HepPidPdfGen.C(<opt>)` expects one parameter string `opt` with the `:` separated parameters

* `fin`: input ROOT file name containing the TTree or the PID histograms
* `var`: variable name of PID quantity inside TTree
* `name`: name prefix of the output histogram; default: \<var\>
* `cut`: pdg-code dependent selection; default: `abs(trpdg)==%d`
* `fout`: output file name; default: `hpidpdf.root`
* `fopt`: file storage option; default: `update` (adds to existing file); `recreate` will overwrite file
* `hp`: parameters for momentum (x) axis; default: `100,0.0,pmax` (number of bins, p\_min, p\_max)
* `htht`: parameters for theta (y) axis: default: `100,0,180` (number of bins, theta\_min, theta\_max)
* `hvar`: parameters for PID variable (z) axis: default: `100,tree min,tree max` (number of bins, var\_min, var\_max)
* `load`: _flag;_ instead of generating the histograms, read from file `fin`; 
* `inspect`: _flag;_ interactive display of histograms; default: `true`  if `load==true`
* `logy`: _flag;_ if `inspect==true`, log-scale for y-axis in 1D-projection
* `logz`: _flag;_ if `inspect==true`, log-scale for z-axis in 2D-projection

For an ROOT input file named `pid10M.root` containing the quantity `muo` (e.g. for the penetation depth of the particle in the iron yoke of a muon detector covering polar angles 20° < theta < 140°) and the true PDG code `trpdg`, a call of the macro could look like:

```c++
root -l -b -q 'HepPidPdfGen.C("fin=pid10M.root : var=muo : hvar=100,0,100 : htht=20,20,140")'
```

This projects the 3D-distributions (p, theta, var) in `TH3F` with dimensions p,theta,var = (100,0,8) x (20,20,140) x (100,0,100) and stores/updates the histograms `muo_e`, `muo_mu`, ... , `muo_p` in the file `hpidpdf.root`. If we would just like to inspect distributions of a PID variable in histograms with prefix `muo` being already stored in a file named `hpidpdf.root`, the command

```c++
root -l 'HepPidPdfGen.C("fin=hpidpdf.root : var=muoiron : load : logy : logz")'
```
can be used. Two windows pop up, where one (named `PDF`) shows the 2D-projections (p, var) for all five particle types (e, mu, pi, K, p), and in the bottom right pad of this window the 1D-projections of the PDFs for all five particle types are displayed superimposed. The corresponding phase space location (p, theta) is picked in the (p, theta) plane shown in the second window (named `PHSP`). When moving the mouse pointer, the 1D-projection is update. By clicking in the `PHSP` window, the location is fixed until clicking again in the plane.

The generated histogram maps can be used in the simulation configuration with

```
PID ;; name=muon_det : pidpdf=hpidpdf.root, muoiron 
```

like pointed out in [this section](../doc/ConfigurationSetup.md#pid-particle-identification-detectors).

## Histogram Drawing - HepDrawHistos.C
[Back to TOC](#table-of-contents)

The macro `HepDrawHistos.C` offers an easy way to plot a bunch of histograms from a ROOT `TTree`. In particular it simply re-draws live histograms from a simulation job based on the output file and the configuration file containing the `HIST` statements. The macro expects up to four input parameters `HepDrawHistos.C(cfg, file, width, stat)`

* `cfg`: name of the config file with the histogram definitions; see [HIST section](../doc/ConfigurationSetup.md#hist---live-histograms) for details
* `file`: input ROOT file name containing the `TTrees` to plot the histograms (auto-generated from cfg-file if not specified)
* `width`: width of pads; default: 450
* `stat`: configuration code for statistic box; default: 10

For example, if we would like to re-draw the plots from the simulation corresponding to `cfg/demo_mini.cfg`, we just need to execute

```c++
root -l 'HepDrawHistos.C("cfg/demo_mini.cfg")'  // -> auto-generates file name "ana_demo_mini.root"
```

displaying the canvas with the two plots as shown in the [introduction](../README.md).
