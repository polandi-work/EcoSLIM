EcoSLIM
=======
*EcoSLIM* private repo for development.  Will be moved to public repo under `/ParFlow` when published

*EcoSLIM* is a Lagrangian, particle-tracking code that simulates advective and diffusive movement of water parcels.  This code can be used to simulate age, diagnose travel times, source water composition and flowpaths.  It integrates seamlessly with *ParFlow-CLM*.

#### Development Team
+ Reed Maxwell <rmaxwell@mines.edu>
+ Mohammad Danesh-Yazdi <danesh@mines.edu> **#**
+ Laura Condon <lecondon@email.arizona.edu>
+ Lindsay Bearup <lbearup@usbr.gov>

#### Citation
For more details on the model and if you use EcoSLIM in published work please cite the following reference:  
   *Maxwell, R.M., L.E. Condon, M. Danesh-Yazdi and L.A. Bearup, Exploring source water mixing and transient residence time distributions of outflow and  evapotranspiration with an integrated hydrologic model and Lagrangian particle tracking approach, in review, 2018.*

Building and Running
--------------------
To build *EcoSLIM* simply type `make` in the directory with the main directory where `EcoSLIM.f90` sits.

To set the number of parallel threads use either
`export OMP_NUM_THREADS=16` for bash or
`setenv OMP_NUM_THREADS 16` for t/c-shell.

To run you will need to have a completed *ParFlow* simulation and an
*EcoSLIM* input file that must be named `slimin.txt` and follow the
format described below. Note that the slim input file does not need to be co-located
with the ParFlow simulation.  

To run simply execute `EcoSLIM.exe` from the directory that contains the
`slimin.txt` input file.

Refer to the *Examples* directory described below for example workflows
running  *ParFlow* and *EcoSLIM*

Inputs
--------------------
**Mandatory Inputs**
1. **EcoSlim driver file:** The main *EcoSLIM* input file that includes information on domain geometry, *ParFlow* timing and input, and particle options. This file must be named `slimin.txt`.  Refer to the *Settings* section for a complete description of this file.
2. **ParFlow outputs:** The following *ParFlow* outputs for at least one time must be provided as pfbs.
   * Velocity - In x, y, and z directions. Used for particle advection
   * Saturation - Used to determine the volume of water in a cell for determining particle mass
   * Porosity - Used to determine the volume of water in a cell for determining particle mass

**Optional Inputs**
1. **Elevation:** DEM file for ParFlow domain in pfb format. If no DEM is provided all elevations are set to zero.
2. **Evapotranspiration:** Evapotranspiration files in pfb format (i.e. *out.evaptrans.filenumber.pfb*) that specifies the vertical water flux across the land surface.  This is normally written by *CLM* but can also be written by *ParFlow* . If evaptrans is less than zero particles will be added to the domain if evaptrans is greater than zero particles are removed through ET. **#**
3. **CLM single file output:** Single file CLM output files (i.e. *.C.pfb* files). These are used to distinguish between rain and snow particles. NOTE: *EcoSLIM* assumes the *CLM* outputs have 10 soil layers. If this is not the case you must change *nCLMsoil* in  `EcoSLIM.f90` **#**

Settings
--------------------
The `slimin.txt` file contains all of the settings for an EcoSLIM Simulation.
An example of this file is provided below and other examples are also provided
in the *Examples* folder. Inputs *must* be provided in the oder they appear in the
template shown here.

Here we describe each parameter and what they do:
* **EcoSLIM Runname (runname):** The runname for all of the file outputs
* **ParFlow Runname (pname):** The runname used for the ParFlow simulations. If the *ParFlow*
outputs are not located in the same directory as the *EcoSLIM* run this should also
include the directory path to the *ParFlow* simulation as shown in the example below.
* **DEM File Name (DEMname):** The name of the DEM file for the *ParFlow* simulations. If this line is left blank all elevations will be set to zero. The DEM is used for VTK writing and does not ajust the Z values reported in outputs **#**.
* **ParFlow nx (nx):** Number of grid cells in the x-direction for *ParFlow* domain
* **ParFlow ny (ny):**  Number of grid cells in the y-direction for *ParFlow* domain
* **ParFlow nz (nz):**  Number of grid cells in the z-direction for *ParFlow* domain
* **Number of Initial Particles (np_ic):** If a positive integer is provided this will be the number of particles placed in every grid cell at the start of the simulation. To start from a previous
*EcoSLIM* output set this value to -1. In this case particles will be initialized from the
`runname_particle_restart.bin` file (refer to the outputs section for details on this file).  
* **Total Particles (np)**: The total number of particles allowed in the simulation. If the particle count exceeds this at any point (i.e. through particle addition with initial conditions or rainfall events)
the simulation will exit.
* **ParFlow dx (dx):** *ParFlow* grid cell size in the x-direction
* **ParFlow dy (dy):** *ParFlow* grid cell size in the y-direction
* **ParFlow dz (dz):** *ParFlow* grid cell size in the x-direction. This should be a list separated by comas that is nz long (refer to example below).
* **ParFlow time step (pfdt):** The time step used for the ParFlow simulation. The time units of this are determined by the *ParFlow* simulation and all *EcoSLIM* outputs will have the same units. Currently EcoSLIM assumes a constant time step so this is not compatible with growth times steps in *ParFlow*
* **Starting ParFlow File Number (pft1):** The file number for the *ParFlow* output to start the *EcoSLIM* simulation from. Note that the initial conditions will be set from the file number before this (i.e. the pressure and saturation files used to calcualte particle masses for the initial particle assignment) **#**.
* **Ending ParFlow File Number (pft2):** The file number for the *ParFlow* output to stop the *EcoSLIM* simulation at.
* **EcoSLIM Output Start Counter (tout1):** This initializes the file numbering for the *EcoSLIM* outputs. If this is set to zero then the first *EcoSLIM* output number will be set to match  starting *ParFlow* file number specified above.
* **Time Sequence Repeat (n_cycle):** If the time sequence repeat is greater than one the
*ParFlow* inputs from *pft1* to *pft2* will be repeated the specified *n_cycle* times. (i.e. the total number of time steps simulated will be *n_cycle(pft2-pft1)*)
* **ASCII 3-D Particle File Output Frequency (ipwrite):** Controls ASCII output of particle locations and ages. This output format is not recommended for performance. Refer to the *Outputs* section for the details on this output. If frequency is set to 0 this output will not be written at all, 1 will write the output every timestep, n>1 will write the output every n timesteps (i.e. n=2 writes outputs every other step). If n<0 then all outputs will be written to one file **#**. (Transient outputs files: `runname_transient_particle.filenum.3D`, static output files: `runname_total_particle_trace.3D`)
* **Particle VTK Output Frequency (ibinpntswrite):** Controls VTK, binary output of particle locations and attributes (`runname_pnts_filenum.vtk`).  Refer to the *Outputs* section for the details on this output. If frequency is set to 0 this output will not be written at all, 1 will write the output every timestep, n>1 will write the output every n timesteps (i.e. n=2 writes outputs every other step)
* **Gridded ASCII ET Output Frequency (etwrite):** Controls ASCII grid based ET output (`runname_ET_summary.filenum.out.txt`). Refer to the *Outputs* section for the details on this output. If frequency is set to 0 this output will not be written at all, 1 will write the output every timestep, n>1 will write the output every n timesteps (i.e. n=2 writes outputs every other step)
* **Gridded VTK Output Frequency (icwrite):** Controls VTK, binary grid based output (`runname_cgrid_filenum.vtk`). Refer to the *Outputs* section for the details on this output. If frequency is set to 0 this output will not be written at all, 1 will write the output every timestep, n>1 will write the output every *n* timesteps (i.e. n=2 writes outputs every other step)
* **Velocity Multiplier (V_mult):**  Set to 1.0 for forward particle tracking and -1.0 for backward particle tracking
* **CLM Evapotranspiration Flag (clmtrans):** Logical flag (1=True, 0=False) indicating whether evapotranspiration outputs should be read (`pfrunname.out.evapotrans.filenum.pfb`). These files are normally generated by *CLM* but can also be generated by *ParFlow* (refer to examples for *ParFlow* and *ParFlow-CLM* test cases).  If this is set to false then no particles are added or removed through recharge and ET. **#**
* **CLM Variable Read Flag (clmfile):** Logical flag (1=True, 0=False) indicating whether CLM variables should be read to classify recharge events as rain and snow. If false then all recharge events are assumed to be rain. **#**
* **Number of Flux Particles (iflux_p_res):** The number of particles to be added per grid cell per ParFLow time step when there is a positive flux into the domain.
* **Density of Water (denh20):** Density of water [M/L3], used for mass calculations. The units should match the mass and length units set by the *ParFlow* simulation.
* **Molecular Diffusivity(moldiff):** Molecular diffusivity **##**
* **Numerical Stability Scaler (dtfrac):** Controls on the maximum timestep a particle can take to ensure numerical stability (i.e. in one dimension: Particle Timestep= min[(pfdt)( dtfrac), (dtfrac)(dx/Vpx)], where Vpx is the velocity in the x direction)  **##**


**Example format for the EcoSLIM input file**
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

Outputs
-----------------
### Single File Outputs
+ **EcoSLIM log:**  A log of the settings used for the simulations and any warnings that occur.
  * File name: `runname.out.log`
+ **Restart:** Binary file containing all the particle information for all active particles at the end of the simulation.
  * File name: `runname_particle_restart.bin`
  * Variables:  *X, Y, Z, Age, Saturated_Age, Mass, Source, Status, Particle_Concentration, Exit_Status*
+ **Exited Particles Log:**
  * File name: `runname_exited_particles.bin`
  * Variables: *Time, X, Y, Z, Age, Mass, Comp, Exit_Status*
+ **ET summary:** Spatially aggregated summary of ET flux composition
  * File name: `runname_ET_output.txt`
  * Variables: Time (*TIME*), mass weighted mean ET age (*ET_age*), mass fraction of ET from initial condition particles (*ET_comp1*), mass fraction of ET from rain particles (*ET_comp2*), mass fraction of ET from snow particles (*ET_comp3*), total ET mass (*ET_mass*), total number of ET particles (*ET_NP*)
+ **Outflow summary:** Spatially aggregated summary of outflow composition
    * File name: `runname_flow_output.txt`
    * Variables: Time (*TIME*), mass weighted mean outflow age (*Out_age*), mass fraction of outflow from initial condition particles (*Out_comp1*), mass fraction of outflow from rain particles (*Out_comp2*), mass fraction of outflow from snow particles (*Out_comp3*), total outflow mass (*out_mass*), total number of outflow particles (*Out_NP*)
+ **Precipitation - ET summary:** Spatially aggregated summary of total precipitation and ET mass fluxes calcualted from the *ParFlow* intputs.
    * File name:`runname_PET_balance.txt`
    * Variables: Time (*TIME*), total precipitation mass (*P[kg]*), total evapotranspiration mass (*ET[kg]*)
+ **Particle ASCII:** Detailed output of every particle in ascii FORMAT written to a single file.
  * File name:`runname_particletrace.3D`
  * Variables: *X, Y, Z, Age*
  * Note: This format is very slow in parallel and is NOT advised for large particle numbers. This file is only written if *ipwrite* is less than zero in `slimin.txt`

### Transient File Outputs
Transient files outputs are written at the frequencies specified in the slimin.txt file (see: *ibinpntswrite*, *icwrite*, *etwrite*, *ipwrite*)
+ **Particle VTK**: Detailed output of every particle in VTK format
  * File name: `runname_pnts_filenum.vtk`
  * Variables: *X, Y, Z, Age, Saturated_Age, Mass, Source, Status, Particle_Concentration, Exit_Status*
+ **Gridded VTK** : Gridded summary of model outputs in VTK format
  * File name: `runname_cgrid_filenum.vtk`
  * Variables: *Grid_Particle_Concentration, Mean_Age, Grid_Mass, Mean_Source, Grid_Concentration, ET_Particles, ET_Mass, ET_Age, ET_Source*
+ **Gridded ASCII ET**: Gridded summary of ET composition  
  * File name: `runname_ET_summary.filenum.out.txt`
  * Variables: *i, j, k, ET_Particles, ET_Mass, ET_Age, ET_Source, Saturation, Porosity*
  * Note: ET variables are defined below, saturation and porosity are from the *ParFlow* inputs.
+ **Particle ASCII:** Detailed output of every particle in ascii FORMAT
  * File name: `runname_transient_particle.filenum.3D`
  * Variables: *X, Y, Z, Age*
  * Note: This format is very slow in parallel and is not advised for large particle numbers

### Output Variable Definitions
**Particle Variables:** Variables for individual particles
+  *X* = X coordinate [L]
+  *Y* = Y coordinate [L]
+  *Z* = Z coordinate relative to the bottom of the domain (i.e. not taking the DEM into account **#**) [L]
+  *Age* = Particle Residence Time [T] **#**
+  *Saturated_Age*= Total time that particle spends in saturated cells [T]
+  *Mass*= Particle Mass [M]
+  *Source* = Particle source type (1= particles initialized in the domain, 2=particles added by a rain event, 3= particles added by a snow event) [-]
+  *Status* = Flag indicating whether particle is currently active (1=active, 0=inactive) [-]
+  *Particle_Concentration*= Placeholder, currently set to 1 (**##**)
+  *Exit_Status* = Flag indicating how inactive particles left the domain (1=outflow, 2=ET) [-]

**Gridded Variables:** Variables aggregated to the *ParFlow* grid cells at every *ParFlow* time step
+ *i* = grid cell location in X (counting starts with 1 **#**) [-]
+ *j* = grid cell location in Y (counting starts with 1) [-]
+ *k* = grid cell location in Z (counting starts with 1) [-]
+ *Grid_Particle_Concentration* = Total active particle mass in cell divided by the mass of water in cell [-]
+ *Mean_Age* = Mass weighted average age of active particles in a cell [T]
+ *Grid_Mass* = Active particle mass in a cell [M]
+ *Mean_Source* = Mass weighted average source of active particles in a cell [-]
+ *Grid_Concentration* = Mass weighted *Particle_Concentration*
+ *ET_Particles* = Number of particles leaving as ET from a cell [-]
+ *ET_Mass* = Total mass of particles leaving as ET from a cell [M]
+ *ET_Age*  = Mass weighted average age of particle leaving as ET from a cell [T]
+ *ET_Source* = Mass weighted average source of particles leaving as ET from a cell [-]

Note: All Length [L] and Time [T] units are set by the ParFlow simulation and spatial coordinates start from the lower left hand corner of the bottom layer consistent with ParFlow (**#**).


Examples and tests contained in the repo
----------------------------------------
The *Examples* folder contains the following test cases. A short description
is provided here. For more details on how to run the examples refer to the
readme files in that directory.
1. **ParFLow_SteadyFlux**: A hillslope domain with constant recharge and ET applied at the top and bottom of the hill respectively.  Example is setup to run *EcoSLIM* on transient *ParFlow* outputs without *CLM*. The documentation for this example includes all the steps for running *ParFlow* and *EcoSLIM*
2. **Example name**:Write short description here **##**
3. **Example name**:Write short description here **##**
4. **Example name**:Write short description here **##**
4. **Example name**:Write short description here **##**
