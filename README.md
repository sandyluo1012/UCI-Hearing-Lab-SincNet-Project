# Auditory Signal Classification with SincNet

[中文说明](README.zh-CN.md)

An exploratory research project developed at the **UC Irvine Hearing Lab (June–September 2024)**. It adapts an existing waveform-based neural network to classify auditory stimuli and compares the effect of first-layer filter count.

The project began with synthetic stimuli while participant experiment data were unavailable. After experiment data became available, audio was reconstructed from the stored acoustic parameters and a separate set of models was trained and evaluated.

**Task:** predict the stimulus condition, not the participant's reported answer. Synthetic and experiment-derived datasets have separate training and test splits; this repository does not demonstrate synthetic-to-experimental transfer learning.

## Data and workflow

| | Synthetic stimuli | Experiment-derived stimuli |
|---|---|---|
| Source | Programmatically specified tone mixtures | Acoustic parameters from a participant listening experiment |
| Audio clips | 300 | 400 |
| Duration / sample rate | 0.5 s / 44.1 kHz | 0.2 s / 44.1 kHz |
| Train / test clips | 240 / 60 | 320 / 80 |
| First-layer filters compared | 5, 10, 80 | 5, 10, 15, 20 |

Synthetic clips combine tones from 300 to 2780 Hz at 20 Hz intervals. Target components at 500, 1000, 1500, 2000 and 2500 Hz have mean amplitude coefficients of 53 versus 50 in the two conditions, with random amplitude perturbations and phases. **Both classes contain the target frequencies**; the difference is their amplitude distribution. These coefficients are not calibrated sound-pressure levels.

Experiment clips are reconstructed from 23 components using `freqval` (frequency), `orderdb` (level), and `orderphase` (phase). The original reconstruction groups trials using `nint - 1` and assigns labels through the resulting filenames. The participant's `resp` field is not used as the SincNet target. Both datasets are synthesized waveforms; experiment-derived clips are not microphone recordings.

Workflow: prepare audio → assign stimulus labels → split files → train SincNet and fully connected layers → evaluate clips → compare configurations.

## Historical results

Accuracy below is **1 − `err_te_snt`**, the full-clip error stored in the original logs. Values were extracted from saved logs, not obtained by rerunning training during repository cleanup.

| Dataset | First-layer filters | Last evaluated epoch | Accuracy at last evaluation | Highest logged accuracy |
|---|---:|---:|---:|---:|
| Synthetic | 5 | 96 | 46.67% | 53.33% |
| Synthetic | 10 | 96 | 41.67% | 51.67% |
| Synthetic | 80 | 96 | 50.00% | 60.00% |
| Experiment-derived | 5 | 49 | 67.50% | 75.00% |
| Experiment-derived | 10 | 49 | **76.25%** | **78.75%** |
| Experiment-derived | 15 | 49 | 73.75% | 77.50% |
| Experiment-derived | 20 | 49 | 73.75% | 77.50% |

In the recorded experiment-derived runs, increasing filter count beyond 10 did not improve final test accuracy. Synthetic runs achieved low training error but weak held-out performance, consistent with overfitting. These are exploratory observations from small datasets, not a statistically established ranking of architectures.

- Training evaluates the same held-out split repeatedly. The highest logged score is not an independent final-test estimate.
- Epochs are zero-indexed. Synthetic training runs through epoch 99, but the last scheduled evaluation is epoch 96.
- Some logs contain repeated epoch-zero entries. The analysis retains them as recorded and does not claim they are independent replications.
- The historical evaluator can include an unused zero row in frame-level metrics. The zero row adds nothing to the clip score sum; this does not validate the remaining evaluation protocol. See [methodology and limitations](docs/METHODOLOGY.md).

## Repository contents

| Path | Purpose |
|---|---|
| `simulation_WAV.ipynb` | Generate synthetic audio locally |
| `experiment_WAV.ipynb` | Reconstruct experiment audio from local MATLAB files |
| `generate_scp.ipynb` | Validate/preserve file splits and generate labels |
| `sincnet_binary_response.py` | Historical CUDA training and evaluation workflow |
| `dnn_models.py` | Upstream SincNet and MLP implementation |
| `data_io.py` | Configuration and file-list utilities |
| `cfg/` | Separate synthetic and experiment configurations |
| `data_lists/` | Retained historical filename splits; no audio or responses |
| `results/raw/` | Seven historical training logs |
| `results/summary.csv` | Summary extracted from those logs |
| `analysis/results_analysis.ipynb` | Recreate the summary and plot learning curves |

## Run the project

Use a Python environment with the packages listed in `requirements.txt`. The original package versions were not preserved; this is a dependency inventory, not a tested lockfile. Training uses unconditional `.cuda()` calls and requires a compatible NVIDIA GPU and CUDA-enabled PyTorch installation. Audio preparation and log analysis do not require a GPU.

```bash
python -m pip install -r requirements.txt
python -m jupyterlab
```

Run notebooks with the **repository root as their working directory**. The analysis notebook also accepts its own `analysis/` directory.

1. Run `simulation_WAV.ipynb` to create `sim_data/`. Generation now uses a fixed seed for future runs and refuses to replace existing WAV files.
2. Run the synthetic cell in `generate_scp.ipynb`. It keeps the checked-in historical split and generates `data_lists/sim_labels.npy`.
3. Train from the repository root:

   ```bash
   python sincnet_binary_response.py --cfg=cfg/SincNet_Sim.cfg
   ```

4. For the experiment branch, supply authorized local MATLAB files at the paths in `experiment_WAV.ipynb`, run that notebook, then run the experiment cell in `generate_scp.ipynb`:

   ```bash
   python sincnet_binary_response.py --cfg=cfg/SincNet_Exp.cfg
   ```

5. To inspect historical results without private data or a GPU, run `analysis/results_analysis.ipynb`.

The checked-in configurations select **80 synthetic filters** and **20 experiment filters**. To run another configuration, copy the relevant config, change the first `cnn_N_filt` value, and choose a new `output_folder`. Each run appends its log and overwrites its checkpoint within that output folder. Both configs use `pt_file=none` (training from scratch).

The original synthetic generation was unseeded. Regenerating audio now preserves the design, not the exact historical waveforms or scores. Only saved splits and logs are available publicly. Training was not rerun during this cleanup.

## Data availability and project scope

Participant MATLAB records, reconstructed audio, generated WAVs, label binaries, trained checkpoints, and spreadsheet exports are excluded through `.gitignore`. The experiment branch requires local data that are not distributed here. See [data dictionary](docs/DATA_DICTIONARY.md).

The work provides a stimulus-classification workflow and configuration comparisons. It does **not** establish participant-response prediction, reduced human testing time, adaptive test selection, or estimation of individual auditory filters. Those are possible future research questions, not completed outcomes.

## Attribution

This project adapts [Mirco Ravanelli's SincNet repository](https://github.com/mravanelli/SincNet), based on *Speaker Recognition from raw waveform with SincNet* by Mirco Ravanelli and Yoshua Bengio. The model architecture is upstream work; this project's additions are auditory data preparation, binary-task adaptation, experiment configurations, and analysis.

The original [MIT license](LICENSE) and copyright notice are retained. It applies to the code; private laboratory data are not included. This is a personal research-project archive, not an official laboratory software release.
