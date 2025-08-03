# BraTS-3D-UNET

A simple 3D U-Net implementation for segmenting brain tumors in the BraTS MRI dataset.

## What’s in here?

- **3DUNET.py**  
  Defines the 3D U-Net architecture:  
  - A contracting “encoder” path (downsampling) to capture context  
  - An expanding “decoder” path (upsampling) to recover resolution  
  - Skip-connections between matching levels so fine details aren’t lost  

- **input.py**  
  Loads raw BraTS `.nii` MRI files (T1, T1c, T2, FLAIR) and stacks them into 4-channel volumes.

- **input processing.py**  
  Slices or patches those full volumes into smaller 3D cubes, applies any basic preprocessing (normalization, cropping, augmentation) so training fits in memory.

- **model.train.py**  
  Brings it all together:  
  1. Calls the data loader → gets batches of 3D patches + labels  
  2. Instantiates the U-Net from `3DUNET.py`  
  3. Defines loss (e.g. Dice + BCE) & optimizer  
  4. Loops over epochs, logs training/validation scores, saves the best model

## How it all flows

1. **Data loading** (`input.py`) →  
2. **Preprocessing & patching** (`input processing.py`) →  
3. **Model definition** (`3DUNET.py`) →  
4. **Training loop** (`model.train.py`) →  
5. **Saved checkpoint** ready for inference or further tuning
