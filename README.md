# risk-calibrated-stego

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24286822.svg)](https://doi.org/10.5281/zenodo.24286822)

Conformal risk control for per-image payload allocation in spatial-domain image steganography, and a test of whether the resulting guarantee transfers to steganalyzers the sender never queries.

## What the code does

A sender embeds with HILL costs in grayscale 256×256 BOSSbase 1.01 covers and chooses a payload for each cover. Three allocation rules are compared:

- **Score-driven per-image**: the largest payload that the sender's own detectors do not flag.
- **Calibrated fixed payload**: one payload for every cover.
- **Detectability-limited**: the largest payload whose model-based deflection stays below a bound.

Each rule is a nested family indexed by one threshold, with a monotone loss (running-maximum envelope over nested embeddings). The threshold is calibrated by conformal risk control so that the expected detection risk measured by the sender's detector family (SRNet-style, HPF-Net, Xu-type) is at most a target ε. The notebook then:

1. checks the guarantee numerically over 100 random calibration/test splits;
2. checks two exact results: with abstention, the risk of the images sent equals the calibrated risk divided by the probability of sending; discarding the smoothest fraction *q* of covers is detectable from a single image with AUC at most 1/2 + *q*/2;
3. measures the risk assigned by two independent wardens to the images actually sent;
4. runs a leave-one-detector-out test inside the sender's family to choose an internal calibration target, without using the wardens;
5. compares the payload of the rules at their corrected operating points.

## Main results (from `results/full_results.json`)

- All rules meet their guarantee against the sender's detectors at ε = 0.10, 0.20 and 0.30.
- The score-driven rule is flagged by the stronger warden on 54–56% of the images it sends at every ε. The leave-one-detector-out test finds no internal target for it, so it is declared non-transferable.
- After the leave-one-detector-out correction, the fixed and detectability-limited rules keep the worst-warden risk within ε.
- At the corrected operating points, the detectability-limited rule carries 0.176 vs. 0.100 bpp (ε = 0.20; paired gain 0.076, 95% interval [0.015, 0.113]) and 0.257 vs. 0.152 bpp (ε = 0.30; gain 0.105, interval [0.083, 0.128]) relative to the fixed payload, at nearly equal warden risk.
- At ε = 0.10, with a minimum payload of 0.05 bpp, the corrected detectability-limited rule is infeasible and the corrected fixed rule is feasible in 61 of 100 splits.

## Repository contents

| Path | Content |
|---|---|
| `notebook/RiskCalibratedStego.ipynb` | Complete pipeline: data preparation, embedding simulator, detector training, score lattices, calibration, all analyses, figures and tables |
| `results/full_results.json` | Every number reported above: configuration, detector AUC per payload, guarantee check, propositions, margins, transfer and paired comparisons |
| `figures/` | Detector AUC vs. payload, admission-detectability bound, warden risk without and with correction |

## Running the notebook

Requirements: Python 3.10+, PyTorch with a CUDA GPU, NumPy, pandas, Pillow, matplotlib.

1. Download BOSSbase 1.01 (`BOSSbase_1.01.zip`, 10,000 PGM images) and set `BOSS_DIR` to the folder that contains it, or let the notebook download it.
2. Optionally set `WORK_DIR` (large files: image cache, models, score lattices) and `RESULT_DIR` (small result files).
3. Run all cells. On one 20 GB GPU slice, training the five detectors took about 5 hours and the scoring and analyses about 15 minutes. Trained networks and score lattices are cached, so an interrupted run resumes where it stopped.

Set `SMOKE_TEST = True` in the configuration cell for a short end-to-end check on 160 images (results are meaningless at that size).

Seeds are fixed for the data split, network initialization, payload draws and re-splits. GPU non-determinism can change trained weights slightly between runs.

## Notes

- Embedding is simulated with optimal ternary change probabilities; no practical coding is performed.
- The detectability-limited rule uses a simplified local residual-variance estimate.
- In the reported run, the SRNet-style network was stopped at epoch 61 of its second training phase, and the checkpoint with the lowest smoothed validation loss (epoch 58) was used.

## Citation

Archived release: Hamam, H. (2026). *risk-calibrated-stego* (version 1.0.0). Zenodo. https://doi.org/10.5281/zenodo.24286822

## License

MIT (see `LICENSE`).
