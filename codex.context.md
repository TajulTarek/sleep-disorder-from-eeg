You recovered enough to restart **healthy-versus-unhealthy EEG classification offline** and rebuild the datasets for **five-disorder classification**. The most important thing to recover from Drive is the **per-subject EEG segments or a row-to-subject mapping**. That would let you test whether the models generalize to people they have never seen.

I reviewed all seven notebooks and the `.ipynb.txt` export, and scanned every row of the 27 CSVs, about 21.8 GB. The files had the expected column counts and label values. I also ran one CPU training batch and a prediction using the binary skip-CNN architecture and 16 local EEG windows. No workspace files were changed.

## Which notebook does what

| Notebook | Purpose and where it fits |
|---|---|
| [Dataset_preparation_CAP_phases.ipynb](<C:/Users/tarek/Colab Notebooks-for eeg/Dataset_preparation_CAP_phases.ipynb>) | Combines per-subject MATLAB segments into cohort CSVs, then balances CAP phases. Its CSV outputs are already present in `Data_combine` and `Data_csv1`. The source `.mat` files are missing. |
| [Dataset_preparation_disease_classification.ipynb](<C:/Users/tarek/Colab Notebooks-for eeg/Dataset_preparation_disease_classification.ipynb>) | Turns the balanced cohort files into two dataset families, healthy versus unhealthy and five-way disorder classification. The three binary outputs are present. The three five-way outputs can be rebuilt from local inputs. |
| [Healthy_Unhealthy_Classification.ipynb](<C:/Users/tarek/Colab Notebooks-for eeg/Healthy_Unhealthy_Classification.ipynb>) | Main notebook to restart with. It reads `healthy_unhealthy1.csv` and contains a plain 1D CNN, a skip-connection CNN, a CNN–LSTM, and an Inception-style CNN–GRU. Start with the skip-CNN in cells 10–11. |
| [Healthy_Unhealthy_Classification.ipynb.txt](<C:/Users/tarek/Colab Notebooks-for eeg/Healthy_Unhealthy_Classification.ipynb.txt>) | Another version of the notebook saved as JSON with a `.txt` extension. It has extra A-only and B-only evaluation sections and archived outputs. It references missing models and saved splits, and its reported test sizes differ from the main notebook. Keep it as a record of earlier runs. |
| [Disease_Classification.ipynb](<C:/Users/tarek/Colab Notebooks-for eeg/Disease_Classification.ipynb>) | Trains a CNN–LSTM to distinguish five disorders among the unhealthy cohorts. It needs the rebuilt `both_phases.csv`. The saved run was interrupted during epoch 10, before final test evaluation. |
| [transformer.ipynb](<C:/Users/tarek/Colab Notebooks-for eeg/transformer.ipynb>) | Experimental EEGPT version of the same five-way task. The EEGPT code and checkpoint are absent, and the saved run failed because `EEGPTClassifier` had no `from_pretrained` method. The notebook passes 1,024-sample single-channel windows to an experiment referencing a `58chs_4s` checkpoint, so input compatibility also needs checking. |
| [testColab.ipynb](<C:/Users/tarek/Colab Notebooks-for eeg/testColab.ipynb>) | Prototype for raw EEG extraction. It reads `brux1.edf` and its annotation text, selects `F4-C4`, and extracts A1 windows using a hard-coded 512 Hz. The input pair is absent, and the notebook stops at an in-memory result. The code that created the full MATLAB dataset was not in this folder. |
| [dataset_trim.ipynb](<C:/Users/tarek/Colab Notebooks-for eeg/dataset_trim.ipynb>) | Separate audio project. It trims leading silence from WAV files and copies annotations under `AI-Fiction/shobdotori`. It has no dependency on the EEG pipeline. |

There are three distinct tasks in this collection. **CAP phase** is a label in the prepared data. **Healthy versus unhealthy** is the binary model task. **Which disorder** is the five-class model task. I found preparation code for CAP phase labels, with no dedicated four-class CAP phase training notebook in this download.

## What’s already on disk

- [Data_combine](<C:/Users/tarek/Colab Notebooks-for eeg/FYP/A1_A2_A3_B_data/Data_combine>) contains 11 merged cohort CSVs before phase balancing. RBD and NFLE were processed in numbered batches.
- [Data_csv1](<C:/Users/tarek/Colab Notebooks-for eeg/FYP/A1_A2_A3_B_data/Data_csv1>) contains 13 balanced CSVs, including the six cohort files consumed by the next preparation notebook. The five disorder files each have enough windows of every phase to rebuild the five-way dataset.
- [healthy_unhealthy_classification/dataset](<C:/Users/tarek/Colab Notebooks-for eeg/FYP/healthy_unhealthy_classification/dataset>) contains the three binary datasets below. I verified the counts directly.

| Local binary dataset | Windows | Class balance |
|---|---:|---|
| `healthy_unhealthy1.csv` | 21,600 | 10,800 healthy and 10,800 unhealthy |
| `healthy_unhealthyA.csv` | 10,800 | 5,400 per class, phases A1–A3 |
| `healthy_unhealthyB.csv` | 10,800 | 5,400 per class, phase B |

The CSVs are headerless. Columns 1–1024 hold EEG samples and column 1025 holds the CAP phase (`0=B`, `1=A1`, `2=A2`, `3=A3`). The final binary datasets add column 1026 for healthy (`0`) or unhealthy (`1`). The five-way preparation code uses that same last column for insomnia (`0`), NFLE (`1`), narcolepsy (`2`), RBD (`3`), or PLM (`4`).

## What to look for on Drive

| Priority | Old Drive location or file | Why it matters |
|---|---|---|
| **Recover first** | `MyDrive/Done/DATA_A1_A2_A3_B/` | The CAP prep notebook expects four `.mat` files per listed subject, such as `n1_A1.mat` and `n1_B_F4C4.mat`. Recover this folder, or a trusted row-to-subject manifest, for subject-level splits. Also look for the MATLAB scripts or notes that produced these segments, especially sampling rate, units, filtering, and windowing. |
| **Recover for exact old results** | `MyDrive/ML/H&U_classification/skip/` and `lstm/` | Look for `model_v1.h5`, `model_vA1.h5`, `model_vB1.h5`, `model_v4.h5`, their histories and corresponding `xtrain`, `ytrain`, `xtest`, and `ytest` exports. The text export also mentions `model_skip_HU (1).h5`. The old splits were unseeded, so the notebooks and pooled CSVs alone cannot reproduce those exact splits. |
| **Rebuild locally** | `MyDrive/ML/Disease_classification/Dataset/` | `both_phases.csv`, `A_phases.csv`, and `B_phases.csv` are absent. The saved prep code and verified local inputs are sufficient to recreate the expected 29,310 combined windows and 14,655 windows in each phase subset. |
| **Recover if revisiting raw extraction or EEGPT** | `MyDrive/TEST/` and `MyDrive/ML/Disease_classification/EEGPT/` | The raw prototype needs `brux1.edf` and `brux1.txt`. The transformer experiment needs the EEGPT source version and `eegpt_mcae_58chs_4s_large4E.ckpt`, plus changes to the loading and training code. |

## How I’d restart

1. **Get the binary skip-CNN running locally first.** In `Healthy_Unhealthy_Classification.ipynb`, replace the Drive mount and input path with the local CSV, skip the Colab package-install cell, and use a fresh local directory for outputs. Use `float32` and shape the model input as `(windows, 1024, 1)`. Start with one short run through cells 5–6 and 10–11. Evaluate the model returned by that run against its own held-out data. The notebook reuses `model1` in the LSTM evaluation section after loading the skip-CNN into that variable, so those later cells can report predictions from the wrong model.
2. **Use a clean local TensorFlow environment.** Your current Python 3.11 environment contains TensorFlow 2.18.0, but importing it normally failed because JAX 0.7.0 expects an attribute unavailable in the installed `ml-dtypes` 0.4.1. For the smoke test, I bypassed the optional JAX import in that process only. The skip-CNN trained one batch and returned 16 predictions on CPU. That verifies the local CSV, input shape, and model wiring. The full training run is still unverified.
3. **Recover subject IDs before producing new research results.** The original notebooks randomly split EEG windows, which allows windows from the same person to appear in training and testing. Treat any run on the existing pooled files as a development baseline. With the per-subject files, carry the subject ID into the new dataset, split by subject first, and apply any training resampling within the training set. Then compare A-only, B-only, and combined inputs using the same held-out subjects.
4. **Rebuild the five-way datasets next.** Use the disease section of `Dataset_preparation_disease_classification.ipynb` with local paths and a fresh output folder. The original code takes the first matching windows in each cohort and phase, so keep that selection if you are reconstructing the historical experiment. For a new experiment, reuse the subject-level split design.

The quickest restart is the **binary skip-CNN with `healthy_unhealthy1.csv`**. The highest-value recovery is **`Done/DATA_A1_A2_A3_B/` and its preprocessing code**, because that is what can turn a working offline baseline into a defensible subject-level experiment.