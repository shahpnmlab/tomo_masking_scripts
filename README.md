# Segmentation Methods for Tomographic Data

This document provides comprehensive instructions for two different approaches to segmenting 3D tomographic data. Each method has different strengths and use cases, allowing you to choose the most appropriate technique for your specific data and requirements.

## Table of Contents

1. [Overview of Segmentation Methods](#overview-of-segmentation-methods)
2. [Method 1: IMOD-Based Segmentation with Binary Masks](#method-1-imod-based-segmentation-with-binary-masks)
3. [Method 2: Interactive Napari-Based Segmentation](#method-2-interactive-napari-based-segmentation)
4. [Choosing the Right Method](#choosing-the-right-method)
5. [Troubleshooting](#troubleshooting)
6. [Resources](#resources)

## Overview of Segmentation Methods

### Method 1: IMOD-Based Segmentation with Binary Masks
This method uses IMOD model files (.mod) to generate binary masks. It is ideal for well-defined structures where you can accurately trace contours in orthogonal planes.
It as inspired by the method published in this [paper](https://www.sciencedirect.com/science/article/pii/S2590152422000174)

**Key Features:**
- Uses predefined contours from IMOD software
- Generates masks from XY and XZ contours
- Non-interactive batch processing
- Suitable for consistent structures

### Method 2: Interactive Napari-Based Segmentation
This method provides an interactive GUI using Napari for direct point selection and real-time mask generation. It is well-suited for exploratory analysis and irregular structures.

**Key Features:**
- Interactive point selection
- Real-time visualization
- Spline-based contour fitting
- Smooth interpolation between slices
- Suitable for complex, irregular structures

## Method 1: IMOD-Based Segmentation with Binary Masks

### Prerequisites

- Python 3.x
- Required packages: typer, imodmodel, numpy, matplotlib, mrcfile, scipy
- IMOD model files (.mod) with:
  - Object ID 0: XY contour
  - Object ID 1: XZ contour
- Volume data (.mrc files)

### Installation

```bash
pip install typer imodmodel numpy matplotlib mrcfile scipy
```

### Script Source

[Download the IMOD Segmentation Script](https://gist.github.com/shahpnmlab/9c67b77b698575a313369f64cad01dc9)

### Directory Structure

```
project/
│
├── mod_files/         # Directory containing your .mod files
│
├── volume_data/       # Directory containing your .mrc volume files
│
└── masks/             # Output directory for generated masks
```

### Usage

```bash
python mod2mask.py --i /path/to/mod_files --o /path/to/output --v /path/to/volume_data
```

#### Required Arguments

- `--i`: Path to the directory containing .mod files
- `--o`: Path to save the output mask files (default: "./masks/")
- `--v`: Path to the directory containing volume data (.mrc files)

#### Optional Arguments

- `--verbosity`: Verbosity level (0: WARNING, 1: INFO, 2: DEBUG) (default: 1)

### How It Works

1. Reads volume information (shape and voxel size) from the first MRC file
2. For each .mod file:
   - Reads XY and XZ contours
   - Creates a 3D binary mask using:
     - XY contour for the shape in each slice
     - XZ contour for scaling across Z-slices
   - Saves the resulting mask as an MRC file

### Example Workflow

1. Create XY contours in IMOD by tracing the structure in the XY plane
2. Create XZ contours in IMOD by tracing the structure in the XZ plane
3. Save the .mod files with both contours
4. Run the script to generate masks from all .mod files
5. Verify the output masks align with the original data

## Method 2: Interactive Napari-Based Segmentation

### Prerequisites

- Python 3.x
- Required packages: numpy, napari, scipy, scikit-image, magicgui, qtpy, mrcfile, trimesh

### Installation

```bash
pip install numpy napari scipy scikit-image magicgui qtpy mrcfile trimesh
```

### Script Source

[Download the Napari Segmentation Script](https://gist.github.com/shahpnmlab/9c67b77b698575a313369f64cad01dc9)

### Usage

1. Run the script:
   ```bash
   python mesh2mask.py
   ```

2. The Napari viewer will open with a control panel on the right side.

### Interface Components

- **Open Tomogram**: File browser to select and load MRC files
- **Spline Smoothing**: Slider to adjust smoothness of contours (0.01-1.0)
- **Create 3D Mesh & Mask**: Button to generate 3D mask from selected points
- **Clear Points**: Button to remove all selected points
- **Save Mask**: Button to save the current mask as an MRC file
- **Apply Lowpass Filter**: Button to apply Gaussian smoothing to the tomogram

### Step-by-Step Guide

1. **Load Data**:
   - Click "Open Tomogram" and select your MRC file
   - The tomogram will be displayed in the main viewer

2. **Mark Points**:
   - Navigate to different Z-slices using the slider at the bottom
   - Click on the tomogram to mark points along the structure's boundary
   - Add points on multiple slices to define the 3D structure

3. **Adjust Smoothing**:
   - Use the "Spline Smoothing" slider to control contour smoothness
   - Lower values (closer to 0.01) create more precise contours
   - Higher values (closer to 1.0) create smoother contours

4. **Generate Mask**:
   - Click "Create 3D Mesh & Mask" after marking sufficient points
   - The script will:
     - Fit splines to points on each slice
     - Interpolate between slices
     - Apply 3D smoothing and hole filling
     - Generate and display the 3D mask and mesh

5. **Refine Results**:
   - Add more points if needed
   - Adjust smoothing and regenerate the mask
   - Use "Clear Points" to start over if necessary

6. **Save Results**:
   - Click "Save Mask" to save the mask as an MRC file
   - Files are saved in the "./results" directory

### Advanced Features

- **Automatic Interpolation**: The script smoothly interpolates between slices with marked points
- **3D Smoothing**: Applies Gaussian filtering to create smooth transitions
- **Component Analysis**: Ensures only the largest connected component is retained
- **Mesh Visualization**: Creates a 3D surface mesh for visualization

## Choosing the Right Method

### Use Method 1 (IMOD-Based) When:
- You already have IMOD contours from previous work
- You need batch processing of multiple structures
- Your structures have well-defined boundaries in orthogonal planes
- You prefer a non-interactive workflow

### Use Method 2 (Napari-Based) When:
- You want interactive, real-time feedback
- Your structures have complex, irregular shapes
- You need to explore the data while segmenting
- You prefer a graphical user interface

## Troubleshooting

### Method 1 (IMOD-Based)

**No MRC files found:**
- Ensure the volume directory contains .mrc files and the path is correct

**Missing or incorrect contours:**
- Verify that your .mod files contain both XY (object_id 0) and XZ (object_id 1) contours

**Empty masks:**
- Check if the contours are within the volume boundaries
- Verify that the contours are correctly defined

### Method 2 (Napari-Based)

**Nothing happens when clicking on the tomogram:**
- Ensure that the tomogram layer is active
- Check if the mouse drag callback is properly registered

**Spline fitting fails:**
- Add more points (minimum 4 points per slice)
- Try adjusting the smoothing parameter

**Mesh generation fails:**
- Check console for error messages
- Verify that enough points were marked across different slices
- Try adjusting the smoothing parameter

**Mask interpolation issues:**
- Mark points on more slices to improve interpolation
- Adjust the smoothing parameter

## Resources

- [IMOD Software](https://bio3d.colorado.edu/imod/)
- [Napari Documentation](https://napari.org/stable/)
- [MRC File Format Specification](https://www.ccpem.ac.uk/mrc_format/mrc2014.php)
- [Trimesh Documentation](https://trimsh.org/trimesh.html)
