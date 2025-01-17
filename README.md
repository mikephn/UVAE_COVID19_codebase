## UVAE: Integration of Heterogeneous Unpaired Data with Imbalanced Classes
### associated codebase.

This repository contains the code used in the paper to extract data, train, and benchmark models.

Data files including training data, models, and results are accessible on Zenodo: `https://doi.org/10.5281/zenodo.13854783`.

You will also need the UVAE framework from `https://github.com/mikephn/UVAE` placed in the project folder.

- [1 Extracting flow cytometry samples](#1-extracting-flow-cytometry-samples)
- [2 Generating synthetic data](#2-generating-synthetic-data)
- [3 Benchmarking with synthetic data](#3-benchmarking-with-synthetic-data)
- [4 Comparison with cyCombine](#4-comparison-with-cyCombine)
- [5 Training data integration models](#5-training-data-integration-models)
- [6 Training longitudinal regression models](#6-training-longitudinal-regression-models)

## 1 Extracting flow cytometry samples

`extract_flow.R` was used to process .fcs files from a number of experiments. The script allows for subsampling flow cytometry events either from raw files or extracting corrected and annotated files from .wsp workspaces. Rare cell-type sub-populations can be up-sampled and separately stored. Combined dataset is stored in a .hdf5 file. 

`data/lineage.h5` and `data/chemo.h5` contain combined, sub-sampled, annotated patient data from lineage and chemokine panels, respectively, saved in HDF5 format. They can be read with a class `HdfDataset` from `HdfDataset.py`, which provides functions to obtain series of data filtered by common feature sets.

`data/channels-rename.tsv` contains assignment of experimental channels to a common namespace of flow features and markers. This assignment was used to combine analogous features for further processing.

`data/meta.csv` contains assignments of samples to anonymised patient IDs, peak disease severity, processing batch, and timepoints.

## 2 Generating synthetic data

`ToyDataUnsup.py` contains the script to generate synthetic data from a source flow file `data/WB_main 20200504_YA HC WB_main_018.fcs`. Synthetic data is set to contain 3 panels (with disparate features), with 3 batches in each. The assignment of GMM clustering from `scikit-learn` is used to unevenly split the data across batches.

The synthetic dataset is saved as `data/ToyDataWB-3x3.pkl`.

## 3 Benchmarking with synthetic data

`UVAE_test_configs.py` contains a list of model configurations which were tested. These configurations contain settings recognised by the training script to create a particular model configuration.

`UVAE_test.py` is the training script for instantiating a particular model configuration, training it, and saving test results. It trains the ground truth models when one of the first 3 configs is specified.

## 4 Comparison with cyCombine

CLL samples were downloaded from `http://flowrepository.org/id/FR-FCM-Z52G` and unpacked to `FlowRepository_FR-FCM-Z52G_files` folder. `cll_prep.R` was used to apply the pre-processing defined by cyCombine authors, and save relevant columns to .RDS. `train-cycombine-comp.py` contains the script used for training three comparison models (on panel 1, panel 2, and a common batch of the two panels).

## 5 Training data integration models

Two models are trained, one for lineage panels, and one for neutrophils from the chemokine panel. For both models, a number of the largest panels are first integrated. Hyper-parameters are optimised with the search scores saved to `hyper_lisi.csv`. Then the remaining panels are trained to project on top of the initial latent space.

`train-lineage.py` contains the script for training the lineage model. It is first run with the `test` variable set to False, then repeated with it set to True. The resulting model is saved as `model_lineage.uv`. After training, a subset of corrected data is saved to `embs/lineage.pkl`, which is used in subsequent training of the longitudinal regression models.

`train-chemo-neutro.py` contains the script for training the neutrophil chemokine model. The resulting model is saved as `model_chemo.uv`. A subset of corrected data is saved to `embs/chemo.pkl`. For this dataset, Gaussian Mixture Models are used to cluster the data, and are saved in the `gmm` folder.

## 6 Training longitudinal regression models

`train-severity-reg.py` contains a script for training and evaluating individual regression models. The list of `configs` defines each model configuration, a configuration is selected by providing a variable externally or setting the `config_ind` variable. After training the models, setting `config_ind` to None will aggregate all results into a table. By setting `gradientAttribution` to True, gradients of severity will be collected during prediction for patient IDs defined in `pids_to_plot` (by default, PIDs with number of timesteps >= `plot_min_timesteps`, set to 3 are included).
