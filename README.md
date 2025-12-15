# Unsupervised Clustering of Tubules in Kidney Biopsy Images

## Overview
Chronic kidney disease (CKD) affects approximately 10% of the global population and can progress to kidney failure, potentially requiring replacement therapy. Diagnosis relies on identifying lesions in kidney tissue and assessing structural deviations from normal morphology.

In digital nephropathology, whole-slide images (WSIs) of kidney biopsies enable machine learning approaches to support diagnostic and prognostic assessment. While abnormalities in tubules are crucial for accurate diagnosis, tubular lesions are less studied compared to other kidney structures.

This project aims to address this gap by developing an effective unsupervised clustering strategy for labelling tubular lesions, thereby laying the groundwork for an AI-assisted tool to support pathologists in tubular-lesion review and classification.

Read more about it in: [Unsupervised clustering of tubules in kidney biopsy images.pdf](Unsupervised%20clustering%20of%20tubules%20in%20kidney%20biopsy%20images.pdf)

---

## Dataset
The dataset consists of **809 manually annotated tubular structures**, classified into four categories:
- Normal
- Chronically damaged
- Acutely damaged
- Atrophic

Tubules labelled as *normal* are considered healthy, while all other categories are treated as *lesioned*.

---

## Tubule Extraction
tubule_extraction_code extracts tubules from WSIs using three different approaches, resulting in three separate datasets:
1. **Scaled patches**
2. **Masked and scaled patches**
3. **Masked and centred patches**, preserving the relative tubule size

These different representations are evaluated to determine which extraction strategy is most suitable for downstream clustering.

---

## Feature extraction
The feature_extraction_code script uses a pre-trained ResNet model to extract deep feature embeddings from tubular image patches. The extracted features are then used for unsupervised clustering and dimensionality reduction, enabling visualization and exploration of underlying structure and subgroups within the data.

---

## Project workflow

This repository is organised as a two-step pipeline:

1) **Tubule extraction** (creates the `data/` folder)  
2) **Feature extraction + clustering/visualization** (uses the content of `data/`)

---

### Folder structure

- `tubule_extraction/`  
  Contains the code and required input files for extracting tubule patches from whole-slide images.

- `data/` *(generated)*  
  Created by the extraction step. Holds exported patches and CSV files describing them. This folder is the input to the next step.

- `feature_extraction/`  
  Uses a ResNet-based feature extractor to compute embeddings from the generated patches, then performs dimensionality reduction and clustering.

---

### Step 1: Extract tubule patches

Run:
- `tubule_extraction/tubule_extraction_code.ipynb`

This notebook requires the following folders to be present inside `tubule_extraction/`:

#### Required input folders

##### `Annotations json/`
Contains tubule annotations exported from QuPath in JSON/GeoJSON format (one file per slide).

**Expected structure and required fields:**
- Top-level key: `features`
- For each feature/annotation:
  - `properties.classification.name` (tubule class label)
  - `geometry.type`
  - `geometry.coordinates` (polygon coordinates)

##### `images/`
Contains the corresponding whole-slide images (WSIs).

**Expected format and naming:**
- WSI files in `.svs`
- File name must match the annotation file base name, e.g.  
  `2016_220503_ANON.geojson` → `images/2016_220503_ANON.svs`

##### `target_image/`
Contains the stain normalization reference patch.

**Expected file:**
- `target_image/target_image.png`

This image is used as the target for stain normalization of extracted tubule patches.

#### Output
Running the notebook creates a `data/` folder in the project root containing extracted tubule patches and metadata (used in Step 2).

This notebook reads required resources from the `tubule_extraction/` folder (e.g. annotations and images) and generates the `data/` folder containing tubule patch datasets, for example:
- `data/tubule_patches/`
- `data/masked_patches/`
- `data/center_patches/`
- `data/masked_patches_center/`
and corresponding CSV files (e.g. `tubule_patches.csv`, `masked_patches.csv`, ...).

These outputs are needed for the next step.

---

### Step 2: Extract features + cluster

Run:
- `feature_extraction/feature_extraction_code.ipynb`

This notebook loads the patch datasets from `data/` and uses a (pre-trained) **ResNet** model to extract deep feature embeddings for each tubule patch.  
The extracted features are then used for:
- **Dimensionality reduction** (e.g. UMAP) for visualization  
- **Unsupervised clustering** (e.g. Leiden) to explore structure and potential subgroups

Supporting files:
- `feature_extraction/get_feature.py` (feature extraction utilities)
- `feature_extraction/ResNet.py` (ResNet model definition)
- `feature_extraction/best_ckpt.pth` (model checkpoint used for feature extraction, available from [Google Drive](https://drive.google.com/drive/folders/1AhstAFVqtTqxeS9WlBpU41BV08LYFUnL))

