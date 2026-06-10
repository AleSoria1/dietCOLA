# Displacement & Instantaneous Edge Tracking for COLA
# dietCoLA: Displaced Element Tracking for Cortical Laser Ablation

## Overview
**dietCoLA** is a machine learning pipeline designed to automate the extraction of recoil dynamics following cortical laser ablation. 

Quantifying mechanical forces during morphogenesis is essential for understanding tissue organization, but direct force measurements remain experimentally demanding. Cortical laser ablation is a powerful technique that infers cortical tension from recoil dynamics; however, tracking this recoil is often done manually, which limits throughput and introduces subjectivity. dietCoLA aims to solve this bottleneck by using a vision-based machine learning approach to track cortical motion objectively and reproducibly.

## Key Features & Benchmarking
The dietCoLA model was trained on synthetic data and benchmarked against two established tracking methods—Particle Image Velocimetry (PIV) and Trackpy—using a dataset of 181 manually annotated cortical laser ablation experiments in *C. elegans* embryos.

* **dietCoLA**: Successfully captures the velocity directionality and relative magnitude changes between frames without requiring manual parameter adjustments. While it currently overestimates initial recoil velocities and subsequent decay rates compared to annotated data, it significantly outperforms PIV.
* **Trackpy**: Identified as the most reliable automated method evaluated, closely reproducing manual trajectories and capturing the fast initial movement immediately after ablation. However, it requires extensive parameter tuning and relies on high-contrast tracking points.
* **PIV**: Failed to capture the expected symmetric recoil in the noisy, low-texture cortical images, producing imprecise displacement vectors.

## Methodology Pipeline
Due to the limited size of the dataset, dietCoLA relies on synthetically generated training data that models an actomyosin cortex and its dynamic response to ablation. 

### 1. Synthetic Data Generation
* **Synthetic Cortex:** An actomyosin cortex texture is generated using procedural noise techniques (simplex and cellular noise) to model actin and myosin components, which are then restricted by an elliptical mask representing the cell cortex.
* **Ablation Model:** The laser cut is modeled using a time-evolving signed distance function (SDF) shaped as a capsule. 
* **Velocity Field Calculation:** The velocity field is modeled by an exponential prior based on the distance to the boundary: $v(x,y,t)=v_{\Omega}(t)e^{-d(x,y,t)/\lambda}$.

### 2. Advection Simulation
* The synthetic actomyosin cortex is modeled as a scalar field $\phi(x,t)$. 
* Its transport by the velocity field is governed by the linear advection equation to enforce conservation: $\frac{\partial\phi}{\partial t}+v\cdot\nabla\phi=0$. 
* This is discretized using a semi-Lagrangian advection scheme for numerical stability.

### 3. U-Net Machine Learning Architecture
* **Model Input/Output:** A U-Net convolutional neural network with residual connections accepts consecutive, normalized, grayscale frame pairs $(t, t+1)$ and predicts the two-channel directional representation of the velocity field $(v_x, v_y)$.
* **Training & Loss:** Training is supervised using the synthetic data. The loss function minimizes mean squared error (MSE), applies a smoothness loss to penalize sharp features, and utilizes a divergence loss to encourage incompressible flow $(\nabla\cdot v=0)$.

### 4. Recoil Kinetics Analysis
* Velocity vectors are decomposed into components parallel $(v_{||})$ and perpendicular $(v_{\perp})$ to the cut axis.
* Time-dependent profiles of $v_{\perp}$ are fitted to an exponential decay model to extract the initial recoil velocity $v_0$ and decay time $\tau$: $v(t)=v_{0}e^{-t/\tau}$.

## Authors and Acknowledgments
**Authors:** Nicolò Casonato, Matthias Kovačić, Signe Nynäs, Alejandro Soria Martinez.
**Supervisors:** Casper van Bavel and Trifone Pastore (Lab for Predictive Genetics and Multicellular Systems, KU Leuven).

See for reference: Vanslambrouck M, Thiels W, Vangheel J, van Bavel C, Smeets B, Jelier R (2024) Image-based force inference by biomechanical simulation. PLoS Comput Biol 20(12): e1012629. https://doi.org/10.1371/journal.pcbi.1012629

