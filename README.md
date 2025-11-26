# Head-Only Cutter — Google Colab Edition (2025)

Perfect Head Extraction: Face + Hair + Beard (No Neck / No Body)

## This notebook provides an end-to-end head extraction pipeline, designed to work in Google Colab with any type of photo:

- Full-body photos
- Half-body photos
- Portraits
- Side views
- Far or small faces

It automatically analyzes the photo, crops it properly, zooms the head, and uses a custom U-Net segmentation model to output a transparent head-only PNG.

## Notebook Logic Overview

This Colab notebook is built using seven logical stages, each visible inside the notebook and shown through images.

**1️. Upload Image**

- Upload any image directly from device storage
- The notebook loads & displays the image for confirmation

**Logic:**
User needs to visually verify the uploaded photo.
No processing happens yet.

**2️. Face Analysis (Full-Body vs. Portrait Detection)**

- Uses MediaPipe Face Detection to determine the head location and size.

**Logic:**

- If the head height is small (<25%) relative to the image height → assume full-body

- If the head is reasonably large → assume portrait / half-body

- This determines whether auto-portrait cropping is needed.

**3️. Auto Portrait Crop (If Full-Body)**

If the image is detected as full body:
- The notebook extracts the top 65% of the image
- This makes the head significantly bigger relative to the crop
- Ensures the segmentation model sees enough head detail
- The cropped portrait is shown for confirmation.

**Logic:**
Segmentation accuracy increases when the head occupies more of the frame.
This step prevents poor results from tiny heads.

**4️. Head Zoom Crop**

After portrait cropping, we do a tight head crop:
- Detect face again
- Compute center & bounding box
- Add 120% padding around the head (for hair/beard)
- Crop this area

**Logic:**
The segmentation model performs best when given a close-up head image, typically 300–500 px.
This zoom ensures consistent scaling regardless of input image type.

**5️. Run Segmentation Model (okaris/head-segmentation)**

Load custom U-Net–based segmentation model:

- ResNet encoder
- Multi-layer decoder
- Segmentation head
- Input resolution: 512×512
- Outputs 2-class segmentation mask
-   0 → background
-   1 → head

**Logic:**
This provides a raw head-only mask where white pixels = head.

The mask is shown for verification.

**6️. Reconstruct Full-Size Mask**

The head mask generated from the zoomed crop is:

- Resized back to its crop dimensions
- Pasted into a blank full-size mask based on the original image
- Everything aligns perfectly with no distortion

**Logic:**
We return segmentation to the original image coordinate space.

**7️. Create Final Head-Only PNG**

The notebook:

- Loads the original image
- Replaces its alpha channel with the full head mask
- All non-head pixels become transparent
- Saves the final image as head_only.png
- Displays the final output

Logic:
This produces a transparent PNG containing only:

- ✔ Face
- ✔ Hair
- ✔ Beard
- ❌ No shoulders
- ❌ No neck
- ❌ No background

## Colab Notebook Workflow Diagram
```txt
Upload Image
     ↓
Face Detection (MediaPipe)
     ↓
If Full-Body → Auto Portrait Crop
     ↓
Head Zoom Crop (tight + padded)
     ↓
Head Segmentation (U-Net model)
     ↓
Rebuild Full-Size Mask
     ↓
Apply Mask → Head-Only PNG
```

## Requirements (Automatically installed in Colab)

- mediapipe
- torch
- torchvision
- diffusers
- numpy
- opencv-python
- pillow

## Model Information

This notebook uses the custom segmentation U-Net from:
```txt
okaris/head-segmentation
```

## Key characteristics:

- ResNet-style encoder
- Multi-stage decoder
- ~100MB model checkpoint
- High-accuracy head extraction
- Trained specifically to extract:
- face
- hair (all styles)
- beard / facial hair


## Output Example (Stages Shown in Notebook)

- Original Image
- Auto Portrait
- Head Zoom
- Mask (binary)
- Final Image (PNG)
- Every step includes a plt.imshow() visual so the user can confirm processing.

💾 Saving Results

The following files are created:

File	Description
head_mask_crop.png	Mask of the zoomed head
full_mask.png	Mask aligned to original size
head_only.png	Transparent PNG with only the head
🧭 Why This Logic Works (Important)

This workflow solves two common problems:

Problem #1 — Full-body images produce terrible head segmentation

Solution: detect small head → crop to portrait → zoom head

Problem #2 — Hair & beard often get chopped off

Solution: 120% padding around head bounding box

Problem #3 — Models trained on portrait images fail on small heads

Solution: always resize to 512×512 after zooming

Problem #4 — Mask doesn’t align with original image

Solution: paste resized mask into full-size blank canvas at exact (x1, y1)

This ensures consistent, high-quality results for all photo types.

⭐ Recommended Usage

This notebook is ideal for:

AI avatars

Talking head animation

Video dubbing

Character extraction

Head swapping

Digital puppetry

Creative projects requiring head-only graphics
