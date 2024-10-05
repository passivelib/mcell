
# Make Cell (mcell): On-Chip Passive Components Synthesis
## Motivation
Silicon foundries generally offer a limited set of models for passive components in their PDKs. Transformers, despite their importance in single-ended to differential signal conversion and matching network design, are often excluded, with no models provided in the PDK. While models for inductors are typically available, they are usually constrained to a narrow frequency range, up to around 20 GHz insufficient for designers working in the millimeter-wave (MW) frequency range.

To address this issue, EM simulators are often used to model the missing passive components. However, running design iterations and optimizations with an EM simulator in the loop is a slow and laborious process, leading to tape-out delays and increased development costs.

 ![](/mcell/img/all2-1024x609.png)

Make Cell (mcell) is a tool that automates the synthesis of on-chip passive components, generating models that are customized to the required geometry and frequency range for any process node. 

## Supported components
MCELL supports the following components:
- Spiral inductor (inductor-spiral): common mode inductor with arbitrary number of windings
- Symmetric inductor (inductor-symmetric): differential inductor with arbitrary number of windings
- 1:1 transformer (transformer1o1): the secondary is placed in the metal layer directly below the primary to maximize the coupling coefficient
- 1:2 transformer (transformer1o2): the secondary is placed in the metal layer directly below the primary to maximize the coupling coefficient
- 2:1 transformer (transformer2o1): the secondary is placed in the metal layer directly below the primary to maximize the coupling coefficient
- 2:2 transformer (transformer2o2): the secondary is placed in the metal layer directly below the primary to maximize the coupling coefficient
- Spiral transformer (transformer-spiral): the primary and secondary are placed on the same metal layer, supporting an arbitrary number of windings on both sides

These components can be designed with either rectangular or octagonal geometries and can include an optional patterned ground shield. For symmetric components (all except the spiral inductor), a DC feed can be integrated at the center of symmetry.

## How it works 
MCELL can performe the following tasks:
- Generate DRC clean set of GDS files for a given component

  Example:
  ```
  mcell -d 150:200:10 -w 6:10:2 -s 6:10:2 -n 2:5:1 -t inductor-symmetric --pin-lenght=20 --top-metal=TM2 --rect-geometry
  ```
  Output is folder `gdsFile` populated with DRC clear GDS files.
- Generate DRC clean set of GDS files for a given component, and prepare EMX simulation

  Example:
  ```
  mcell -d 150:200:10 -w 6:10:2 -s 6:10:2 -n 2:5:1 -t inductor-symmetric --pin-lenght=20 --top-metal=TM2 --rect-geometry --generate-em
  ```
  The output consists of the following:
  - A `gdsFile` folder containing DRC-clean GDS files
  - A `yFile` folder where simulation results (Y-parameters) will be saved
  - A script, `runEmx.sh`, which needs to be executed to start the simulation

  To start the simulation for all GDS files in the `gdsFile` folder, execute the script by running `source runEmx.sh`. All simulation results  (Y-parameters), will be saved in the `yFile` folder.
    
- Generate FastHenry input files for a given component

  In addition to EMX, MCELL supports FastHenry, which can extract inductance and resistance up to two orders of magnitude faster than a full-wave EM solver.  
  Example:
  ```
  mcell -d 150:200:10 -w 6:10:2 -s 6:10:2 -n 2:5:1 -t inductor-symmetric --pin-lenght=20 --top-metal=TM2 --rect-geometry --fast-henry
  ```
  The output consists of the following:
  - A `fastHenryFile` folder containing FastHenry input files
  - A `zFile` folder where simulation results (Z-parameters) will be saved
  - A script, `runFastHenry.sh`, which needs to be executed to start the simulation
 
  To start the simulation for all FastHenry input files in the `fastHenryFile` folder, execute the script by running `source runFastHenry.sh`. All simulation results  (Z-parameters), will be saved in the `zFile` folder.

## Desigh flow
- Perform a geometry sweep and simulate using FastHenry. FastHenry can model thousands of components in just a few minutes. Based on the extracted parameters (L, Q, k), the user can identify several potential candidates for the desired component.
- Use EMX to more accurately model the selected candidates, conducting a narrower geometry sweep if needed, and then choose the optimal component.

## Cadence Virtuoso Interface
MCELL has dedicated Cadence Virtuoso Interface. It can be started from Layout Editor and resulting cell is directly drown in it.

###  **Your input:**
<img src="/mcell/img/tr1o1sw.png" width="300" height="auto">

- Component type and range of geometric parameters (diameter, metal width, number of turns, spacing between turns, …)
- Every component can have octagonal or rectangular geometry, and it can be with or without ground shield
- Interface on the left side is used to draw layout in Cadence Virtuoso or to sweep geometry using EMX or FastHenry
- For layout drawing every parameter needs to take one value and user should press apply or OK button
- Parameters with the name ending with :: are sweepable
- Sweep format is min:max:step
- In the section Sweep two option are available EMX (SweepEMX) or FastHenry (SweepFastH) to sweep geometry

###  **Sweep using FastHenry:**

 <img src="/mcell/img/trSweepFh.png" width="500" height="auto">

- FastHenry can be used to sweep thousands of different geometries in the range of several minutes
- This is specially valuable for designing transformers (for matching networks), since primary and secondary inductors and coupling coefficient are dependent on each other, and it is difficult to design them independently
- Results (L, Q, k) are calculated at DC
- Once simulation is finished, filter section can be used to show only components with required performance
- Format for filtering is min:max
- Example: if we want to see only transformers with primary in range from 1nH to 1.1nH we should use 1e-9:1.1e-9 for Lp in the Filter section

###  **Sweep using EMX:**

<img src="/mcell/img/trSweepEmx.png" width="500" height="auto">

- EMX is used for better accuracy
- After selecting several geometries based on FastHenry simulation, EMX can be used for fine tuning
- Frequency at which we want to calculate L/Q/k should be chosen
- After selecting component in the Report section, user can plot L/Q/k by applying some of the options from the Plot section
- Simulation results can be saved for latter use
- Format for filtering is min:max or only min for Q and srf
- Example: if we use 10 for Qp it will show all components with Qp higher than 10, if we use 10:15 it will show all components with Qp in range from 10 to 15

Apply for 30 days free evaluation license.

### Contact: info@passivelib.com
