# MUBD-DecoyMaker 3.0: Making Maximal Unbiased Benchmarking Data Sets with Deep Reinforcement Learning

## Introduction

MUBD-DecoyMaker 3.0 is a brand-new computational software to make Maximal Unbiased Benchmarking Data Sets (MUBD) for in silico screening. Compared with our earlier two versions, i.e. [MUBD-DECOYMAKER](https://github.com/jwxia2014/MUBD-DECOYMAKER) (Pipeline Pilot-based version, or MUBD-DecoyMaker 1.0) and [MUBD-DecoyMaker 2.0](https://github.com/jwxia2014/MUBD-DecoyMaker2.0), MUBD-DecoyMaker 3.0 has two noteworthy features:

1. Virtual molecules generated by recurrent neural netwrok (RNN)-based molecular generator with reinforcement learning (RL), instead of chemical library molecules, constitue the unbiased decoy set (UDS) component of MUBD. 

2. The criteria (or rule) for an ideal decoy previously defined in the earlier versions are integrated into a new scoring function for RL to fine-tune the generator.


Below is how to implement and run MUBD-DecoyMaker 3.0.

![Figure from manuscript](figures/model_1.png)

## Requirements

As [REINVENT](https://github.com/MolecularAI/Reinvent) is used to make virtual decoys of MUBD 3.0, users are required to install this tool as instructed. The corresponding `conda` environment named `reinvent.v3.2` is created for virtual decoy generation. Please note we have modified the [PyPI](https://pypi.org) packages `reinvent_chemistry` and `reinvent_scoring` here in order to include our scoring functions specific for MUBD. Another `conda` environment named `MUBD3.0` is also created for preprocessing and postprocessing.

1) Install [REINVENT](https://github.com/MolecularAI/Reinvent).

2) Clone this repository and navigate to it:
```bash
$ git clone https://github.com/Sooooooap/MUBD3.0.git
$ cd MUBD3.0
```

3) Replace the packages `reinvent_chemistry` and `reinvent_scoring` with our modified ones:
```bash
$ conda activate reinvent.v3.2 
$ pip show reinvent_chemistry # Location: ~/anaconda3/envs/reinvent.v3.2/lib/python3.7/site-packages
$ cp -r reinvent_chemistry/ reinvent_scoring/ ~/anaconda3/envs/reinvent.v3.2/lib/python3.7/site-packages
```

4) Create the `conda` environment called `MUBD3.0`:
```bash
$ conda env create -f MUBD3.0.yml
```

## Usage

`ACM Agonists` is used as a test case to demonstrate how to build MUBD 3.0 with MUBD-DecoyMaker 3.0. All the test files are in the directory of `resources`. 

### Build the unbiased ligand set (ULS 3.0)
Run `build_uls.py` to process the raw ligand set. This script takes the raw ligands in SMILES representation as input (`raw_actives.smi`) and puts out the unbiased ligand set (`Diverse_ligands.csv`). Four files regarding ligand properties, i.e. `Diverse_ligands_PS.csv`, `Diverse_ligands_PS_maxmin.csv`, `Diverse_ligands_sims_maxmin.txt` and `Diverse_ligands_len.txt`, are also generated.

IMPORTANT: Ligand curation, including molecule standardization, salt removal and protonization at a specific range of pH (implemented by [Dimorphite-DL](https://github.com/Sulstice/dimorphite_dl)), is required if the ligands are not curated. For ligand curation, we provide the `--cure` option for `build_uls.py`. Please note the raw ligands in this test case are curated. Also, users may use `--help` option to see all the available options.
```bash
$ conda activate MUBD3.0
(MUBD3.0) $ python build_uls.py
```

### Generate the potential decoy set

`mk_config.py` writes out the configurations for the generation of MUBD3.0 virtual decoys. We provide `gen_decoys.sh` to iterate over all the ligands and set the configurations specific for each of them. Please make sure the `</path/to/REINVENT>` and `</path/to/MUBD3.0>` in all the scripts are replaced with user-defined directories.
```bash
$ mkdir output
$ chmod +x ./gen_decoys.sh
$ conda activate reinvent.v3.2
(reinvent.v3.2) $ ./gen_decoys.sh
```

### Build the unbiased decoy set (UDS 3.0)
The file with the directory of `output/ligand_$idx/results/scaffold_memory.csv` contains the potential decoy set specific for the `ligand_$idx`. The potential decoy set is refined by SMILES curation and structural clustering (script: `curating_clustering.py`). Then the unbiased decoys for each ligand were annotated with the properties and merged  (script: `merge_decoys.py`) to consitute the whole data set (`Final_decoys.csv`). We provide `build_uds.sh` to automatically runs the above mentioned scripts.
```bash
$ chmod +x ./build_uds.sh
$ conda activate MUBD3.0
(MUBD3.0) $ ./build_uds.sh
```

## Validation
The MUBD 3.0 is validated and measured with four metrics. Please go through the notebook `basic_validation.ipynb` for more details.
```bash
$ conda activate MUBD3.0
(MUBD3.0) $ jupyter notebook
```
