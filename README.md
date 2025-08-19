# Encoding Models and Representational Similarity Analysis of Visual Cortex (Algonauts 2023 / NSD)


This project analyzes high-level visual cortex responses from the **Algonauts Challenge dataset**, 
a subset of the **Natural Scenes Dataset (NSD)** (Allen et al., 2022).  
We implement **encoding models** and **representational similarity analysis (RSA)** to compare 
fMRI brain responses with state-of-the-art vision models (e.g., CLIP, ResNet).

---


## Dataset (Algonauts 2023 / NSD)

**Source**  
The Algonauts challenge data comes from the **Natural Scenes Dataset (NSD)** (Allen et al., 2022): high-quality 7T fMRI from **8 subjects** viewing natural images (COCO; Lin et al., 2014).

**Stimuli & design**
- Each subject viewed **10,000 distinct images**; **1,000** were shared across all subjects (8 × 9,000 unique + 1,000 shared = **73,000** total unique images).
- Each image was shown **3×** ⇒ **30,000 image trials** per subject.
- Task: **continuous recognition** while maintaining fixation (report if the current image was seen before).

**Sessions & splits**
- Data were collected over ~**40 sessions** per subject (not all completed every session).
- For Algonauts 2023, the **last 3 sessions** per subject are **withheld** and define the **test split** (test images only; **no test fMRI**). The remaining sessions form the **training split**.

**Modality & preprocessing**
- fMRI responses are **preprocessed BOLD amplitudes**, **projected to cortical surface** (FreeSurfer **fsaverage**).
- The challenge distributes a **subset of visual-cortex vertices** (LH/RH provided separately) that were maximally visually responsive.
- **Training fMRI** are **z-scored within session** and **averaged across repeats**.
- We analyze **surface vertices** (not volume voxels) and build **bilateral** ROIs by concatenating LH/RH.

> **ROIs used here:** EBA, FFA (FFA-1/2), PPA (from provided challenge-space labels/mappings).

---

## Analysis methods

1. **ROI extraction**
   - Load Algonauts cortical-surface matrices and label maps; build **bilateral** EBA/FFA/PPA vertex matrices per subject.
   - Persist per-ROI arrays and per-subject image metadata (NSD IDs, file names).

2. **Feature extraction**
   - Vision models: **CLIP ViT-B/32** (LAION) and **ResNet-50** (ImageNet).
   - Apply model-specific preprocessing and **L2 normalization**.
   - Features are cached to avoid recomputation (GPU/AMP supported).

3. **Encoding (vertex-wise ridge regression)**
   - **Nested cross-validation**: inner loop selects α (log-grid), outer loop estimates generalization.
   - Metrics: per-vertex **R²**; ROI summaries reported as **median**, **mean**, and **top-10% mean** R².

4. **RSA (representational geometry)**
   - RDMs: **correlation distance** on fMRI (responses column-zscored per session, then averaged across repeats), **cosine distance** on model features.
   - To manage O(n²) scaling, a **fixed random subset of 600 images per subject** is used (identical across models for comparability).
   - Similarity: **Spearman ρ** between upper triangles of model and fMRI RDMs.
   - Significance: **permutation test** by shuffling feature RDM entries (**N=1000 by default, configurable**).

5. **Aggregation & visualization**
   - Group-level plots: ROI × subject heatmaps, bar plots with bootstrap **95% CIs**, and **R² vs ρ** scatter.
   - Model comparison (CLIP vs ResNet): paired **Wilcoxon tests**, **BH-FDR correction**, and **rank-biserial effect sizes**.

> **Note:** All training fMRI are **surface-projected vertex responses**. Data are **z-scored within session** and **averaged across repeats** before model fitting.

---

## Repository layout
```
algonauts_nsd_encoding_rsa/
│
├── notebooks/
│ ├── 1. data_prep_roi_extract.ipynb # extract ROI data & prepare voxel/vertex responses
│ ├── 2. encoding_rsa.ipynb # run nested ridge encoding + RSA per ROI (with randomization options)
│ ├── 3.group-aggregation_viz.ipynb # aggregate subjects & ROIs, group-level stats and visualization
│ └── 4. compare_models.ipynb # model comparisons (CLIP vs ResNet, etc.)
│
├── algonauts outputs/
│ ├── multiROI/
│ │ ├── EBA/ FFA/ PPA/meta # per-ROI outputs (json, npy, plots)
│ │ ├── subj01.npy ... subj08.npy # per-subject arrays
│ │
│ ├── group_clip/ # group-level CLIP analysis
│ │ ├── plots/ # figures (RSA heatmaps, barplots, etc.)
│ │ └── group_summary_day3.csv # group summary stats
│ │
│ ├── group_resnet/ # group-level ResNet analysis
│ │ └── plots/ # figures
│ │
│ └── group_compare/ # CLIP vs ResNet deltas, stats, figs
│ └── plots/
│
├── README.md # project description and instructions
└── requirements.txt # Python dependencies
```

---

## Requirements

Install dependencies with:

```bash
pip install -r requirements.txt

```
---

## Citation

- Gifford AT, Lahner B, Saba-Sadiya S, Vilas MG, Lascelles A, Oliva A, Kay K, Roig G, Cichy RM. 2023. The Algonauts Project 2023 Challenge: How the Human Brain Makes Sense of Natural Scenes. arXiv preprint, arXiv:2301.03198. DOI: https://doi.org/10.48550/arXiv.2301.03198 
- Allen EJ, St-Yves G, Wu Y, Breedlove JL, Prince JS, Dowdle LT, Nau M, Caron B, Pestilli F, Charest I, Hutchinson JB, Naselaris T, Kay K. 2022. A massive 7T fMRI dataset to bridge cognitive neuroscience and computational intelligence. Nature Neuroscience, 25(1):116–126. DOI: https://doi.org/10.1038/s41593-021-00962-x 

---

## Contact
- Amit Yelin — amityelin@gmail.com  
- Iking Jose Enrique Lopes — ikinglopez1@gmail.com

