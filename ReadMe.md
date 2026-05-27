<div align="center">
  <img src="https://www.ru.nl/sites/default/files/styles/content_full/public/2023-10/RU_drupal_logo_RU_0.png.webp?itok=m6D0pAxp" alt="Radboud University Logo" style="height: 110px; width: auto; margin-bottom: 20px;" />
  <h1 style="margin-top: 0;">Retinal Blood Vessel Segmentation</h1>
  <p><em>Automated image analysis pipeline optimized using Bayesian Optimization.</em></p>
  <p>
    Course: <strong>NWI-IBC046 Image Analysis</strong><br>
    Author: <strong>Simon Wagener</strong>
  </p>
  
  [![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
  [![Jupyter Notebook](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
</div>

---

## Introduction
Retinal blood vessel segmentation is a critical preprocessing step in the analysis of fundus images for the extraction of vascular features such as vessel length, width, tortuosity, branching structure, and bifurcation angles. These features serve as important biomarkers for diagnosing a variety of ocular and systemic conditions, including diabetic retinopathy, hypertensive retinopathy, and retinal artery occlusion. 

Traditionally, the segmentation of blood vessels in retinal images is carried out manually by trained experts, a process that is time-consuming, labor-intensive, and subject to inter-observer variability. Consequently, there is significant interest in developing automated methods for retinal vessel segmentation that can provide accurate and consistent results with minimal human intervention. Finally, the segmentation algorithm is assessed on how well it performs compared to manually segmented blood vessels.

## Approach
The proposed approach utilizes a dataset comprising 20 retinal images, each accompanied by a corresponding circular mask delineating the region of interest, and 20 ground truth binary vessel segmentation images annotated manually (the DRIVE dataset). 

The goal is to design a modular image processing pipeline whose parameters can be optimized using Bayesian optimization. This approach allows for flexible adaptation to different performance metrics, such as the area under the receiver operating characteristic curve (AUC-ROC), the Dice similarity coefficient, or pixel-wise classification accuracy.

<br>

<div align="center">
  <img src="https://raw.githubusercontent.com/Dimon821/image-analysis-of-retinal-blood-vessels/refs/heads/main/img/overview.jpg" alt="Schematic overview of the processing pipeline" style="width: 100%; max-width: 1000px; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">
  <br>
  <span style="color: #555; font-style: italic;">Schematic overview of the processing pipeline.</span>
</div>

<br>

## Image Processing Pipeline
The proposed image processing pipeline for retinal blood vessel segmentation follows a strict sequential process:

1. **Green Channel Extraction:** The pipeline begins with the extraction of the green channel from each RGB fundus image, as blood vessels exhibit the highest contrast against the retinal background in this channel. 
2. **Contrast Stretching:** Contrast stretching is applied to expand the dynamic range of pixel intensities, improving the distinction between vascular and non-vascular regions.
3. **Background Homogenization:** To mitigate uneven illumination, a Gaussian filter is used to estimate the background illumination, which is subtracted from the contrast-enhanced image. The result is then inverted, enhancing vessel-like structures.
4. **Edge Detection:** An algorithm such as the Canny edge detector is used to capture strong intensity gradients typically associated with blood vessel boundaries.
5. **FOV Masking:** To eliminate false positives at the periphery of the image caused by the outer mask, the circular boundary of the retina is identified using a Hough transform. All pixels outside this region are suppressed.
6. **Noise Filtering:** A non-linear median filter is applied to remove isolated noise pixels and small artifacts without significantly affecting vessel continuity. 
7. **Morphological Cleanup:** Morphological closing is performed to reconnect any fragmented vessel segments, enhancing the continuity of the vascular network.
8. **Connected Component Analysis:** A filtering step retains only the largest connected component, assumed to correspond to the primary vessel structure. 
9. **Artifact Elimination:** Bright circular artifacts (camera reflections) are detected through a secondary Hough transform combined with edge detection. Their radii are expanded, and these ring-shaped regions are removed.
10. **Final Reconstruction:** A second morphological closing operation is applied, retaining the largest connected component. The central circular region is then reintroduced to maintain anatomical integrity.

## Bayesian Optimization
To optimize the performance of the retinal vessel segmentation pipeline, a Bayesian optimization framework was employed using Gaussian Process-based minimization. The objective was to maximize the segmentation quality as measured by the Area Under the Receiver Operating Characteristic Curve (AUC-ROC), which evaluates the alignment between the predicted vessel maps and the manually segmented ground truth images.

The optimization process targets a set of **15 parameters** that control various aspects of the pipeline, including:
* Gaussian smoothing scales
* Edge detection thresholds
* Morphological filtering settings
* Geometric constraints for masking and artifact removal

The optimization explores the multi-dimensional search space by executing the pipeline on the training dataset. Predicted masks are flattened and compared against ground truth using `roc_auc_score`. A negative AUC is returned as the objective value to conform to the minimization strategy.

To prevent excessive evaluations and overfitting, an early stopping mechanism is integrated via a custom `EarlyStopper` class, halting the search if no improvement in AUC is observed over a predefined patience window. After up to 50 iterations, the optimal parameter set yielding the highest AUC is reported and used for final evaluation.

---

## Getting Started

### Prerequisites
To run the notebook, ensure you have the following Python libraries installed:
```bash
pip install numpy pandas scipy opencv-python pillow scikit-image scikit-learn scikit-optimize matplotlib
