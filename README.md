# Aura-Core: Autonomous Biometric Computational Engine

![Version](https://img.shields.io/badge/Aura%20Core-v3.0.0-blue?style=flat-square)
![Build Status](https://img.shields.io/badge/build-passing-brightgreen?style=flat-square)
![Python](https://img.shields.io/badge/Python-3.9%20|%203.10-blue?style=flat-square)
![Architecture](https://img.shields.io/badge/Architecture-Event%20Driven%20Microservices-orange?style=flat-square)
![Compliance](https://img.shields.io/badge/Compliance-HIPAA%20|%20GDPR%20Aligned-red?style=flat-square)
![Code Style](https://img.shields.io/badge/code%20style-black-000000.svg?style=flat-square)

> **A high-throughput, asynchronous Digital Signal Processing (DSP) framework for non-invasive physiological estimation using computer vision and spectro-temporal audio analysis.**

---

## 📑 Table of Contents
1. [Abstract](#-abstract)
2. [Theoretical Framework](#-theoretical-framework)
3. [System Architecture](#-system-architecture)
4. [Algorithmic Methodology](#-algorithmic-methodology)
5. [Performance Benchmarks](#-performance--benchmarks)
6. [API Specification](#-api-specification)
7. [Deployment](#-deployment)
8. [References](#-references)

---

## 🔬 Abstract

**Aura-Core** serves as the backend intelligence for the Aura ecosystem. It is designed to address the challenges of **Remote Photoplethysmography (rPPG)**—the extraction of cardiac signals from standard video feeds—in real-world environments. 

Unlike traditional synchronous backends, Aura-Core utilizes a **non-blocking, event-driven architecture** powered by FastAPI and Celery. It orchestrates complex computer vision pipelines and audio spectral analysis to detect physiological biomarkers (Heart Rate, SpO2, Respiratory Distress) with clinical-grade latency (< 200ms per frame batch). The system is architected to be **stateless**, ensuring strict data privacy by processing biometric data in volatile memory without persistent storage.

---

## 📐 Theoretical Framework

### 1. Optical Cardiac Estimation (rPPG)
The core vision module relies on the principle that hemoglobin absorbs light differently than surrounding tissue. As the heart pumps, the blood volume in facial vessels changes, causing minute color variations.

We implement the **Plane-Orthogonal-to-Skin (POS)** algorithm, which projects RGB signals onto a plane orthogonal to the skin tone vector to eliminate motion artifacts.

**Mathematical Model:**
Given the normalized color channels $C_n(t)$, the projection matrix $P$ defines the signal $S(t)$:
$$S(t) = C_1(t) + \alpha \cdot C_2(t)$$
Where $\alpha$ is the tuning parameter derived from the standard deviation of the projection signals:
$$\alpha = \frac{\sigma(C_1)}{\sigma(C_2)}$$

### 2. Acoustic Respiratory Analysis
The audio module analyzes forced cough and speech samples to identify spectral signatures associated with respiratory pathology.
* **Feature Extraction:** We compute **Mel-frequency Cepstral Coefficients (MFCCs)** to represent the short-term power spectrum of the sound.
* **Dimensionality Reduction:** High-dimensional audio vectors are compressed using **Principal Component Analysis (PCA)** to maximize variance retention while minimizing inference time.

---

## 🏗 System Architecture

Aura-Core decouples the *Interface Layer* (API) from the *Compute Layer* (Workers) to ensure high availability under load.

```mermaid
graph TD
    subgraph "Ingestion Layer"
    Client[Client Application] -->|WSS / HTTPS| Gateway[API Gateway]
    Gateway -->|Load Balanced| API[FastAPI Orchestrator]
    end

    subgraph "Message Broker"
    API -->|Enqueue Job| Redis[Redis Queue]
    end

    subgraph "Compute Layer (Scalable Workers)"
    Redis -->|Pop| VisionWorker[rPPG Worker Node]
    Redis -->|Pop| AudioWorker[Audio DSP Worker Node]
    
    VisionWorker -->|CV Pipeline| MediaPipe[Face Mesh & ROI]
    MediaPipe -->|Signal| POS[POS Algorithm]
    
    AudioWorker -->|Waveform| Librosa[Librosa Backend]
    Librosa -->|Features| PCA[PCA Transformer]
    end

    subgraph "Data & Analytics"
    POS & PCA -->|Results| Aggregator[Risk Engine]
    Aggregator -->|JSON| DB[(Timeseries DB)]
    end
