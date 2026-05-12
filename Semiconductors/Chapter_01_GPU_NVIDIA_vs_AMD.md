# Chapter 01: GPUs — NVIDIA vs AMD

## What Is a GPU?

A **GPU (Graphics Processing Unit)** was originally designed to render pixels on a screen — a massively parallel task. Rendering a 4K frame means computing the color of 8 million pixels simultaneously, every 16ms. That requires a fundamentally different architecture than a CPU.

### CPU vs GPU: A Mental Model

| Aspect | CPU | GPU |
|--------|-----|-----|
| Cores | 8–128 powerful cores | Thousands of simpler cores |
| Design goal | Low latency, complex tasks | High throughput, parallel tasks |
| Memory | ~64 MB L3 cache | 24–80 GB VRAM (GPU-local) |
| Analogy | A few expert surgeons | An army of factory workers |
| Best for | Serial code, OS, apps | Rendering, AI training, simulation |

```mermaid
flowchart LR
    subgraph CPU["CPU (e.g. Intel Core i9)"]
        direction TB
        C1[Core 1\nComplex] 
        C2[Core 2\nComplex]
        C3[Core 3\nComplex]
        C4[... 24 cores]
        Cache[Large shared\nL3 Cache]
        C1 & C2 & C3 & C4 --> Cache
    end

    subgraph GPU["GPU (e.g. NVIDIA RTX 4090)"]
        direction TB
        SM1[SM 1\n128 CUDA cores]
        SM2[SM 2\n128 CUDA cores]
        SM3[SM 3\n128 CUDA cores]
        SMN[... 128 SMs\n= 16,384 cores]
        VRAM[24 GB GDDR6X\nVRAM]
        SM1 & SM2 & SM3 & SMN --> VRAM
    end
```

---

## NVIDIA Architecture

NVIDIA names its GPU microarchitectures after scientists:

```mermaid
timeline
    title NVIDIA GPU Architecture Generations
    2016 : Pascal (GP100/GP102)
         : GTX 1080, Tesla P100
         : 16nm TSMC
    2017 : Volta (GV100)
         : Tesla V100
         : Introduced Tensor Cores
         : 12nm TSMC
    2018 : Turing (TU102)
         : RTX 2080
         : Ray Tracing cores (RT Cores)
         : 12nm TSMC
    2020 : Ampere (GA100/GA102)
         : RTX 3090, A100
         : 2nd gen Tensor Cores
         : 8nm Samsung / 7nm TSMC
    2022 : Hopper (GH100)
         : H100 (Data center only)
         : Transformer Engine
         : 4nm TSMC
    2022 : Ada Lovelace (AD102)
         : RTX 4090
         : DLSS 3, Frame Gen
         : 4nm TSMC
    2024 : Blackwell (GB100/GB200)
         : B100, B200, GB200 NVL72
         : 2-chip die, FP4 support
         : 4nm TSMC CoWoS
```

### Inside an NVIDIA GPU: SM Architecture

The **Streaming Multiprocessor (SM)** is NVIDIA's fundamental compute unit:

```mermaid
flowchart TD
    SM["Streaming Multiprocessor (SM)"]
    
    SM --> INT["INT32 Cores\n(integer math)"]
    SM --> FP32["FP32 CUDA Cores\n(float math)"]
    SM --> FP64["FP64 Cores\n(double precision)"]
    SM --> TC["Tensor Cores\n(matrix multiply for AI)"]
    SM --> RT["RT Cores\n(ray tracing BVH traversal)"]
    SM --> L1["L1 Cache / Shared Memory"]
    SM --> REG["Register File\n(per-thread local storage)"]
    SM --> WARP["Warp Scheduler\n(32 threads per warp)"]
```

**Tensor Cores** are the key innovation for AI. They perform matrix multiply-accumulate (MMA) operations at massive speed — critical for training neural networks (which are essentially giant matrix multiplications).

---

## AMD Architecture

AMD splits its GPU line into **two separate architectures**:

```mermaid
flowchart TD
    AMD[AMD GPU Division]
    AMD --> RDNA["RDNA - Gaming\n(Radeon RX series)"]
    AMD --> CDNA["CDNA - Compute/AI\n(Instinct MI series)"]

    RDNA --> RDNA1["RDNA 1 (2019)\nRX 5700 XT"]
    RDNA --> RDNA2["RDNA 2 (2020)\nRX 6000 series\nAlso in PS5/Xbox"]
    RDNA --> RDNA3["RDNA 3 (2022)\nRX 7000 series\nChiplet design"]
    RDNA --> RDNA35["RDNA 3.5 (2024)\nRX 9000 series"]

    CDNA --> CDNA1["CDNA 1 (2020)\nInstinct MI100"]
    CDNA --> CDNA2["CDNA 2 (2021)\nInstinct MI200\nHBM2e, 400GB/s"]
    CDNA --> CDNA3["CDNA 3 (2023)\nInstinct MI300X\nHBM3, 5.3 TB/s"]
```

AMD's compute unit is called a **CU (Compute Unit)**, equivalent to NVIDIA's SM but structured differently:

| Feature | NVIDIA (Ampere SM) | AMD (RDNA 3 CU) |
|---------|-------------------|-----------------|
| SIMD width | 32 threads/warp | 64 threads/wavefront |
| FP32 ALUs | 128 | 128 (2x64 SIMD) |
| Matrix units | Tensor Cores | Matrix Cores |
| Cache | L1 + Shared Mem | L0 + L1 |
| Clock speed | ~1.5–2.5 GHz | ~2.5–3.0 GHz |

---

## Product Line Comparison

```mermaid
flowchart LR
    subgraph NV["NVIDIA Product Lines"]
        NV1["Consumer Gaming\nGeForce RTX\n(3060 → 4090 → 5090)"]
        NV2["Professional Viz\nRTX Pro / Quadro\n(3D, CAD, Media)"]
        NV3["Data Center\nA100, H100, H200\nB100, B200"]
        NV4["Automotive\nDRIVE Thor/Orin"]
        NV5["Edge/Embedded\nJetson Orin"]
    end

    subgraph AM["AMD Product Lines"]
        AM1["Consumer Gaming\nRadeon RX\n(7600 → 9070 XT)"]
        AM2["Professional Viz\nRadeon PRO\n(W7900, etc)"]
        AM3["Data Center AI\nInstinct MI300X\nMI325X"]
        AM4["Embedded in APUs\nRyzen AI CPUs\nPS5, Xbox Series"]
    end
```

---

## NVIDIA vs AMD: Direct Comparison

### Gaming GPUs (Consumer)

| Spec | NVIDIA RTX 5090 (2025) | AMD RX 9070 XT (2025) |
|------|------------------------|------------------------|
| Architecture | Blackwell | RDNA 4 |
| VRAM | 32 GB GDDR7 | 16 GB GDDR6 |
| Memory BW | ~1,800 GB/s | ~640 GB/s |
| TDP | ~575W | 304W |
| Price | ~$2,000 | ~$600 |
| Upscaling | DLSS 4 (AI-based) | FSR 4 (AI-based) |

**NVIDIA's advantages in gaming:**
- DLSS (Deep Learning Super Sampling) — uses AI to upscale, better quality
- Frame Generation — AI-generated intermediate frames
- Better ray tracing RT cores
- Larger VRAM on high-end cards
- CUDA for game physics/compute

**AMD's advantages in gaming:**
- Better price/performance at mid-range
- Open-source FSR works on ANY GPU (including NVIDIA)
- Infinity Cache on RDNA chips (reduces need for bandwidth)
- AMD CPUs + AMD GPUs can use Smart Access Memory (SAM)

### Data Center / AI GPUs

This is where the real money is:

| Spec | NVIDIA H100 SXM | AMD MI300X |
|------|-----------------|------------|
| Architecture | Hopper | CDNA 3 |
| FP8 TFLOPS (AI) | 3,958 | 2,614 |
| BF16 TFLOPS | 1,979 | 1,307 |
| VRAM | 80 GB HBM3 | 192 GB HBM3 |
| Memory BW | 3.35 TB/s | 5.3 TB/s |
| TDP | 700W | 750W |
| Price | ~$30–40K | ~$15–20K |
| Key Advantage | CUDA ecosystem | Larger VRAM |

> **Why does AMD's MI300X have more VRAM?** The MI300X is a **chiplet** design — it combines CPU chiplets + GPU chiplets + HBM memory all in one package. This allowed AMD to stack more HBM for less cost. For very large AI models that need to fit in a single GPU, MI300X can hold models that won't fit in an H100.

---

## Software Ecosystems: The Critical Difference

This is where NVIDIA **dominates**:

```mermaid
flowchart TD
    subgraph NV_SW["NVIDIA Software Stack"]
        APP1["AI Frameworks\nPyTorch, TensorFlow, JAX"] --> CUDA
        APP2["HPC Applications\nGROMACS, NAMD"] --> CUDA
        APP3["Graphics\nVulkan, DirectX, OpenGL"] --> CUDA
        CUDA["CUDA\n(C++ GPU programming language)"]
        CUDA --> cuDNN["cuDNN\n(Neural Network primitives)"]
        CUDA --> cuBLAS["cuBLAS\n(Linear Algebra)"]
        CUDA --> TensorRT["TensorRT\n(Inference Optimization)"]
        CUDA --> NCCL["NCCL\n(Multi-GPU communication)"]
        CUDA --> Triton["Triton\n(Python-like GPU kernels)"]
    end

    subgraph AMD_SW["AMD Software Stack"]
        APP4["AI Frameworks\nPyTorch (ROCm backend)"] --> ROCm
        APP5["HPC Applications\n(limited support)"] --> ROCm
        ROCm["ROCm\n(AMD's CUDA equivalent)"]
        ROCm --> MIOpen["MIOpen\n(like cuDNN)"]
        ROCm --> RCCL["RCCL\n(like NCCL)"]
        ROCm --> HIP["HIP\n(CUDA-compatible API)"]
    end
```

**Why CUDA is a moat:**
- Launched in 2006 — NVIDIA had 15+ years of head start
- Millions of engineers know CUDA
- Every major AI framework (PyTorch, TensorFlow) was built on CUDA first
- Thousands of optimized libraries (cuDNN, cuBLAS, TensorRT)
- HIP (AMD's compatibility layer) can translate CUDA code, but with performance penalties and compatibility gaps
- Most research papers release CUDA code, not ROCm code

---

## Upscaling Wars: DLSS vs FSR

Both companies have AI-powered upscaling that lets games run at lower resolution internally but display at higher resolution — more frames per second for "free":

```mermaid
flowchart LR
    subgraph DLSS["NVIDIA DLSS 4"]
        D1["Internal: 1080p render"] --> D2["AI Tensor Core upscale"] --> D3["Output: 4K display"]
        D4["Frame Generation\nAI creates extra frames"]
        D5["Multi Frame Gen\n3-4 AI frames per real frame"]
    end

    subgraph FSR["AMD FSR 4"]
        F1["Internal: 1080p render"] --> F2["ML upscale algorithm"] --> F3["Output: 4K display"]
        F4["Works on ANY GPU brand\nincl. NVIDIA & Intel"]
    end
```

**DLSS** uses AI (Tensor Cores) — produces better image quality but only works on NVIDIA GPUs.
**FSR** is open-source and hardware-agnostic — less quality but broader reach.

---

## Summary: When to Use Which

| Use Case | NVIDIA | AMD |
|----------|--------|-----|
| AI/ML Training | **Best choice** (CUDA ecosystem) | Good for large models (more VRAM) |
| AI Inference | **Best** (TensorRT) | Catching up |
| 4K Gaming | Best RT, DLSS | Better price/perf at $400-600 |
| 1080p/1440p Gaming | Overkill at high end | Great value |
| Professional 3D/VFX | Quadro/RTX Pro | Radeon PRO (cheaper) |
| Cloud / Hyperscaler | H100/H200/B200 | MI300X (growing) |
| Automotive | DRIVE platform | Less presence |

---

## Next: [Chapter 02 — CPUs](./Chapter_02_CPU_Landscape.md) | [Chapter 04 — NVIDIA's Full Ecosystem](./Chapter_04_NVIDIA_Ecosystem.md)
