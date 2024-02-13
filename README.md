# ORCA Analysis GUI

## Introduction
This repository contains a Graphical User Interface (GUI) designed to streamline the creation and analysis of files generated by the [ORCA computational chemistry package](https://orcaforum.kofo.mpg.de/app.php/portal). Launch the GUI by running [Launcher.py](/GUI/Launcher.py) in your preferred Python environment. 

## Prerequisites
While the [Launcher](/GUI/Launcher.py) will prompt the installation of any missing packages and automatically add them to your system's PATH, it's advisable to pre-install the required software and Python modules before using the GUI.

### External Software
Ensure the following software is installed on your system:

- **Python 3.11+**: Available from [Python Downloads](https://www.python.org/downloads/). During installation, add Python to your system's PATH and choose a convenient installation location, such as `C:\Python311`.
- **GitHub Desktop**: Available from [GitHub Desktop](https://desktop.github.com/). Installation does not require a GitHub account.
- **Git**: Available from [Git SCM](https://git-scm.com/). Install using the default settings in the suggested location, unless you have specific preferences. No GitHub account is required.

### Python modules
Install the necessary Python modules using `pip` in your command prompt:

```console
pip install importlib numpy scipy matplotlib git lxml PyQt6 pyarrow pandas openpyxl csv natsort
```

# GUI Usage
The GUI features 13 modules designed to facilitate the analysis of ORCA output files across various common and advanced applications, organized by their functionality.

## T1. CREST .xyz Splitter:
This module extracts conformers and their energies identified by the [Conformer–Rotamer Ensemble Sampling Tool (CREST)](https://crest-lab.github.io/crest-docs/), an exceptionally efficient program for mapping low-energy conformations of any analyte. *XYZ* coordinates of each conformer are stored in the `crest_conformers.xyz` file, which is generated upon completion of a CREST run. The `CREST .xyz splitter` module extracts each conformer to individual `.gjf`, `.xyz`, or `.inp` files . Extracting of conformers to alternative file types will be added in future updates. 

- **Inputs:**
    - **.xyz file**: Specify the path and filename of `crest_conformers.xyz`. 
    - **Output basename**: Define the basename for exported conformer files. Each file will be suffixed with _N, where N is an integer, to ensure unique identification.
    - **Export file type**: Choose whether to extract conformers from `crest_conformers.xyz` to `.gjf`, `.xyz`, or `.inp` files. 
- **Outputs:**
    - Writes all conformers extracted from `crest_conformers.xyz` to individual .gjf files in the directory `Conformers`. 
    - Writes `Energies.csv`, a file summarizing the name, energy, and relative energy sorted in ascending order for direct use with `T2. Cosine similarity sorting`.

### **Usage of T1 and [example files](https://github.com/HopkinsLaboratory/ORCA_Analysis_GUI/tree/master/Sample_Files/T1_CREST_xyz_splitter)**: 
- **Select .xyz File**: Click the "Browse" button to open a file explorer window. Navigate to and select the .xyz file you wish to split, or paste the file path into the dialog box. 
- **Specify Output Basename**: Enter a basename for the output files in the designated input box.
- **Execute Splitting**: Click the `Run` button to start the splitting process. The script will write output files with named determined by the `output basename` to the `Conformers` directory.
- **Integrated error handling**: Any errors or issues encountered will be reported to the `Status window`, prompting the user to adjust their input accordingly.

## T2. Cosine similarity sorting:
This module processes multiple .gjf, .inp, and/or .xyz files to identify unique conformations using the cosine similarity method detailed [here](https://www.frontiersin.org/articles/10.3389/fchem.2019.00519/full). It is specifically designed to integrate with outputs from the [CREST .xyz Splitter](#1-crest-xyz-splitter), including the `Energies.csv` file.

- **Inputs:**
    - **Directory or .csv**: 
        - **Directory input**: Provide the path to a directory containing at least two files with the extensions `.gjf`, `.xyz`, or `.inp` files. These files need not originate from the CREST .xyz Splitter, allowing for flexibility in applications like conducting a pairwise similarity comparison further into your workflow.
        - **.csv input**: While the `Energies.csv` file generated by the CREST .xyz Splitter is already in the correct format, users may also supply their own .csv file, provided it adheres to the specified format: 
            - The .csv's first row must include these headers: `Filename` `Energy / hartree` and `Relative Energy / kJ mol**-1`. 
            - **Filename column**: Entries here must list the filename with its extension. The files mentioned must be located in the same directory as the Energies.csv file.
            - **Energy columns**:  Entries must be numeric and arranged in ascending energy order. An error will be displayed if the files are not sorted correctly or if the entries are not numeric.
            - The .csv file must include *at least* two rows beyond the header. Fewer entries imply that only a single file is present, precluding the possibility of any pairwise comparison.

    - **Similiarity threshold(%)**: Specify the similiarity threshold that determines whether one geometry is unique from another. The similiarity between two geometries is defined by the cosine of the angle between two 1D-vectors $(\vec{V}_a, \vec{V}_b)$, where each vector is a 1D-array of mass weighted distance ($d_{CoM}$) of each atom from the molecule's centre of mass ($CoM$):
    
        $d_{CoM} = m_{atom} \times \sqrt{(x_{atom} - x_{CoM})^2 + (y_{atom} - y_{CoM})^2 + (z_{atom} - z_{CoM})^2}$
    
        The 1D array is sorted from smallest to largest to such that variations in atom ordering from the input file and rotational equivalences (*e.g.,* the C2 symmetry of the phenyl moiety) do not result in unqiue geometries. The cosine similiarity between $(\vec{V}_a, \vec{V}_b)$ is then evaluated by:
    
        $sim(\vec{V}_a, \vec{V}_b) = \left(1 - \left(\frac{1}{\pi} \right) \times \arccos\left(\frac{\vec{V}_a \cdot \vec{V}_b}{\|\vec{V}_a\| \times \|\vec{V}_b\|}\right)\right) \times 100$

    - **Write pairwise similiarities to a .csv file**: If checked, $sim(\vec{V}_a, \vec{V}_b)$ of each pairwise comparison will be written to `pairwise_similiarities.csv`.

- **Outputs:**
    Files exhibitng a pairwise $sim(\vec{V}_a, \vec{V}_b)$ below the specified threshold will be copied to a newly created directory called `uniques_sim_N`, where N is the similiarity threshold. If the `Write pairwise similiarities to a .csv file` option was checked, `pairwise_similiarities.csv` will be written to the `uniques_sim_N` directory. 

### **Usage of T2 and [example files](https://github.com/HopkinsLaboratory/ORCA_Analysis_GUI/tree/master/Sample_Files/T2_Cosine_sim)**: 

1. **Directory or .csv input:**
- Click the `Browse dir` button to select a directory containing your .gjf, .xyz, and/or .inp files, or paste the directory path into the dialog box.
- Alternatively, use the `Browse .csv` button to select a .csv file, or paste the directory path into the dialog box. Ensure your .csv file is formatted correctly, with headers that are `Filename`, `Energy / hartree`, and `Relative Energy / kJ mol**-1`.

2. **Setting the similiarity threshold:**
- Adjust the `Cosine similarity threshold (*100):` field to set your desired threshold for similarity comparisons. Thresholds can be defined anywhere between 0.00 to 99.99.

3. **Writing Pairwise Similarities:**
- Check the `Write pairwise similarities to a .csv file?` option if you wish to save the results of the pairwise similarity comparisons to a .csv file.

4. **Running the Analysis:**
- Once you have configured your inputs and options, click the `Sort files by cosine similarity` button to initiate the analysis.

**Integrated error handling**: Any errors or issues encountered will be reported to the `Status window`, prompting the user to adjust their input accordingly.

## T3. Generate ORCA .inp: 
This module processes a directory containing .gjf, .inp, and/or .xyz files and converts each into a new ORCA .inp file with its content controlled through the GUI interface. 

**Inputs:**
- **Directory**: Users must provide a directory path that contains at least one .gjf, .xyz, or .inp file. This directory can be selected directly through the GUI by clicking the Browse button.
- **Performance Settings**: Specify the memory per core (MB), number of cores, charge, and multiplicity for the ORCA calculations. These settings allow for fine-tuning the computational resources allocated for each job.
- **Calculation Method**: Choose between predefined methods (DFT, CCSDT) or a custom method. The GUI automatically populates the method line based on the selection.
- **ESP Charges**: Optionally compute ESP charges by selecting the `Compute ESP Charges?` checkbox. This action enables additional input fields for grid and Rmax (both in Angstroms). Reccomended values for $grid$ and $R_{max}$ are 0.1 Å and 3.0 Å, respectively. With these settings, the CHELPG partition scheme uses a grid composed of points spaced apart by 0.1 Å, where each $grid$ point is at most 3.0 Å away from any atom in the molecule. For larger molecules (> 350 electrons without pseudopotentials), using a coarser grid with the parameters $grid = 0.3 Å$ and $R_{max} = 3.0 Å$ is reccomended.
- **Additional Options**: Users can opt to calculate the Hessian on the first optimization step, compute dipole/quadrupole moments, or call XYZ coordinates from an external .xyz file by selecting the respective boxes.

**Outputs:**
For each input file within the specified directory, the module generates a new ORCA input file (.inp) with the requested parameters. These files are saved to a new directory called `New_Inputs` or `New_Inputs_xyz` depending on whether the `Call XYZ coordinates from external .xyz file?` option was selected. 

### **Usage and [example files](https://github.com/HopkinsLaboratory/ORCA_Analysis_GUI/tree/master/Sample_Files/T3_gjf_to_ORCA_inp)**: 

1. **Select directory**: Use the `Browse` button to choose the directory containing your molecular geometry files (.gjf, .xyz, or .inp), or paste the file path directly into the dialog box.

2. **Configure the performance settings**: Adjust the memory per core (in MB), number of cores, charge, and multiplicity as required for your calculations.

3. **Choose the calculation method**: Select a predefined method or customize your own. The method line can be edited in the Custom mode for specific requirements.

4. **Compute ESP charges (optional)**: If desired, enable ESP charge computation and specify the $grid$ and $R_{max}$ values.

5. **Additional options**: Decide whether to calculate the Hessian, compute dipole/quadrupole moments, or call XYZ coordinates from an external .xyz file.

6. **Generate .inp files**: Click the Run button to create the ORCA .inp files in the specified directory.

**Integrated error handling**: Any errors or issues encountered will be reported to the `Status window`, prompting the user to adjust their input accordingly.

## **T4. ORCA .out to ORCA .inp (Opt/Freq)**: 
This module processes a directory containing .out files from ORCA calculations, extracting the final geometries to generate new ORCA .inp files for optimization/frequency jobs. The configuration of these .inp files, including their content and options, is managed through the GUI interface.

**Inputs, outputs, usage, and [example files](https://github.com/HopkinsLaboratory/ORCA_Analysis_GUI/tree/master/Sample_Files/T4_ORCA_out_to_ORCA_inp):**
The inputs and outputs for this module align with those described in [T3. Generate ORCA .inp](#t3-generate-orca-inp), with one additional feature:
- **Additional Options**: There's an option to `Write final coordinates to a .gjf file`. Selecting this checkbox creates `.gjf` files alongside `.inp` files, facilitating visualization using external software packages like GaussView.

**Integrated error handling**: Any errors or issues encountered will be reported to the `Status window`, prompting the user to adjust their input accordingly.

## **T5. ORCA .out to ORCA .inp (VGFC/TD-DFT)**
This module processes a directory containing .out files from ORCA calculations, extracting the final geometries to generate new ORCA .inp files for computing UV-Vis absorption spectra via the Vertical Gradient Franck-Condon (VG|FC) approach, which is sometimes called the Independent Mode Displaced Harmonic Oscillator (IMDHO) model. The configuration of these .inp files, including their content and options, is managed through the GUI interface.

### **T6. Plot optimization trajectory**:
Plots the energy change, SCF energy, max gradient, max step, RMS gradient, and RMS step as a function of the number of optimziation cycles for a single ORCA .out file containing the Opt keyword (or any other vairant that performs a geometry optimization).

###  **T7. Calculate thermochemistry**: 
Take a directory containing a series of ORCA .out files and calculates thermochemical corrections and parition functions using the methods of [McQuarrie and Simon (1999)](https://gaussian.com/wp-content/uploads/dl/thermo.pdf) at user specified values of temperature and pressure. Users can also calculate thermochemistry using scaled vibrational frequencies if desired. Morever, users will be notified about the presence of imaginary frequencies and/or imcomplete Opt/Freq jobs. Outputs for this module include the following:
    - Filename
    - Number of imaginary freqs
    - Electronic energy
    - Thermal energy
    - Total enthalpy (H)
    - Total entropy (T * S)
    - Gibbs energy (G) 
    - A, B, and C rotational constants
    - Rotational symmetry number
    - X, Y, and Z dipole components, as well as the total dipole moment
    - Isotropic polarizability
    - Translational, rotational, vibrational, and electronic partition functions 

### **T8. Extract coupled cluster energies**: Takes a directory containing a series of ORCA .out files containing information from coupled cluster calculations and extracts relevant values, including:
    - Filename
    - CCSD(T) energy
    - CCSD energy
    - Triples correction
    - Correlation energy
    - T1 diagnostic (indicates whether the coupled cluster results are trustworthy)

### **T9. Extract/plot IR spectra**: 
Takes a directory containing a series of ORCA .out files that have computed vibrational freqencies via the `Freq` keyword (or similiar). IR spectra are then calculated by applying a Gaussian convolution to each absorption (with a userspecified FWHM), and exported to an Excel file (.xlsx) along a user specified range in cm**-1 and step size. Exporting options include the following:
    - Apply a multiplicative scaling factor to all calculated vibrational frequencies (between 0 - 2)
    - Normalize spectra to a maximum value of 1
    - Create and save an additional plot of all exported spectra.

### **T10. Extract/plot UV-Vis spectra (VG-FC)**: 
Takes a directory containing a series of .spectrum.rootN and/or .spectrum files (can be for multiple analytes!) generated Excited State Dynamics (ESD) simulations performed using ORCA and exports their spectra to an Excel file along a user specified range and energy unit (nm, cm**-1, or eV). Options:
    - **Input unit**: Specify the unit of energy of the data stored in the .spectrum / .spectrumrootN files (nm, cm**-1, or eV).
    - **Output unit**: Specify the unit of energy of that you would like your spectra to be exported as (nm, cm**-1, or eV).
    - **Export range**: Specify the range and step size for your exported spectra; default values applicable to most systems auto-update in the GUI interface. 
    - **Shift spectra**: Apply a uniform redshift/blueshift to the data in the .spectrum.rootN and .spectrum files for alignment with experimental data. Shifts can be specified in nm, cm**-1, or eV (eV reccomended due to non-liear spacing of the nm and cm**-1 scales.)
    - **Output basename**: The name that the output files will be called (no extensions or pathname)
    - **Normalize output**: Normalize spectra to a maximum value of 1
    - **Plot spectra externally**: Create and save additional plots of all exported spectra.     

### **T11. Boltzmann-weight CCSs**: 
Calculates the ensemble-averaged CCS of an molecule by Boltzmann-weighting the CCS of each conformation according to its population. Requires:
    - **DFT Thermochem .csv**: .csv file containing DFT electronic energies and thermochemistry, which is generated from the [**7. Calculate thermochemistry**](#calculate-thermochemistry) tab. 
    - **Coupled cluster .csv**: .csv file containing CCSD(T) or CCSD energies generated from the [8. Extract coupled cluster energies](#extract-coupled-cluster) tab.
    - **CCS .csv**: .csv file containing CCSs, which is generated from the [many .mout analyzer](https://github.com/HopkinsLaboratory/MobCal-MPI) tab of the MobCal-MPI GUI.

### **T12. ORCA LED analysis**: 
Plots the results from a Local Energy Decomposition (LED) analysis, decomposing the interaction energy between the parent molecule and its N fragments. Currently, the code supports two fragments (N = 2). Inputs:
    - 

### **T13. ORCA NEB analysis**: 
Plots the results from a Nudged Elastic Band (NEB) analysis, including the reaction coordinate during eachNEB interation along several user-specified parameters, including reaction coordinate, atomic distances, bond angles, or dihedral angles. Exporting the *XYZ* coordinates images from a specific run and generation of an ORCA .inp from these coordinates is also supported. Implementation will be including in the next major update. Inputs:
- _trj.xyz file from a NEB calculation 