# Bayesian Inverse Heat Transfer Paper (Data Assimilation Framework)

This repository provides all the necessary code and data to reproduce the results presented in our paper entitled **Optimized Bayesian Framework for Inverse Heat Transfer Problems Using Reduced Order Method** [arXiv](https://arxiv.org/abs/2402.19381). It includes Gaussian and Multiquadric RBF-based reconstructions, visualization scripts, and reproducible containerized environments using **Docker** or **Singularity**.

---
## Repository Structure
```
.
├── Data_Assimilation_Gaussian_RBF/
│   ├── Files/
│   └── Results/
├── Data_Assimilation_Multiquadric_RBF/
│   ├── Files/
│   └── Results/
├── Modified_ITHACA_Files
│   ├── C Files
│   └── H Files
├── SupplementaryImages/
│   ├── Files/
│   └── Results/
├── environment.yml
└── README.md
```
## Path Mapping: Host vs. Container

| Location | Example Path | Notes |
|----------|--------------|-------|
| **Host (before mounting)** | `.../Docker/bayesian-inverse-heat-transfer-journal` | Your cloned repo folder on your workstation |
| **Inside Container (after mounting)** | `/data/paper_repository` | Mount point inside Docker/Singularity — always use this path in container commands |

**Important:** The name of the folder **inside the container** will always be `/data/paper_repository`, regardless of the name it has on your host.  
<!--
That’s why in container steps, we `cd /data/paper_repository` instead of using the original folder name.
-->
---

## Project Setup Instructions (Docker or Singularity)

This project uses a **pre-configured container** that includes:

- `OpenFOAM` for CFD simulations  
- `MUQ` for uncertainty quantification  
-  ITHACA-FV
-  All are already installed and configured in the container.
---

##  Step-by-Step Instructions

### 1A. Clone this repository and move into the cloned repository
```bash
git clone https://github.com/KabirBakhshaei/bayesian-inverse-heat-transfer-journal
cd bayesian-inverse-heat-transfer-journal
```
### 1B. Save the absolute path of this folder into a shell variable (on the host)
If you are on Windows PowerShell:
```bash
$path_files = (Get-Location).Path
Write-Output $path_files
```
If you are on Linux, WSL, or Git Bash:
```bash
path_files=$(pwd)
echo "$path_files"
```
## If You're Using Docker
### 2. Pull (download) the pre-built Docker image
(Optional) Remove any old container and image with the same name
```       
docker rm -f ithacafv                # Remove the container   
docker rmi -f ithacafv/ithacafv      # Remove the image
```

**Note: Docker image version (pinned for reproducibility):**

``` 
ithacafv/ithacafv@sha256:2f0046edfe653710a8a560de81d4fa906d0400f0784451f39f5825e3ef61c51e
``` 

Using this digest ensures that the software environment is identical and prevents
incompatibilities caused by future updates to the `latest` tag.

The following code was copied from this [Official Docker Hub page](https://hub.docker.com/r/ithacafv/ithacafv).
```
# Reproducible image (fixed SHA256 digest used for the paper)
docker pull ithacafv/ithacafv@sha256:2f0046edfe653710a8a560de81d4fa906d0400f0784451f39f5825e3ef61c51e

```
### 3. Start a Docker container and mount your repo folder
Windows PowerShell version

Note: Some Windows setups may not support ```bash``` inside the image. If ```bash``` fails, use ```/bin/sh``` as shown below.
```
docker run -it --name ithacafv -v "${path_files}:/data/paper_repository" --entrypoint /bin/sh ithacafv/ithacafv@sha256:2f0046edfe653710a8a560de81d4fa906d0400f0784451f39f5825e3ef61c51e


```
Linux / WSL / Git Bash version
```
docker run -it --name ithacafv -v "$path_files":/data/paper_repository ithacafv/ithacafv@sha256:2f0046edfe653710a8a560de81d4fa906d0400f0784451f39f5825e3ef61c51e
```
<!--
**Note**: Inside Docker, your repo will appear at ```/data/paper_repository``` regardless of the name it has on your host.
## If You're Using Singularity (e.g., SISSA Workstations)
### 2B. Load Singularity (on the host)
```
module load singularity
```
### 3B. Pull the Docker image as a Singularity .sif file
```
singularity build ithacafv.sif docker://ithacafv/ithacafv
```
### 4B. Start the Singularity container and mount your repo
```
singularity shell --bind $path_files:/data/paper_repository ithacafv.sif
```
**Important**: Inside Singularity, your repo will appear as ```/data/paper_repository``` — not as ```bayesian-inverse-heat-transfer-journal```.
-->
## Inside the Container (Linux Environment)
Once inside the container (whether using ```bash``` or ```/bin/sh```):

<!--**Note:** Depending on your Singularity configuration, you may already start inside `/data/paper_repository` after running 
`singularity shell --bind ...`. In that case, the `cd /data/paper_repository` command is optional.-->
```
cd /data/paper_repository
```
### 4.  Clone ITHACA-FV inside the container
The container already includes a precompiled ITHACA-FV in `/usr/dir/ITHACA-FV` it can be checked via `find / -type d -name "ITHACA-FV" 2>/dev/null`.
But it is better to clone and compile the own version (for development or persistence) to the mounted `/data/paper_repository` directory. 

It is fine to have two copies of ITHACA-FV: the precompiled(read only) which is `/usr/lib/ITHACA-FV` is part of the container’s filesystem, changes here will be lost when the container is removed or the editable clone which is `/data/ITHACA-FV` is bind-mounted to the host machine, changes here will persist between container runs.
**ITHACA-FV version (pinned for reproducibility):** we use the exact commit tested for the manuscript results.
```
rm -rf /data/paper_repository/ITHACA-FV
git clone https://github.com/ITHACA-FV/ITHACA-FV
cd ITHACA-FV

# Pinned ITHACA-FV commit (exact version used for the paper, dated 2025-08-08)
git fetch --all --tags
git checkout "$(git rev-list -n 1 --before="2025-08-08 23:59" origin/HEAD)"

# Sync and initialize submodules at the same historical snapshot
git submodule sync --recursive
git submodule update --init --recursive

# Sanity check (optional but recommended)
git rev-parse HEAD
git submodule status --recursive | head -n 10
```

### 5A. Modified_ITHACA_Files
This folder contains modified '.C' and '.H' source files that must replace the corresponding originals in the 'ITHACA-FV' source tree in order to reproduce the results presented in the paper.

    ```
    # ITHACA_MUQ
    cp /data/paper_repository/Modified_ITHACA_Files/ensembleClass.C /data/paper_repository/ITHACA-FV/src/ITHACA_MUQ/ensembleClass.C
    cp /data/paper_repository/Modified_ITHACA_Files/ensembleClass.H /data/paper_repository/ITHACA-FV/src/ITHACA_MUQ/ensembleClass.H
    cp /data/paper_repository/Modified_ITHACA_Files/muq2ithaca.C /data/paper_repository/ITHACA-FV/src/ITHACA_MUQ/muq2ithaca.C
    cp /data/paper_repository/Modified_ITHACA_Files/muq2ithaca.H /data/paper_repository/ITHACA-FV/src/ITHACA_MUQ/muq2ithaca.H

    # ITHACA_MUQ/Fang2017filter_wDF
    cp /data/paper_repository/Modified_ITHACA_Files/Fang2017filter_wDF.C /data/paper_repository/ITHACA-FV/src/ITHACA_MUQ/Fang2017filter_wDF/Fang2017filter_wDF.C
    cp /data/paper_repository/Modified_ITHACA_Files/Fang2017filter_wDF.H /data/paper_repository/ITHACA-FV/src/ITHACA_MUQ/Fang2017filter_wDF/Fang2017filter_wDF.H


    # ITHACA_FOMPROBLEMS/sequentialIHTP
    cp /data/paper_repository/Modified_ITHACA_Files/sequentialIHTP.C /data/paper_repository/ITHACA-FV/src/ITHACA_FOMPROBLEMS/sequentialIHTP/sequentialIHTP.C
    cp /data/paper_repository/Modified_ITHACA_Files/sequentialIHTP.H /data/paper_repository/ITHACA-FV/src/ITHACA_FOMPROBLEMS/sequentialIHTP/sequentialIHTP.H
    cp /data/paper_repository/Modified_ITHACA_Files/createThermocouples.H /data/paper_repository/ITHACA-FV/src/ITHACA_FOMPROBLEMS/sequentialIHTP/createThermocouples.H
    ```
    
### 5B.Navigate and Compile ITHACA-FV with MUQ support
(Works in both `bash` and `/bin/sh`)
```
cd ITHACA-FV        
. /usr/lib/openfoam/openfoam2506/etc/bashrc       # Load OpenFOAM environment   
. /data/paper_repository/ITHACA-FV/etc/bashrc     # Load ITHACA-FV environment
```
then
```
git submodule update --init                            # Fetch dependencies
./Allwmake -tauq -j 4                                  # Compiles everything including Tauq (for UQ)
```
### 6. Navigate back to the mounted repo
```
cd /data/paper_repository
```

### 7. Navigate to the Simulation Directory and Run the Solver 
```
cd /data/paper_repository/Data_Assimilation_Multiquadric_RBF/Files/
```
Load the required environments (Works in both `bash` and `/bin/sh`):
```         
. /usr/lib/openfoam/openfoam2506/etc/bashrc            # Load OpenFOAM environment
. /data/paper_repository/ITHACA-FV/etc/bashrc          # Load ITHACA-FV environment

# Set required env vars
export LIB_SRC="$WM_PROJECT_DIR/src"
export ITHACA_DIR=/data/paper_repository/ITHACA-FV
export LIB_ITHACA_SRC="$ITHACA_DIR/src"
export LIB_ITHACA_LIB="$FOAM_USER_LIBBIN"   # ITHACA libs are here on v2506
export MUQ_LIBRARIES=/root/miniconda3
export MUQ_EXT_LIBRARIES=/root/miniconda3
export LD_LIBRARY_PATH="$LIB_ITHACA_LIB:$MUQ_LIBRARIES/lib:$LD_LIBRARY_PATH"

# Quick pre-flight checks (optional but helpful)
echo WM_PROJECT_DIR=$WM_PROJECT_DIR; echo LIB_SRC=$LIB_SRC; echo LIB_ITHACA_SRC=$LIB_ITHACA_SRC; echo LIB_ITHACA_LIB=$LIB_ITHACA_LIB
for d in "$LIB_SRC/finiteVolume/lnInclude" "$LIB_ITHACA_SRC/ITHACA_MUQ" "$MUQ_LIBRARIES/include"; do [ -d "$d" ] && echo OK:$d || echo MISSING:$d; done
ls -1 "$FOAM_USER_LIBBIN"/libITHACA_*.so
cd /data/paper_repository/Data_Assimilation_Multiquadric_RBF/Files/
```

Then compile and run the simulation:
```
# module load muq                                      # Load MUQ module, if it not already pre-installed and linked inside the Docker/Singularity image
wclean                                                 # Clean old builds
wmake -j4                                              # Compile the solver
blockMesh                                              # Mesh generation
06enKFwDF_3dIHTP                                       # Run the solver 
```
### Running the Gaussian RBF version:
To use the **Gaussian RBF** instead of the multiquadric version:

Navigate to the Gaussian RBF directory:
```
cd /data/paper_repository/Data_Assimilation_Gaussian_RBF/Files/
```
Edit the `sequentialIHTP.C file` located in this directory `/data/paper_repository/ITHACA-FV/src/ITHACA_FOMPROBLEMS/sequentialIHTP/sequentialIHTP.C`. Go to the source and back it up
```
cd /data/paper_repository/ITHACA-FV/src/ITHACA_FOMPROBLEMS/sequentialIHTP; cp sequentialIHTP.C sequentialIHTP.C.mq.bak
```
Toggle to Gaussian RBF

Comment the multiquadric line (the `sqrt(...)` one) and uncomment the Gaussian line (`exp(...)`):
inside the file, commant out the line marked ```heatFluxSpaceBasis[funcI][faceI] = Foam::sqrt(1 + (shapeParameter * radius) * (shapeParameter * radius));``` and uncommand the line marked with```heatFluxSpaceBasis[funcI][faceI] = Foam::exp(-1.0 * (shapeParameter * shapeParameter) * (radius * radius));```to switch the configuration to Gaussian RBF mode. 

This change toggles the reconstruction kernel used by the solver from multiquadric to Gaussian. 
```
sed -i -E '/heatFluxSpaceBasis\[.*\].*=.*Foam::sqrt\(/ s@^([[:space:]]*)@&// @' sequentialIHTP.C; sed -i -E '/heatFluxSpaceBasis\[.*\].*=.*Foam::exp\(/ s@^([[:space:]]*)//[[:space:]]*@\1@' sequentialIHTP.C
```
Verification, optional
```
grep -nE 'heatFluxSpaceBasis.*(Foam::sqrt|Foam::exp)' sequentialIHTP.C
```
You should see the `sqrt` line commented and the `exp` line active.

Rebuild the ITHACA library that uses this file
```
cd /data/paper_repository/ITHACA-FV/src/ITHACA_FOMPROBLEMS; wclean libso; wmake libso -j4
```
If you see a link error, make sure your env is loaded and paths are set (same as before):
```
. /usr/lib/openfoam/openfoam2506/etc/bashrc; . /data/paper_repository/ITHACA-FV/etc/bashrc; export LIB_ITHACA_LIB="$FOAM_USER_LIBBIN"; export LD_LIBRARY_PATH="$LIB_ITHACA_LIB:/root/miniconda3/lib:$LD_LIBRARY_PATH"
```
Then re-run the `wmake libso`.
Rebuild the solver
```
cd /data/paper_repository/Data_Assimilation_Gaussian_RBF/Files; wclean; wmake -j4
```
Run the Gaussian case
```
blockMesh
06enKFwDF_3dIHTP
```

### 8. Generated Output Files and Folders After Simulation
**Folders:**
```
ITHACAoutput/
```
**Files:**
```
B.npy
Btemp.npy
condNumber.txt
condNumberAutoCovInverse.txt
condNumberCrossCov.txt
condNumberKalmanGain.txt
gTrueMatrix.npy
measNoiseCovTotal.txt
measurementsMat.npy
measurementsMatNoise.npy
parameterPriorCov.npy
parameterPriorMean.npy
parameterPriorMeanWithoutShifting.npy
radius_kb.npy
replay_pid2156.log
Temp.npy
Temp2.npy
thermocouplesCellsID_mat.txt
thermocoupleXValues.npy
thermocoupleYValues.npy
thermocoupleZValues.npy
xyz.npy
```

Once data is generated, postprocessing should be done outside Docker/Singularity using Python, and MATLAB.

### 9. Reproducibility Notes

We tested the container on two different systems and obtained consistent results within a small margin of variation:
The table below compares the mean relative error values reported in the paper with those reproduced on two different systems using Docker.  
| Method        | Paper Error (mean relative) | System 1 (Docker) | System 2 (Docker) |
|---------------|-----------------------------|-------------------|-------------------|
| Gaussian RBF  | 7.59                        | 7.61              | 7.88              |
| Multiquadric  | 6.31                        | 6.32              | 6.59              |

**Notes:**
- Small variations between systems are expected due to nondeterministic operations in OpenFOAM/MUQ, CPU architecture diffrences, floating-point rounding, and different BLAS/LAPACK libraries used by Docker runtimes.  
- Results are consistent and validate the reproducibility of the framework.
---
---
## Hyperparameter Sensitivity Studies (Figures 5--10)

Figures 5--10 in the paper are obtained from **parametric sensitivity studies** of the EnSISF-wDF framework.  
Each data point shown in these figures corresponds to an **independent execution of the full data assimilation algorithm**, not to a postprocessing step.

For each study, one hyperparameter is varied while all others are kept fixed, and the resulting **spatiotemporal relative error** is computed.

### Mapping between figures and hyperparameters

| Figure | Varied hyperparameter |
|------|----------------------|
| Fig. 5 | Ensemble size |
| Fig. 6 | RBF shape parameter $\eta$ |
| Fig. 7 | Prior mean shifting | 
| Fig. 8 | Prior covariance scaling $\kappa$ |
| Fig. 9 | Time step $\Delta t$ | 
| Fig. 10 | Observation span |

### How these figures are generated

For each value of the selected hyperparameter:
1. The full EnSISF-wDF solver is compiled (if required),
2. The simulation is executed,
3. The spatiotemporal relative error is computed from the generated output,
4. The error values are collected and plotted.

Because these figures rely on **multiple full simulations**, they are not generated by a single postprocessing script.

## Postprocessing Guide
Note: This section covers only figures generated from a single simulation run (Figures 11--14 and supplementary visualizations). Figures 5--10 are generated via multiple full simulation runs as described in the Hyperparameter Sensitivity Studies section above.
### Inside `Data_Assimilation_Multiquadric_RBF/Files/`

#### `plots.ipynb`

- **Reads the following as inputs:**
  - From `./ITHACAoutput/true/`:  
    - `trueTimeVec_mat.txt`  
    - `probe_true_mat.txt`
  - From `./ITHACAoutput/reconstruction/`:  
    - `probe_rec_mat.txt`  
    - `probeState_minConf_mat.txt`  
    - `probeState_maxConf_mat.txt`  
    - `gTrue_probe_mat.txt`  
    - `gRec_probe_mat.txt`  
    - `gRec_probeMaxConf_mat.txt`  
    - `gRec_probeMinConf_mat.txt`  
    - `parameterMean_mat.txt`  
    - `parameter_minConf_mat.txt`  
    - `parameter_maxConf_mat.txt`
  - From `./ITHACAoutput/projection/HeatFluxSpaceRBF/`:  
    - `heat_flux_space_basis_mat.txt`
  - From `./ITHACAoutput/projection/TrueHeatFlux/`:  
    - `HeatFluxTrue_mat.txt`
  - From current directory:  
    - `parameterPriorMean.npy`  
    - `parameterPriorMeanWithoutShifting.npy`  
    - `condNumberAutoCovInverse.txt`  
    - `condNumberCrossCov.txt`  
    - `condNumberKalmanGain.txt`  
    - `xyz.npy`  
    - `Temp.npy`

- **Outputs (to `../Results/`):**
  - Figures:  
    - `Figure 11B TrueAndReconstructedMeanTemperatureAtaProbe_0.91_0.02_0.55_OverTimeMultiquadric.png`  
    - `Figure 12B TrueAndReconstructedMeanHeatFluxAtProbe_0.91_0.0_0.55_over_time.png`  
    - `Figure 13B TrueAndReconstructedMeanHeatFluxAtTheHotSideWithConfidenceInterval.png`
  - Text files:  
    - `meanOfMeanRelativeError.txt`  
    - `meanRelativeErrorAtT0.txt`  
    - `meanRelativeErrorAtT0WithoutShifting.txt`  
    - `autoCovInverse_mean_std.txt`  
    - `crossCov_mean_std.txt`  
    - `kalmanGain_mean_std.txt`  
    - `xyz.txt`  
    - `Temp.txt`

#### `Surface3DAnimation.m`

- **Creates (to `../Results/`):**
  - `3D Combined surface plot.avi`
  - `3D Combined surface plot.gif`

#### `ContourFigurePaperMultiquadricRelative.m`

- **Creates (to `../Results/`):**
  - `Figure 14b SnapshotCountorsMultiquadric.png`  
  - `Figure 14C SnapshotCountorsTrue.png`  
  - `Figure 14e SnapshotCountorsMultiquadricRelative.png`

#### `RBFsThermocouples.m`

- **Creates (to `../Results/`):**
  - `Figure 2 Thermocouples_RBF_Centers.png`

---

### Inside `Data_Assimilation_Gaussian_RBF/Files/`

- Run:
  - `plots.ipynb`
  - `Surface3DAnimation.m`
  - `ContourFigurePaperGaussianRelative.m`

- Produces:
  - Figures analogous to the Multiquadric RBF setup:  
    `Figure 11A, 12A, 13A`–`Figure 14a, 14C, 14D`
  - `3D Combined surface plot.avi`
  - `3D Combined surface plot.gif`

---

### Inside `SupplementaryImages/Files/`

#### `RBFsComparison.m`

- **Reads (from local folder):**
  - `xyz.txt`
  - `TempG0_5.txt`, `TempG1.txt`, `TempG2.txt`, `TempG2_5.txt`
  - `TempM0_5.txt`, `TempM1.txt`, `TempM3.txt`, `TempM7_5.txt`

- **Creates (in `../Results/`):**
  - **Individual Surface Plots:**
    - `Figure 4a1`–`Figure 4a4` (Gaussian)  
    - `Figure 4b1`–`Figure 4b4` (Multiquadric)
  - **Combined Images:**
    - `Figure 4a Combined1.png` / `.pdf`  
    - `Figure 4b Combined2.png` / `.pdf`
  - **Face Centers:**
    - `Figure 4c XZCentersOfFaces.pdf`

#### `CombinedFigures14.ipynb`

- **Reads (from local folder):**
  - `Figure14aSnapshotCountorsGaussian.png`, `Figure14bSnapshotCountorsMultiquadric`, `Figure14CSnapshotCountorsTrue.png`
  - `Figure14dSnapshotCountorsGaussianRelative.png`, `Figure14eSnapshotCountorsMultiquadricRelative.png`

- **Creates (in `../Results/`):**
  - **Combined Images:**
    - `Figure14_combined.png`
---

## Postprocessing environment (Python/MATLAB)

Postprocessing (figure generation and notebooks) is performed **outside** Docker/Singularity using Python and MATLAB.

### Python (recommended: Conda)
A portable Conda environment specification is provided in `environment.yml` (pinned package versions).

Create and activate it with:
```
conda env create -f environment.yml
conda activate postprocess-ihtp
```

### Software versions used for postprocessing

- **Python:** 3.10.19
- **MATLAB:** R2024b

## Zenodo Archive (Permanent Reference)

To ensure long-term accessibility and full reproducibility of the results presented in the manuscript, a **frozen and citable release** of this repository has been archived on Zenodo.

The archived version corresponding **exactly** to the results reported in the paper is available at:

[![DOI](https://zenodo.org/badge/1034378347.svg)](https://doi.org/10.5281/zenodo.16925045)

This Zenodo record represents the **authoritative reference for reproducibility** and
corresponds to a snapshot of the codebase (including scripts, configuration files, and
postprocessing tools) used for the final manuscript. It should be used for citation and
long-term reference, independent of future changes to the GitHub repository.

---

## How to Cite

If you use this code or reproduce results from this work, please cite the Zenodo archived release:

Kabir Bakhshaei, Umberto Emil Morelli, Giovanni Stabile, Gianluigi Rozza. (2025).  
*Optimized Bayesian Framework for Inverse Heat Transfer Problems Using Reduced Order Method* (Version 1.0.0).  
Zenodo. https://doi.org/10.5281/zenodo.16925045

