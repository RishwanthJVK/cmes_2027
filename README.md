# EEG Mind-Wandering Analysis: Project Documentation

## Purpose and scope

This repository contains two parallel, process-focused EEG analysis workflows for a mind-wandering study.  The workflows retain their respective source datasets, preprocessing materials, analysis notebooks/scripts, intermediate feature tables, and presentation-ready figures.  This document describes the purpose of every file type and each uniquely named file or repeated filename pattern.  It intentionally documents procedures, inputs, transformations, and artefact roles only; it does not interpret, summarise, or report analytical findings.

The two top-level directories are separate implementations of a common analytical programme:

* `dataset 1 analysis/` contains the 19-channel dataset, the original MATLAB/EEGLAB and FieldTrip workflow, and Python/Kaggle notebooks for band-power and machine-learning analyses.
* `dataset 2 analysis/` contains the BIDS-style 64-channel Breath Counting dataset and equivalent Python/Kaggle workflows, including standalone 64-channel scripts.

## Repository conventions

### Labels and analytical units

Both workflows derive binary analytical labels representing Focus and mind-wandering (MW) from their dataset-specific behavioural or trigger information.  Analyses operate at more than one level:

1. Continuous EEG recordings are preprocessed and segmented into labelled epochs.
2. Epochs are subdivided into overlapping time windows where a method requires temporal or feature-level resolution.
3. Window-level values are aggregated to epoch- or participant-level inputs for statistical modelling, or pooled within group-preserving validation folds for classification.

The frequency bands used by the Python band-power and feature-extraction workflows are Delta (1–4 Hz), Theta (4–8 Hz), Alpha (8–12 Hz), Beta (13–30 Hz), and Gamma (30–45 Hz).  Files whose names include `raw`, `Raw`, or `extracted` preserve the non-ICA branch; files whose names include `clean`, `ICA`, or `ica_cleaned` are derived from an ICA-cleaned branch.

### Directory and file naming

`output/` directories hold reproducible derived artefacts rather than new source data.  The repeated `csv files/` and `plots/` directories use the same roles throughout the repository:

* `csv files/` holds tabular data produced by an analysis step.
* `plots/` holds figures generated from those tables or from held-out predictions.
* `extracted data/` holds compressed, reusable epoch datasets.

No analytical conclusion should be inferred from the presence, ordering, or file size of any output artefact.

## Dataset 1 workflow (`dataset 1 analysis/`)

### Source dataset and original MATLAB pipeline (`mw_dataset1/`)

`mw_dataset1/Raw data/EEG/P_<participant>_MW.txt` comprises the continuous, participant-specific EEG text recordings.  Each file follows the same 21-channel plus event-column format used by the MATLAB import code.  The available `P_1_MW.txt` through `P_28_MW.txt` files are the primary EEG recordings; the participant numbers represented in the directory determine the available recordings.

`mw_dataset1/Raw data/EOG/MW_P<participant>.txt` comprises the matching continuous EOG recordings.  Each file provides the ocular channels and event information used to assess and remove ocular components from the corresponding EEG recording.  The repeated participant-number naming pattern establishes the EEG–EOG correspondence.

`mw_dataset1/Other files/EEG21locs.mat` stores the sensor-location structure used to assign channel positions during EEGLAB import, interpolation, component visualisation, and topographic processing.

`mw_dataset1/Other files/weights_per_epoch_after_rejection.mat` stores per-epoch confidence weights and condition indices after epoch rejection.  The later spectral and instantaneous-frequency scripts use it to calculate condition-specific weighted summaries.

`mw_dataset1/Other files/robust_estimation_1f_per_electrodeandsubject_excl_alpha.mat` stores the electrode- and participant-specific aperiodic spectral estimates generated for use by the thresholded peak-detection procedure.

`mw_dataset1/Other files/debriefings.xlsx` is the spreadsheet source for the study debriefing material and associated participant-level information.

`mw_dataset1/Other files/eprime task files/mind_wandering_experiment.es2` and `mind_wandering_experiment.ebs2` are the editable and compiled E-Prime task definitions, respectively.  They specify the experiment sequence, behavioural prompts, and trigger-producing task events.

`mw_dataset1/Other files/eprime task files/mind_wandering_experiment.wndpos` stores the E-Prime editor window-position configuration.  It does not participate in EEG analysis.

`mw_dataset1/Manuscript accepted in the European Journal of Neuroscience.pdf` is a reference manuscript retained with the dataset for methodological and study-context consultation; it is not executed by the analysis pipeline.

`mw_dataset1/readme.txt` provides the dataset's original accompanying notes.

### MATLAB scripts (`mw_dataset1/MATLAB scripts/`)

These scripts require MATLAB together with EEGLAB; `exploratory_condition.m` and `convertDatatoFieldtrip3.m` additionally require FieldTrip.  Path placeholders in the scripts must be configured before executing them on another system.

| File | Procedure implemented |
| --- | --- |
| `read_preprocess_and_epoch_EEG.m` | Imports each EEG text file; identifies event strings; applies 1–40 Hz filtering, flat-channel detection, ASR cleaning, interpolation, average rereferencing, epoching around the bell event, amplitude-based epoch marking, baseline removal, and ICA decomposition.  It saves an EEGLAB-compatible epoched structure for each participant. |
| `read_preprocess_and_epoch_EOG.m` | Imports the paired EOG text files, filters the two EOG channels, reconstructs bell events, epochs the signal using the same temporal interval, resamples the epoched EOG to 512 Hz, and saves participant-level EOG structures. |
| `get_behavioral_data.m` | Parses event annotations from the EEG text files to recover condition labels, confidence ratings, and arousal ratings, then constructs the participant-by-epoch behavioural arrays required for weighting and condition assignment. |
| `reject_components.m` | Loads matched EEG and EOG epoch files, aligns retained trials, visualises ICA components, correlates component activity with EOG, applies a manually specified component exclusion list, verifies before/after EOG correlation, and saves the cleaned EEG structure and component metadata. |
| `amplitude_analysis.m` | Runs short-time spectral analysis over each retained EEG epoch and electrode, calculates absolute and relative amplitude across the configured frequency range, and creates confidence-weighted condition-specific participant summaries. |
| `estimate_1f.m` | Computes a spectral estimate across concatenated clean epochs, omits the alpha range while fitting a robust log–log aperiodic trend, and stores the corresponding theta-to-alpha aperiodic spectrum for each participant and electrode. |
| `findpeaks_analysis_nothreshold.m` | Detects transient theta and alpha spectral peaks without aperiodic threshold subtraction; derives peak counts, peak-frequency summaries, and alpha/theta frequency-ratio occupancy, then calculates confidence-weighted condition summaries. |
| `findpeaks_analysis_1fthreshold.m` | Repeats transient theta/alpha peak extraction after subtracting the stored aperiodic estimate, and creates the same count, frequency, ratio, and confidence-weighted summaries. |
| `IF_and_PLV.m` | Filters each epoch into theta and alpha ranges, estimates analytic phase with the Hilbert transform, calculates 2:1 and 3:1 phase-locking values in sliding windows, estimates median-filtered instantaneous frequencies, and computes weighted condition-level summaries. |
| `IF_and_PLV_withfindpeaks.m` | Combines instantaneous-frequency and phase-locking calculations with peak-derived information, handling missing peak observations before condition-level weighted aggregation. |
| `convertDatatoFieldtrip3.m` | Converts matrix or cell-array electrode data to a FieldTrip data structure, assigns channel labels, sampling information and condition markers, and prepares the BioSemi layout used by FieldTrip analyses. |
| `permutation stats/exploratory_condition.m` | Converts condition-specific amplitude arrays to FieldTrip structures and specifies a paired, cluster-corrected Monte-Carlo permutation test with sensor neighbourhoods and a within-participant design matrix. |

`mw_dataset1/MATLAB scripts/permutation stats/electrode19.mat` stores the 19-channel label list used by the FieldTrip conversion helper.  `chanlocs.mat` stores the associated channel-location structure used to prepare the FieldTrip sensor layout.

### Dataset 1 Python/Kaggle analyses

Each `.ipynb` file is a self-contained, Kaggle-oriented notebook.  Its path constants identify the required mounted inputs and its `output/` directory preserves the artefacts created when it is run.

| Notebook | Procedure implemented |
| --- | --- |
| `dataset_1_band_wise_analysis/dataset_1_band_wise_analysis.ipynb` | Loads ICA-cleaned epochs and label data, normalises state labels, calculates Welch power spectra for overlapping windows across the 19-channel montage, derives log band-power trajectories and epoch summaries, fits participant-clustered models for state and within-epoch trend contrasts, and writes band-specific tables and trajectory figures. |
| `dataset_1_raw_vs_cleaned_SVM_Logistic/dataset_1_raw_vs_cleaned_SVM_Logistic.ipynb` | Loads matched raw and ICA-cleaned epochs, extracts spectral, envelope, burst and phase-locking features over windows, preserves epoch grouping during cross-validation, trains SVM and logistic-regression classifiers for each processing branch, aggregates window predictions at epoch level, and prepares metric and confusion-matrix artefacts. |
| `dataset_1_extratrees_and_band_importance/dataset_1_extratrees_and_band_importance.ipynb` | Extracts matched raw/ICA-cleaned window features, runs group-preserving ExtraTrees classification, records feature importances for each branch, aggregates importance by frequency band, and creates comparison figures and supporting tables. |
| `dataset_1_feature_wise_analysis/dataset_1_feature_wise_analysis.ipynb` | Computes an extended set of spectral, envelope, synchrony, time-domain, Hjorth, entropy, and complexity features; applies grouped feature selection and ExtraTrees modelling; assigns features to method families; and exports individual-feature and family-level importance tables and figures. |
| `dataset1_deep_mlp/dataset1_deep_mlp.ipynb` | Forms window-level feature vectors from raw and ICA-cleaned epochs, standardises data within group-preserving folds, trains a multilayer perceptron, aggregates its window predictions to epochs, and writes common classification summaries and figures for the two preprocessing branches. |

### Dataset 1 derived artefacts

The files below are generated by the corresponding notebooks and exist to preserve the intermediate and final products of their procedures.

* `dataset_1_band_wise_analysis/output/csv files/all_band_window_level_data.csv` contains window-level log-power values and the identifiers required to group them by participant, epoch, state, and time.  `alpha_epoch_summaries.csv`, `beta_epoch_summaries.csv`, `delta_epoch_summaries.csv`, `gamma_epoch_summaries.csv`, and `theta_epoch_summaries.csv` aggregate their respective bands to epoch-level mean-power and slope fields.  `all_band_statistical_results.csv` records the model specification fields, estimates, uncertainty fields, and adjusted decision columns produced by the notebook.  The six files in `output/plots/`—one named `*_band_power_over_time.png` per band plus `all_bands_power_over_time_standardised.png`—visualise the trajectories used in that workflow.
* `dataset_1_raw_vs_cleaned_SVM_Logistic/output/extracted data/extracted_dataset.zip` and `extracted_dataset_ICA_CLEANED.zip` package reusable raw and ICA-cleaned epoch inputs.  `output/csv files/df_main_phase1.csv` is the label/metadata table supplied to the classification workflow.  `all_epoch_features.csv` holds engineered epoch features, while `classification_metrics.csv` stores the cross-validated metric records.  `failed_runs.csv` reserves a structured record for files or sessions that could not be processed.  The five PNG files in `output/plots/` provide raw/ICA confusion matrices and per-model metric comparisons.
* `dataset_1_extratrees_and_band_importance/output/csv files/raw_and_clean_windowed_features.csv` is the full matched feature matrix.  `raw_feature_importance.csv` and `ica_cleaned_feature_importance.csv` store feature-importance vectors; `bandwise_feature_importance.csv` aggregates them by spectral band; and `raw_vs_clean_metrics.csv` records the evaluation fields.  Its four PNGs display the metric comparison, two confusion matrices, and bandwise importance visualisation.
* `dataset_1_feature_wise_analysis/output/csv files/windowed_features_all_families.csv` stores the complete feature-family matrix.  `feature_importance_by_feature.csv` stores model-derived importance for each feature and `feature_importance_by_umbrella.csv` stores the family grouping and aggregated importance fields.  `top_individual_features.png` and `feature_family_comparison.png` plot these two levels of the feature inventory.
* `dataset1_deep_mlp/output/csv files/raw_vs_clean_metrics.csv` stores the DeepMLP evaluation fields.  `raw_confusion_matrix.png`, `ica_cleaned_confusion_matrix.png`, and `raw_vs_clean_classification_metrics.png` are the diagnostic and metric-comparison plots created by that notebook.

## Dataset 2 workflow (`dataset 2 analysis/`)

### BIDS-style source dataset (`mwdataset/`)

This directory contains 64-channel BioSemi BDF recordings organised across two participants (`sub-01`, `sub-02`) and eleven sessions per participant.  Recordings are distributed among the archive directories named `sub-01-20260605T131512Z-3-001` through `-003` and `sub-02-20260605T131514Z-3-001` through `-003`.  The Python scripts map every session to the archive that contains its BDF file.

The following repeated file patterns are fully equivalent in role; the participant and session segment selects the relevant recording:

| Pattern | Purpose |
| --- | --- |
| `sub-*/eeg/sub-*_ses-*_task-BreathCounting_eeg.bdf` | Continuous BioSemi BDF recording for one participant/session.  It includes EEG, external channels, and trigger information read by MNE. |
| `sub-*/eeg/sub-*_ses-*_task-BreathCounting_channels.tsv` | Channel inventory for that session.  The scripts use its `channelTypes` column to select the 64 EEG channels and separate EXG channels. |
| `sub-*/eeg/sub-*_ses-*_task-BreathCounting_electrodes.tsv` | Electrode labels, three-dimensional positions, and electrode material/type metadata for the same session. |
| `sub-*/eeg/sub-*_ses-*_task-BreathCounting_eeg.json` | BIDS sidecar metadata describing the Breath Counting EEG recording. |
| `sub-*/sub-*_events.tsv` | Event-code dictionary.  It documents the trigger codes used to create labelled Focus and MW epochs. |
| `sub-*/sub-*_sessions.tsv` | Session identifiers and acquisition timestamps for the participant. |
| `sub-*/anat/sub-*_T1.nii` | Participant-level T1 anatomical image retained as part of the BIDS dataset; it is not read by the EEG scripts in this repository. |

### Dataset 2 Python/Kaggle notebooks

| Notebook | Procedure implemented |
| --- | --- |
| `dataset_2_band_wise_analysis/dataset_2_band_wise_analysis.ipynb` | Locates BDF/metadata files by participant and session, selects EEG channels, rescales event samples after resampling, filters the recordings, performs EXG-correlated ICA cleaning, derives trigger-aligned Focus/MW epochs, calculates Welch band power in overlapping windows, fits clustered state/trend models, and writes tables and figures. |
| `dataset_2_raw_vs_cleaned_SVM_Logistic/dataset_2_raw_vs_cleaned_SVM_Logistic.ipynb` | Applies matched BDF preprocessing to raw and ICA-cleaned branches, extracts window features, uses grouped cross-validation, fits SVM and logistic-regression classifiers, aggregates predictions to the epoch level, and exports metrics and confusion-matrix figures. |
| `dataset_2_extratrees_and_band_importance/dataset_2_extratrees_and_band_importance.ipynb` | Produces raw and cleaned feature matrices, applies grouped ExtraTrees classification and band-level importance aggregation, and saves branch-specific feature/importance tables and visual summaries. |
| `dataset_2_feature_wise_analysis/dataset_2_feature_wise_analysis.ipynb` | Extracts broad spectral, temporal, entropy/complexity, envelope and connectivity feature families; applies grouped univariate selection and ExtraTrees modelling; and exports individual and family-level importance inventories. |
| `dataset2_deepmlp/dataset2_deepmlp.ipynb` | Builds BDF-derived window vectors for both processing branches and uses a multilayer perceptron with foldwise imputation/scaling, early stopping, and epoch-level probability aggregation. |

### Standalone 64-channel scripts (`mwdataset/`)

| File | Procedure implemented |
| --- | --- |
| `mwdataset_64ch_band_power_analysis.py` | Reimplements the 64-channel band-power workflow outside the notebook environment.  It maps BDF and metadata locations, reads trigger events, selects 64 EEG/EXG channels, resamples to 256 Hz, filters 0.5–45 Hz, removes ICA components correlated with EXG channels, constructs fixed-duration event-aligned epochs, estimates band power using Welch windows, fits participant-clustered models, applies Benjamini–Hochberg FDR correction, and writes tables and figures.  Its input/output paths can be overridden with environment variables. |
| `mwdataset_64ch_feature_family_analysis.py` | Reimplements feature-family analysis for all 64 EEG channels.  It performs the same BDF loading, resampling, filtering, EXG-correlated ICA cleaning, and event epoching; derives spectral power, envelope, time-domain, Hjorth, entropy, synchrony, and pairwise phase-locking features; removes exact duplicate columns; performs participant-grouped cross-validation with median imputation, univariate selection, and ExtraTrees; assigns features to families; and writes tabular/figure artefacts. |
| `dataset2_deepmlp_raw_vs_exg_cleaned.py` | Executes the raw-versus-ICA DeepMLP workflow as a script.  It implements deterministic seeding, matched BDF preprocessing, 1.5-second overlapping-window feature vectors, a PyTorch fully connected network, within-subject stratified epoch folds, foldwise imputation/standardisation, learning-rate scheduling and early stopping, epoch-level probability aggregation, and metric/confusion-matrix exports. |
| `mwdataset_64ch_analysis.log` | Runtime log capturing session-level progress messages from the band-power script. |
| `mwdataset_64ch_analysis.err.log` | Standard-error log for the band-power script. |
| `mwdataset_64ch_feature_family_analysis.log` | Runtime log capturing feature-family extraction progress. |
| `mwdataset_64ch_feature_family_analysis.err.log` | Standard-error log for the feature-family script. |

### Dataset 2 derived artefacts

* `dataset_2_band_wise_analysis/output/csv files/all_band_window_level_data.csv`, the five `*_epoch_summaries.csv` files, and `all_band_statistical_results.csv` have the same window, epoch-summary, and model-record roles as their Dataset 1 counterparts.  The six PNGs in `output/plots/` contain the individual-band and standardised multi-band trajectory visualisations.
* `dataset_2_raw_vs_cleaned_SVM_Logistic/output/csv files/all_epoch_features.csv` is the classifier feature table; `classification_metrics.csv` stores the model/branch/metric fields; and `failed_runs.csv` is the structured processing-exception record.  Its five plot files contain SVM and logistic-regression confusion matrices and the combined metric display.
* `dataset_2_extratrees_and_band_importance/output/csv files/windowed_features_raw_and_ica_cleaned.csv` is the common long feature table.  `raw_wide_features.csv` and `ica_cleaned_wide_features.csv` are branch-specific wide matrices.  `raw_feature_importance.csv`, `ica_cleaned_feature_importance.csv`, `raw_bandwise_feature_importance.csv`, and `ica_cleaned_bandwise_feature_importance.csv` record individual-feature and band-grouped importance fields.  `raw_vs_ica_cleaned_classification_metrics.csv` preserves metric records, and `failed_runs.csv` records exceptions.  The four PNGs plot metrics, confusion matrices, and the bandwise feature inventory.
* `dataset_2_feature_wise_analysis/output/csv files/windowed_features_all_families.csv` stores the extracted all-family window matrix; `feature_importance_by_feature.csv` and `feature_importance_by_umbrella.csv` preserve the feature- and family-level model inventories.  `top_individual_features.png` and `feature_family_comparison.png` plot those inventories.
* `dataset2_deepmlp/output/csv files/window_sample_metadata.csv` retains non-vector identifiers for every DeepMLP window.  `raw_vs_ica_cleaned_deepmlp_metrics.csv` stores its metric fields, and `failed_runs.csv` records sessions that could not be processed.  The three PNGs contain the two branch-specific confusion matrices and the DeepMLP metric display.
* `mwdataset/mwdataset_64ch_all_band_results/` contains the standalone band-power script’s `all_band_window_level_data.csv`, five per-band epoch-summary CSVs, `all_band_statistical_results.csv`, five per-band trajectory plots, and `all_bands_power_over_time_standardised.png`.  Each file has the same procedural role as described above for the notebook-based band-power workflow.
* `mwdataset/mwdataset_64ch_feature_family_results/` is the designated destination for the standalone feature-family script.  When populated, it contains `windowed_features_all_families.csv`, `feature_importance_by_feature.csv`, `feature_importance_by_umbrella.csv`, `top_individual_features.png`, and `feature_family_comparison.png`, with the same roles as the notebook feature-family artefacts.

## Reproducing the workflows

The original Dataset 1 preprocessing requires MATLAB, EEGLAB and FieldTrip.  The notebook and standalone Python workflows require an environment providing Python 3 with MNE, NumPy, pandas, SciPy, statsmodels, scikit-learn, matplotlib and seaborn.  The DeepMLP workflows also require PyTorch.  Kaggle paths embedded in the notebooks/scripts should be replaced or supplied through the documented environment variables when running locally.

Run analyses within their own directories to retain each workflow's expected relative `output/` structure.  Raw and ICA-cleaned comparisons should be executed from the paired source recordings specified by the relevant notebook or script so that the two branches retain matched event timing and epoch identities.

## Data stewardship

The repository contains identifiable participant labels in source-file paths and substantial raw physiological recordings.  Before external release or conference dissemination, apply the governing ethics approval, consent restrictions, data-use agreement, and de-identification procedures.  The generated tables and plots should be treated as derived research materials and kept linked to the exact source version and analysis environment used to generate them.
