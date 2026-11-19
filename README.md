# Beyond the Missing Pixels

**A comparative study of classical and learned image inpainting**

This project examines how three image-reconstruction methods respond to missing visual information: a compact U-Net, Telea inpainting, and Navier–Stokes inpainting. Experiments use the Oxford-IIIT Pet dataset with synthetic square masks. The goal is to characterize reconstruction accuracy and failure modes, not to claim that a lightweight learned method outperforms established baselines.

## Research questions

1. How does a small, trained U-Net compare with classical inpainting algorithms on masked-region pixel error?
2. How does reconstruction error change as the missing region grows?
3. What kinds of visual artifacts occur, and what does pixel error miss about perceptual or semantic quality?

## Methods

- **Dataset:** Oxford-IIIT Pet, resized to 128 × 128 RGB.
- **Development split:** 3,312 training images and 368 validation images from the `trainval` split.
- **Held-out test split:** 3,669 images.
- **Missing regions:** synthetic square masks. Random masks are 20–60 pixels wide for training and the main benchmark; a separate sensitivity experiment uses 20, 40, 60, and 80 pixels.
- **Learned model:** a compact encoder–decoder U-Net with skip connections, trained with masked L1 loss. Its input consists of damaged RGB channels plus the binary mask. Initial training used three epochs on a 500-image subset; the model then received one additional completed epoch on all 3,312 training images. Training ran in Google Colab.
- **Baselines:** OpenCV Telea (`cv2.INPAINT_TELEA`) and Navier–Stokes (`cv2.INPAINT_NS`), with inpainting radius 3.
- **Metric:** mean absolute error (MAE) over masked RGB pixels only, on the 0–1 color scale. **Lower is better.**

## Final held-out test results

| Method | Mean masked MAE | Std. dev. |
|---|---:|---:|
| **Telea** | **0.104550** | 0.047080 |
| Navier–Stokes | 0.105862 | 0.047653 |
| Improved U-Net | 0.122866 | 0.046257 |

All three methods were compared on the same 3,669 test images and synthetic masks (test random seed: `2026`). Telea yielded the lowest mean masked MAE. The U-Net had approximately **17.5% more error than Telea** on this test.

## Development findings

- On the 368-image validation benchmark, the original U-Net scored **0.144720** mean masked MAE and the improved U-Net scored **0.1214**, an approximately **16.14% relative reduction**. This measures an observed change between successive checkpoints rather than isolating a single causal factor.
- On that validation run, the improved U-Net beat Telea on **74/368** images and Navier–Stokes on **86/368** images, despite higher average error.
- A controlled **100-image mask-size sensitivity experiment** produced the following mean masked MAE:

| Square mask | U-Net | Telea | Navier–Stokes |
|---|---:|---:|---:|
| 20 px | 0.108426 | **0.087105** | 0.088091 |
| 40 px | 0.127230 | **0.112202** | 0.114480 |
| 60 px | 0.147616 | **0.136856** | 0.140172 |
| 80 px | 0.151534 | **0.143347** | 0.146082 |

Mean error rose with mask size for all methods. The U-Net's relative gap to Telea narrowed as masks grew, but Telea remained best at every tested size. The 80-pixel square lies outside the model's 20–60-pixel training mask range.

## Qualitative findings

The U-Net often filled missing areas with smooth or blurry color approximations, losing fine object detail. Classical inpainting could preserve nearby color and edge continuity better in some cases, but sometimes produced wedge-like or pinwheel distortions when extrapolating across large, semantically complex gaps. Neither approach reliably recovered obscured facial details in the examined examples. **Pixel-level accuracy is not equivalent to plausible semantic reconstruction.**

## Repository layout

```text
beyond-the-missing-pixels/
├── README.md
├── results_analysis.ipynb        # Runs on included aggregate results
├── experiment.ipynb              # Original training notebook: export from Colab
├── experiment_config.json       # Configuration reconstructed from experiment
├── figures/                     # Aggregate comparison plots
├── results/                     # Aggregate results reported in Colab
└── .gitignore
```

This starter package includes the aggregate analysis notebook, configuration, summary CSVs, and figures. **The original training/evaluation notebook is included as `experiment.ipynb`. Model weights, image-level CSVs, and the qualitative comparison figure are not yet included in this repository.** Model checkpoint files (`.pth`) are kept outside ordinary Git history unless using a suitable large-file storage mechanism.

## Reproducing the analysis

1. Run `results_analysis.ipynb` from the repository root to reproduce plots from the included summary CSVs. For full model reproduction, export and open your original `experiment.ipynb` in Google Colab.
2. Install or import the notebook's dependencies: Python, PyTorch, torchvision, NumPy, pandas, Matplotlib, OpenCV, and scikit-image (if used by the exploratory cells).
3. Load the Oxford-IIIT Pet data and reproduce the training/validation split using the notebook's recorded seed.
4. Run the classical baselines, U-Net training and evaluation cells in notebook order. For evaluation from a saved checkpoint, place the checkpoint in the path expected by the notebook, or edit the path.
5. Compare generated metrics with the committed results. Random seeds, data order, and library versions can affect exact replication.

**Reproducibility note:** The notebook was developed interactively, including runtime resets and checkpoint restoration. Before describing the repository as fully reproducible end-to-end, clean and rerun the notebook from a fresh runtime, and pin the package versions and saved mask specifications. The final test split was reserved for final evaluation after model selection.

## Limitations

- Only one compact neural architecture, a limited training budget, and two classical baselines were tested; this does **not** establish that traditional methods generally outperform deep learning.
- Input data and square-mask corruption are simplified relative to real-world restoration.
- Color quantization differs slightly between the floating-point U-Net and 8-bit OpenCV baselines.
- MAE does not measure perceptual realism, structural fidelity, or semantic accuracy.
- The mask-size experiment was conducted on validation images and is exploratory.

## Future work

Explore perceptual or structure-aware losses, longer training, richer inpainting architectures, uncertainty visualization, and human evaluation. A related proposed direction is **The Missing Thread**, investigating AI-assisted reconstruction of damaged traditional embroidery while distinguishing documented historical features from model-generated conjecture.

## Acknowledgments

Dataset: Oxford-IIIT Pet. Classical inpainting: OpenCV Telea and Navier–Stokes implementations. Neural modeling: PyTorch.
