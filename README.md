
# On-Chip Passive Components Synthesis

Silicon foundries provide a limited set of models for passive components in their PDKs. Even though transformers are very valuable for single-ended to differential signal conversion and vice versa, as well as for matching network design, they are mostly completely omitted and models for those components are not shipped within PDK. With inductors, the situation is a bit different, models are usually provided, but only in a limited frequency range (up to 20 GHz). That is rarely sufficient for designers working on products in the MW frequency range.

In order to solve this problem, an EM simulator needs to be used to model missing passive components. However, design iterations and optimization with an EM simulator in the loop are very slow and tedious, leading to tape-out delays and increased development cost.

 ![](/mcell/img/all2-1024x609.png)

Make Cell (mcell) is a tool that automatize synthesis process of on-chip passive components in the required geometry and frequency range for any process node.

Synthesis can be done in two ways:

## **1. Generate parameterized spice model**

###  **Your input:**

```
mcell -d 150:200:10 -w 6:10:2 -s 6:10:2 -n 2:5:1 -t inductor-symmetric --pin-lenght=20 --top-metal=TM2 --rect-geometry --generate-spice-model
```

- Component type and range of geometric parameters (diameter, metal width, number of turns, spacing between turns, …)

###  **MCELL output:**

- DRC clean set of gds files
- Two scripts runEmx.sh and runModelgen.sh

###   **User to do:**

```
source runEmx.sh
source runModelgen.sh
```

- Run script runEmx.sh to simulate all generated gds files for the provided component using Cadence EMX EM solver
- Run script runModelgen.sh to create a scalable model for the required component in the given geometry and frequency range using Cadence Modelgen tool

###  **Result:**

- Result is parameterized spice model (plIndSymRect.scs)

## **2. Sweep geometry**

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
