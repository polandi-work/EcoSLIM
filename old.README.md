EcoSLIM
=======

**EcoSLIM** private repo for development.  Will be moved to public repo under `/ParFlow` when published

**EcoSLIM** is a Lagrangian, particle-tracking that simulates advective and diffusive movement of water parcels.  This code can be used to simulate age, diagnosing travel times, source water composition and flowpaths.  It integrates seamlessly with **ParFlow-CLM**.

#### Development Team
+ Reed Maxwell <rmaxwell@mines.edu>
+ Laura Condon <lecondon@syr.edu>
+ Mohammad Danesh-Yazdi <danesh@mines.edu>
+ Lindsay Bearup <lbearup@usbr.gov>

To build simply type `make` in the main window, to set number of parallel threads use either
`export OMP_NUM_THREADS=16` for bash or
`setenv OMP_NUM_THREADS 16` for t/c-shell.

To run you will need to have a completed ParFlow simulation and an
EcoSLIM input file that must be named `slimin.txt` and follow the
format described below. Note that this file does not need to be co-located
with the ParFlow simulation.  

To run simply execute `EcoSLIM.exe` from the directory that contains the
`slimin.txt` input file.

`slimin.txt`  Main input file. Includes domain geometry, **ParFlow** timing and input, total number of particles,   initial conditions and information about **CLM**.

including/defining:
* Number of ParFlow grid cells in the x-direction
* Number of ParFlow grid cells in the y-direction
* Number of ParFlow grid cells in the z-direction
* ParFlow grid cell size in the x-direction
* ParFlow grid cell size in the y-direction
* ParFlow grid cell size in the z-direction
* Number of particles per cell at start of simulation
* Total number of particles to be tracked
* ParFlow timestep
* ParFlow file number to start from
* ParFlow file number to stop at
* Velocity multiplier: 1.0=forward tracking, -1.0=backward tracking
* A logical parameter that turns CLM Evap Trans reading on / off
* Number of particles entering domain via Evap Trans
* Density of water  
* A logical parameter that turns CLM output reading on / off
* Molecular Diffusivity
* Fraction of Dx/Vx (also Dy/Vy and Dz/Vz) for numerical stability
* Number of concentration constituents

### Example format for this file

```
SLIM_hillslope   ! SLIM run name, path to ParFlow files follows, then DEM file
"/EcoSLIM/hillslope_clm/hillslope_clm"
"ER_dem.pfb"
20          !nx
5           !ny
5           !nz
20          !particles per cell at start of simulation (-1 = use restart file)
11000000    !np Total
5.0         !dx
0.2         !dy, dz follows
0.1, 0.1, 0.1, 0.1, 0.1
1.0         ! ParFlow DT
1          ! ParFlow t1: ParFlow file number to start from (initial condition is pft1-1)
1752       ! ParFlow t2: ParFlow file number to stop at
1          ! EcoSLIM output start counter 0=pft1
2          ! Time Sequence Repeat [n_cycle*(pft2-pft1)]
0         ! ipwrite frequency, controls an ASCII, .3D particle file not recommended due to poor performance
0         ! ibinpntswrite frequency, controls VTK, binary output of particle locations and attributes
0         !  etwrite frequency, controls ASCII ET output
24        ! icwrite frequency,controls VTK, binary grid based output where particle masses, concentrations
1.0d0       ! velocity multiplier 1.0=forward, -1.0=backward
True         ! CLM Evap Trans Read logical
True           ! CLM Variables Read logical
10          ! number of particles per Evap Trans IC
1000.0      ! density H2O
0.00000414   ! Molecular Diffusivity
0.5d0       ! fraction of Dx/Vx for numerical stability
```