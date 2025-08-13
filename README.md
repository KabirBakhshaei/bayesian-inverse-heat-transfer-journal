# Bayesian Inverse Heat Transfer Paper (Data Assimilation Framework)

This repository provides all the necessary code and data to reproduce the results presented in our paper entitled **Optimized Bayesian Framework for Inverse Heat Transfer Problems Using Reduced Order Method**. It includes Gaussian and Multiquadric RBF-based reconstructions, visualization scripts, and reproducible containerized environments using **Docker** or **Singularity**.

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
├── requirements.txt
└── README.md
```
## Path Mapping: Host vs. Container

| Location | Example Path | Notes |
|----------|--------------|-------|
| **Host (before mounting)** | `/u/k/kbakhsha/ITHACA-FV-KF/tutorials/UQ/Docker/bayesian-inverse-heat-transfer-journal` | Your cloned repo folder on your workstation |
| **Inside Container (after mounting)** | `/data/paper_repository` | Mount point inside Docker/Singularity — always use this path in container commands |

**Important:** The name of the folder **inside the container** will always be `/data/paper_repository`, regardless of the name it has on your host.  
That’s why in container steps, we `cd /data/paper_repository` instead of using the original folder name.

---

## Project Setup Instructions (Docker or Singularity)

This project uses a **pre-configured container** that includes:

- `OpenFOAM` for CFD simulations  
- `MUQ` for uncertainty quantification  
- `PyTorch`  
→ All are already installed and configured in the container.

---

##  Step-by-Step Instructions

### 1A. Clone this repository and move into the cloned repository
```bash
git clone https://github.com/KabirBakhshaei/bayesian-inverse-heat-transfer-journal
cd bayesian-inverse-heat-transfer-journal
```
### 1B. Save the absolute path of this folder into a shell variable (on the host)
If you are on Linux, WSL, or Git Bash:
```bash
path_files=$(pwd)
echo "$path_files"
```
If you are on Windows PowerShell:
```bash
$path_files = (Get-Location).Path
Write-Output $path_files
```

## If You're Using Docker
### 2. Pull (download) the pre-built Docker image
(Optional) Remove any old container and image with the same name
```
docker rm -f ithacafv            # Remove the container
docker rmi ithacafv/ithacafv     # Remove the image
```
The following code was copied from this [link](https://hub.docker.com/r/ithacafv/ithacafv).

```
docker pull ithacafv/ithacafv
```
### 3. Start a Docker container and mount your repo folder
Linux / WSL / Git Bash version
```
docker run -it --name ithacafv -v "$path_files":/data/paper_repository \ithacafv/ithacafv bash
```
Windows PowerShell version

**Note**: Some Windows setups may not support ```bash``` inside the image. If ```bash``` fails, use ```/bin/sh``` as shown below.
```
docker run -it --name ithacafv -v ${path_files}:/data/paper_repository --entrypoint /bin/sh ithacafv/ithacafv
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
### 4.  Clone ITHACA-FV inside the container (only if not pre-installed) 
```
git clone --depth 1 https://github.com/ITHACA-FV/ITHACA-FV
```
### 5A. Modified_ITHACA_Files
This folder contains modified .C and .H files that replace the originals in ITHACA-FV/... to reproduce the results in the paper.
    ```bash
    
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
If you are in bash:
```
cd ITHACA-FV
source /usr/lib/openfoam/openfoam2506/etc/bashrc       # Load OpenFOAM environment 
source /data/paper_repository/ITHACA-FV/etc/bashrc     # Load ITHACA-FV environment
```
If you are in /bin/sh:
```         
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
Depending on whether your container shell is ```bash``` or ```/bin/sh``, load the required environments:
If you are in bash:
```
source /usr/lib/openfoam/openfoam2506/etc/bashrc       # Load OpenFOAM environment using linux
source /usr/dir/ITHACA-FV/etc/bashrc                   # Load ITHACA-FV environment
```
If you are in /bin/sh:
```         
. /usr/lib/openfoam/openfoam2506/etc/bashrc            # Load OpenFOAM environment
. /usr/dir/ITHACA-FV/etc/bashrc                        # Load ITHACA-FV environment
```
Then compile and run the simulation:
```
# module load muq                                      # Load MUQ module, if it not already pre-installed and linked inside the Docker/Singularity image
wclean                                                 # Clean old builds
wmake                                                  # Compile the solver
blockMesh                                              # Mesh generation
06enKFwDF_3dIHTP                                       # Run the solver 
```
### Running the Gaussian RBF version:
To use the **Gaussian RBF** instead of the multiquadric version:
**Navigate to the Gaussian RBF directory:**
```
cd /data/paper_repository/Data_Assimilation_Gaussian_RBF/Files/
```
**Edit the file**
```
/data/paper_repository/ITHACA-FV/src/ITHACA_FOMPROBLEMS/sequentialIHTP/sequentialIHTP.C
```
inside the file, commant out the line marked ```heatFluxSpaceBasis[funcI][faceI] = Foam::sqrt(1 + (shapeParameter * radius) * (shapeParameter * radius));``` and uncommand the line marked with```heatFluxSpaceBasis[funcI][faceI] = Foam::exp(-1.0 * (shapeParameter * shapeParameter) * (radius * radius));```to switch the configuration to Gaussian RBF mode. 
This change toggles the reconstruction kernel used by the solver from multiquadric to Gaussian. 

**Recompile ITHACA-FV**
```
cd /data/paper_repository/ITHACA-FV
source /usr/lib/openfoam/openfoam2506/etc/bashrc
source etc/bashrc
./Allwmake -m -j 4
```
**Run the solver as done in the multiquadric case:**
```
wclean
wmake
blockMesh
./06enKFwDF_3dIHTP
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

## Postprocessing Guide

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

## requirements.txt

The `requirements.txt` file specifies:

- Python version
- MATLAB version
- Required libraries
---

## Zenodo Archive

This repository is archived and citable via Zenodo.  
Click the badge below to access the DOI and download the versioned release:
In Progress..............

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.XXXXXXX.svg)](https://doi.org/10.5281/zenodo.XXXXXXX)



