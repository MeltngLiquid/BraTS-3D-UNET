# BraTS-3D-UNET

This repository implements a full 3D U-Net pipeline for brain tumor segmentation using the BraTS dataset. The goal is to segment different tumor regions from multi-modal MRI scans (T1, T1c, T2, FLAIR) using a volumetric deep learning model.

---

## Project Structure

```
.
├── 3DUNET.py             # Defines the 3D U-Net model architecture
├── input.py              # Loads and stacks MRI volumes for each patient
├── input processing.py   # Converts volumes into smaller patches and normalizes them
└── model.train.py        # Trains the model and handles loss, validation, checkpointing
```

---

## 1. Loading MRI Volumes (`input.py`)

* **Input**: Four `.nii` files for each patient: T1, T1-contrast (T1c), T2, and FLAIR
* **Process**:

  * Load each file using `nibabel` or `SimpleITK`
  * Convert to NumPy arrays of shape `(D, H, W)`
  * Stack them to form a single array of shape `(4, D, H, W)`
* **Output**: One 4-channel 3D volume per patient

---

## 2. Patch Extraction & Preprocessing (`input processing.py`)

* **Input**: The stacked 4-channel volume and its corresponding label mask
* **Process**:

  * Extract 3D patches (e.g. 64x64x64) from the full volume
  * Apply preprocessing: zero-mean, unit-variance normalization per channel
  * Optionally augment patches with flips/rotations
* **Output**:

  * Input patches: shape `(B, 4, p, p, p)`
  * Label patches: shape `(B, 1, p, p, p)`

---

## 3. 3D U-Net Architecture (`3DUNET.py`)

* **Structure**:

  * **Encoder Path**: Downsamples the input using repeated Conv3D + ReLU + MaxPool3D blocks
  * **Decoder Path**: Upsamples the features and uses skip connections to combine them with encoder features
  * **Final Layer**: A 1x1x1 Conv3D layer that maps to the number of segmentation classes

* **Skip Connections**: Preserve spatial information across the network for accurate segmentation

---

## 4. Model Training (`model.train.py`)

* **Input**: Patches and labels from preprocessing

* **Process**:

  * Initialize the U-Net model
  * Define the loss function (Dice Loss + Cross Entropy)
  * Use Adam or SGD optimizer
  * Train the model over several epochs
  * Evaluate on a validation set after each epoch
  * Save the model with the best validation performance

* **Output**: Trained model weights (checkpoint file)

---

## 5. Inference and Segmentation

* **Input**: Full test volume and trained model checkpoint

* **Process**:

  * Extract overlapping patches from the test volume
  * Predict segmentation for each patch
  * Reconstruct full volume by averaging overlapping regions
  * Post-process (e.g. thresholding, removing small blobs)

* **Output**: Final segmentation mask for the full volume

---

## Inputs & Outputs Summary

| Stage            | Input                               | Output                                          |
| ---------------- | ----------------------------------- | ----------------------------------------------- |
| Data Loading     | T1, T1c, T2, FLAIR `.nii` files     | 4-channel NumPy array `(4, D, H, W)`            |
| Patch Extraction | Full volume and label               | Patches `(B, 4, p, p, p)` and `(B, 1, p, p, p)` |
| Model Definition | -                                   | U-Net model instance                            |
| Training         | Patches + labels                    | Trained weights (`.pth` or `.h5`)               |
| Inference        | Full test volume + model checkpoint | Final segmentation mask `(D, H, W)`             |

---

