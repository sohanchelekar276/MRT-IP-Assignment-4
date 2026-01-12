# MRT-IP-Assignment-4

# 🤖 Perception Subsystem Report

This document details the computer vision pipeline developed for the UMIC recruitment task. It covers the iterative problem-solving process for **Cone Detection** and the current development status of **Mallet Detection**.

---

## 🚧 Task 1: Traffic Cone Detection

### Objective
To robustly detect a traffic cone in an image, identify its boundaries, and reconstruct its geometric shape (triangle/trapezium) for navigation tasks, regardless of lighting conditions.

### 🧪 Methodologies & Troubleshooting

We iterated through four distinct approaches to solve the "Contrast Problem" where the cone's right edge disappeared against the white background.

#### Method 1: Edge-Based Detection (Canny + Hough)
* **Approach:** Standard pipeline: `Grayscale` → `Gaussian Blur` → `Canny Edge Detector` → `Probabilistic Hough Transform`.
* **Outcome:** Partial failure. The left edge (shadowed) was detected, but the right edge (light orange vs. white background) was invisible.
* **Root Cause:** In Grayscale intensity, the cone (~240) and background (~255) had insufficient contrast for the gradient thresholds.

#### Method 2: Threshold Tuning
* **Approach:** Lowering Canny hysteresis thresholds (e.g., `50, 100`) to force edge detection.
* **Outcome:** Failure. While the edge appeared, it introduced unmanageable noise from the floor texture, making the data unusable for navigation.

#### Method 3: Geometric Reconstruction (Convex Hull)
* **Approach:** Using `cv2.convexHull` on the raw Canny output to "close" the missing shape mathematically.
* **Outcome:** Failure. The hull wrapped around the **image borders** (the frame box) rather than the cone, as Canny detected the sharp image boundary.
* **Visual Evidence:**
    ![Geometric Failure - Box Detection](path/to/image_291688.jpg)

#### ✅ Final Solution: HSV Color Segmentation & Polygon Fitting
* **Approach:** Switched from Edge Detection to **Color Blob Detection**.
    1.  Convert image to **HSV Color Space**.
    2.  Mask specific Orange/Red hues (High Saturation vs. Low Saturation background).
    3.  Find the largest contour and apply `cv2.approxPolyDP`.
* **Why it Works:**
    * **Solves Contrast:** Differentiates objects by *Saturation* (Colorfulness) rather than *Brightness*.
    * **Ignores Artifacts:** Image borders are black/white (not orange), so they are automatically excluded.
    * **Geometry:** `approxPolyDP` reconstructs a clean 3-point triangle or 4-point trapezium even if edges are slightly blurry.

---

## 🔨 Task 2: Mallet (Hammer) Detection

### Objective
To detect the mallet, determine its **Orientation** (Handle vs. Head), and calculate the optimal **Grip Point** for the SpiderBot end-effector.

### 🔄 Current Status: Work in Progress

#### Method 1: Color-Based Segmentation
* **Status:** **Implemented**
* **Approach:** Using HSV masking to isolate the mallet from the floor.
* **Current Challenge:**
    * **Occlusion:** The robot's legs and shadows occasionally split the mallet into two separate contours.
    * **Orientation Ambiguity:** A simple bounding box does not tell us which end is the handle and which is the head.

#### Method 2: Orientation via PCA (Planned)
* **Status:** **In Development**
* **Strategy:** Apply **Principal Component Analysis (PCA)** on the segmented blob.
    * **Primary Eigenvector:** Determines the angle of the handle (rotation).
    * **Secondary Eigenvector:** Determines the width.
    * **Head Logic:** Calculate pixel density along the primary axis to identify the "heavier" end (the head).

#### Method 3: Template Matching (Fallback)
* **Status:** **Research Phase**
* **Strategy:** If lighting conditions make color segmentation unreliable, we will implement `cv2.matchTemplate` with rotated templates to locate the object.

### 📋 Summary Table

| Task | Status | Priority | Key Blocker |
| :--- | :--- | :--- | :--- |
| **Cone Detection** | ✅ **Complete** | - | - |
| **Mallet Segmentation** | 🟡 **Partial** | High | Occlusion splitting contours |
| **Mallet Orientation** | 🟠 **Planned** | High | Implementing PCA logic |
| **Grip Calculation** | 🔴 **Pending** | Medium | Depends on Orientation |
