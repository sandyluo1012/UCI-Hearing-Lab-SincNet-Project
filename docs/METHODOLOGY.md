# Methodology and limitations

## Historical study

The researcher initially had only synthetic data. Experiment records became available later. The two stages use the same family of models but separate dataset splits and models trained from scratch (`pt_file=none`). Neither a participant-response model nor a synthetic-to-experiment transfer evaluation is implemented.

The upstream SincNet architecture learns a parameterized band-pass front end, followed by convolutional and fully connected layers. This project changes the task to two stimulus classes and varies the first-layer filter count. It does not introduce a new neural architecture.

## Interpreting the results

- The experiment-derived test split has 80 clips. One additional correct clip changes accuracy by 1.25 percentage points. The recorded 76.25% corresponds to 61/80 correct classifications at the last evaluation for the 10-filter run.
- Highest logged accuracy is selected over repeated evaluations of the same test split. There is no separate validation/final-test protocol documented here.
- The original random generation and splitting were not fully seeded. Retained filename splits help preserve the historical partition, but regenerated synthetic audio will differ.
- The two datasets differ in synthesis, duration, size, and training settings. Their scores are not a controlled causal comparison of synthetic versus experimental data.
- Saved default configs do not constitute complete per-run provenance for every archived checkpoint. Filter counts are identified by historical log filenames. Some logs include multiple epoch-zero entries, consistent with appended runs, but run boundaries were not fully recorded.
- A single participant's stimulus set cannot establish generalization across people. The model is not trained on that participant's answers in the first place.

## Historical evaluation behavior

The trainer allocates `N_fr + 1` frame predictions but fills them using `while end_samp < signal.shape[0]`. When the remaining length is an exact multiple of the frame shift, one allocated row remains zero and the final boundary-aligned frame is skipped. This affects frame-level loss/error denominators. The extra zero row contributes nothing to the sum used for clip classification; the original clip score still uses only the processed frames.

The behavior is documented rather than silently changed so that historical results remain distinguishable from any future corrected evaluation. A future rerun should correct the frame loop, introduce a separate validation split, and report new scores under a new run identifier.

## Visualization scope

Original exploratory notebooks plotted outputs obtained by passing random inputs through the first convolutional layer, and attempted to reconstruct frequency responses. A layer output is not the filter kernel. The frequency reconstruction also needs review of sample-rate normalization and the model's cutoff constraints. Those notebooks are archived locally rather than presented as validated filter estimates.

The public analysis notebook plots logged performance only. It makes no claim to estimate human auditory filters.

## Repository preparation

The 2026 portfolio cleanup adds documentation, a log summary, and a results notebook. It removes unrelated upstream speaker-identification examples from the public tree, trims unused utility code, and adds directory creation and overwrite guards to preparation notebooks. Synthetic generation now sets a seed; split generation preserves and validates existing splits. The core model, optimization, and historical evaluation behavior are retained. These maintenance changes are not represented as work completed during the 2024 research period.

Original local files are retained in `.local_archive/`, and data/checkpoints remain in ignored paths. Training has not been rerun as part of this cleanup. Runtime dependencies were not available in the cleanup environment; structural and log-consistency checks are reported separately from runtime execution.
