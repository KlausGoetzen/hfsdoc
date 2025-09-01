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
   + [Features](#features)
   + [Limitations](#limitations)
* [Code](../doc/Code.md)
* [Getting Started](../doc/GettingStarted.md)
* [Configuration Setup](../doc/ConfigurationSetup.md)
* [Running on Virgo](../doc/Virgo.md)
* [Tools](../doc/Tools.md)
* [Demos](../doc/Demos.md)
* [Appendix](../doc/Appendix.md)

# General Concept
[Back to TOC](#table-of-contents)

The general concept of the simulation consists of the following steps:

1. Event generation
   * Generate or read event input as 4-vectors and original vertices (with mother-daughter relations)
2. Simulation/reconstruction
   * Perform acceptance cuts and smear particle properties (p, theta, phi, vertex) based on the provided detector information 
     * Neutrals (gamma) can only hit one EMC detector and get energy and direction smearing
     * Charged particles can traverse multiple tracking detectors in acceptance, the overall resolution of momentum and direction is a combination of the individual numbers.
   * Add PID information to charged candidates from all PID detectors and compute a combined (total) PID information
3. Analysis
   * Reconstruct composites by combining particle candidates or previously defined composites
   * Optionally apply selection criteria to the basic or composite lists
   * Dump the candidates from lists to TTrees and finally write to output file
4. Live histogramming
   * Any branch inside these TTrees can be used for live histogramming

Concerning parameter settings and configuration, the main philosophy is the need to specify as few as possible, and most
is anticipated as reasonable default settings.

## Features
[Back to TOC](#table-of-contents)

* Configuration by simple config file
* Event generation
  * EvtGen input file
  * DPMGen/FTFGen input file
  * Box generator
  * Phase space generator (HepFastPhspGen)
* Simulation
  * Tracking detector
  * Electromagnetic calorimetry
  * Particle identification detectors
* Analysis
  * Simple combinatorics
  * Selection
  * Event shape variables
  * Full MC truth matching
  * Simple vertexing
  * 4-constraint fitting
* Output
  * Simple N-tuple dump (Reco + MC)
  * Live histograming 

## Limitations
[Back to TOC](#table-of-contents)

Since we explicitly deal with a _Fast Simulation_ there is certainly a loss of accuracy and precision when compared to a microscopy full simulation based on Geant. Therefore we should mention some of the limitations of the above discussed concept.

* Detector geometry definitions are mostly limited to **simple angular coverage** (polar angle theta, phi). When an optional distance to the detector is specified, one can setup either barrel shapes around the z-axis or discs and rectangular shapes in forward and backward direction.
* In case of a finite magnetic field strength close to the interaction point, **charged trajectory bending is ignored** for acceptance checks.
* **Explicit multiple scattering** effects of particles traversing detector material **is ignored**. It can effectivly be added by setting appropriate direction resolution of particles.
* **Likelihoods for resolutions** of tracking and EMC detectors **are always Gaussian**. For PID detectors there is a way to specify non-Gaussian likelihoods of measured variables in a limited way by momentum/theta-dependent histograms.
* **PID detectors can only provide one relevant quantity**, like e.g. dE/dx or cherenkov angle.
* **Beams are strictly along the z-axis**, i.e. it is (currently) not possible to define a tilt angle.
* There is **no smearing of the position of the interaction point**. The IP is exactly located at (x,y,z) = (0,0,0).

Proceed to the next section: [Code](../doc/Code.md)
