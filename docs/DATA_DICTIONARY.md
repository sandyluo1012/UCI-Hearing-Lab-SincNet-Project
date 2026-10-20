# Data dictionary

## MATLAB inputs (local only)

| Field | Meaning / use in this project |
|---|---|
| `data` | MATLAB cell array of experiment blocks, each containing a `track` structure |
| `track.nint` | Original experiment code used by the reconstruction to separate stimulus conditions; subtract 1 to obtain 0/1 |
| `track.resp` | Recorded participant answer; not used by the SincNet pipeline |
| `freqval` | Frequencies of the tone components; reconstruction uses 23 components |
| `orderdb` | Per-trial component levels, converted by `10 ** (orderdb / 20)` and scaled by `1 / 500000` |
| `orderphase` | Per-trial component phases, in radians in the cosine expression |

Sampling rate (44,100 Hz), duration (0.2 s), and linear onset/offset ramps (5 ms) are specified in the Python reconstruction code. They are not read from the MATLAB input fields.

The original local filenames are `gr2data.mat` and `stim_params_gr2.mat`. They are excluded from version control. Confirm the experiment protocol before assigning any additional behavioral interpretation to `nint` or `resp`.

## Generated audio and labels

| Dataset | Label 1 filenames | Label 0 filenames |
|---|---|---|
| Synthetic | `sig1.wav` … `sig150.wav` | `nosig1.wav` … `nosig150.wav` |
| Experiment-derived | `sigwf*.wav` | `nswf*.wav` |

Labels describe stimulus conditions. They do not encode whether the participant answered correctly or reported hearing a signal.

`data_lists/{sim,exp}_{train,test}.scp` contain one filename per line. These retained splits contain 240/60 and 320/80 entries respectively. Generated `{sim,exp}_labels.npy` files map filename to integer class and stay local.

Experiment filenames are numbered **within each condition**, rather than by original trial order. A future response-prediction task must reconstruct the original trial-to-audio mapping before assigning `resp` labels. Simply attaching the original response vector to the directory listing would be incorrect.

## Saved log metrics

| Metric | Definition in the historical trainer |
|---|---|
| `epoch` | Zero-based training epoch |
| `loss_tr`, `err_tr` | Mean training minibatch loss / classification error |
| `loss_te`, `err_te` | Held-out frame metrics, averaged per clip; see evaluator limitation |
| `err_te_snt` | Full-clip classification error, using the sum of frame log probabilities |

`results/summary.csv` contains the source filename, dataset, filter count, evaluation-row count, last evaluated epoch, and last/highest logged clip accuracy (fractions from 0 to 1). Duplicate logged epochs are not discarded. The CSV's `test_clips` is the retained split size, not a record of how many times each clip was evaluated.
