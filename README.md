# Master's M2 Project Report : Learning Delay-Coupled Nonlinear Dynamics from fMRI Data

This repository contains the final report and documentation for my Master's project, validated as part of a dual-degree program:
*   **M2 Engineering** at CentraleSupélec
*   **M2 Computational Neuroscience and Neuroengineering (CNN)** at Paris-Saclay University

*This project was conducted in collaboration with Dr. Alain Destexhe's laboratory (NeuroPSI) and supervised by Pablo Castro (NeuroSpin) and Alain Destexhe.*
**Evaluation:** Grade A+

> **Note on Source Code:** The computational models and data pipelines were executed on the laboratory's proprietary high-performance clusters (Data Center for Education at CentraleSupélec). Due to data volume and infrastructural constraints, the raw code is not hosted in this public repository but is thoroughly detailed in the attached report.

## Project Objective
Modeling the spatiotemporal dynamics of functional Magnetic Resonance Imaging (fMRI) is a major challenge due to signal non-stationarity and complex non-linear interactions. This project evaluates the capacity of the Koopman operator framework to identify governing equations from simulated BOLD signals. 

## Methodology & Theoretical Framework
*   **Data Simulation:** Time series were generated using *TheVirtualBrain (TVB)* platform, employing a semi-analytical Adaptive Exponential Integrate-and-Fire (AdEx) mean-field formalism. This provided a controlled environment to study brain states under fixed biophysical parameters.
*   **Dynamical Systems Modeling:** The approach relies on **Koopman Theory**, which shifts the analysis of non-linear dynamical systems from a finite-dimensional state space to an infinite-dimensional function space where the temporal evolution can be linearly approximated.

## Deep Learning Architecture: The Koopa Model
The project utilizes the **Koopa** (Koopman Forecasting with Purely Attentionless Architecture) model to discover these linear embeddings via deep learning (Yong Liu et al. "[Koopa: Learning Non-stationary Time Series Dynamics with Koopman Predictors](https://arxiv.org/abs/2305.18803)". In: *Advances in Neural Information Processing Systems (NeurIPS)*. 2023).

### Model Adaptations
To handle the specific physical and statistical properties of simulated BOLD signals, several modifications were made to the core architecture:
1.  **Strict Data Isolation:** The Dataloader was restructured to prevent overlapping sliding windows between distinct TVB simulations, effectively eliminating data leakage.
2.  **Reversible Instance Normalization (RevIN):** Implemented to address the extreme amplitude heterogeneity and near-zero variance of simulated signals. This instance-wise normalization prevents the model from converging toward trivial zero-output predictions.
3.  **MSSE Loss Function:** Optimization was shifted to Mean Squared Scaled Error (MSSE) to ensure errors were weighted fairly across simulations regardless of their absolute amplitude.

### Architecture Schematic
```text
Input BOLD Window (Length L)
     │
     ▼
[ RevIN Layer ] ──────────── (Extracts Shift β, Scale γ) ───────────────────────────────────────────────────┐
(Normalization)                                                                                             │
     │                                                                                                      │
     ▼                                                                                                      │
[ Koopa Block 1 ] ──► ... ──► [ Koopa Block M ]                                                             │
     │                                                                                                      │
     │ (Spectral Separation)                                                                                │
     │   ├─ Invariant Branch: Encoder (MLP) -> K_invar (neural network) -> Decoder (MLP)                    │
     │   └─ Variant Branch:   Encoder (MLP) -> K_var (fitted using least-squares method)  -> Decoder (MLP)  │
     │                                                                                                      │
     ▼                                                                                                      │
[ Total Block Summation ]                                                                                   │
     │                                                                                                      │
     ▼                                                                                                      │
[ RevIN Layer ] ◄─────────── (Recalls Shift β, Scale γ) ────────────────────────────────────────────────────┘
(Denormalization)
     │
     ▼
Forecasted BOLD Series (Length H)
```

## Results & Conclusions

The model was optimized via Bayesian hyperparameter tuning and evaluated on isolated test sets.

*   **Quantitative Performance:** The architecture demonstrated robust predictive capabilities, achieving a Mean Directional Accuracy (MDA) of 69.48% and a Mean Pearson Correlation of 0.51. It successfully captured the macroscopic directional envelope of BOLD fluctuations with a very low reconstruction error (MAE: 1.27e-4).
*   **Critical Analysis of the Operator:** Internal variance analysis revealed that the time-variant branch absorbed over 83% of the signal variance, while the invariant Koopman operator primarily specialized in extracting a global baseline signal rather than localized functional networks.
*   **Conclusion:** While the Koopman framework serves as a mathematically coherent structural foundation, the predictive performance on these specific simulated BOLD signals is primarily driven by the representation capacity of the deep non-linear mappings (MLP encoder-decoder) rather than the theoretical linear constraints.

---
**For a comprehensive analysis of the mathematical frameworks, the hyperparameter optimization process, and visual signal reconstructions, please refer to the full PDF report included in this repository.**
