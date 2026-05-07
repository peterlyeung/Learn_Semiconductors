# Chapter 06: The AI Silicon Race

## Why AI Changed Everything

The launch of ChatGPT in November 2022 triggered the largest demand shock in semiconductor history. Suddenly, every company needed **massive compute** to train and run AI models. NVIDIA's data center revenue went from ~$15B (2022) to ~$90B (2024) in two years.

```mermaid
flowchart TD
    AI_Demand["AI Training + Inference demand"]
    AI_Demand --> GPU_Need["Massive GPU clusters needed\nGPT-4 trained on ~25,000 A100s\nGPT-5 scale: 100,000+ GPUs"]
    AI_Demand --> Memory_Need["Huge memory bandwidth needed\nLLMs must fit model weights in GPU RAM\nLlama-70B = ~140 GB at FP16"]
    AI_Demand --> Network_Need["Fast networking needed\nGPUs must sync gradients\nacross 10,000+ nodes"]
    AI_Demand --> Power_Need["Enormous power consumption\n1 H100 = 700W\n10,000 H100s = 7 MW\n= a small town's electricity"]
```

---

## Training vs Inference: Two Different Problems

The AI compute market splits into two distinct workloads with different requirements:

```mermaid
flowchart LR
    subgraph Training["AI Training"]
        T1["Goal: Learn from data\nAdjust model weights"]
        T2["Hardware needs:\n- Huge FP16/BF16 compute\n- Large memory (HBM)\n- Fast interconnects\n- Multi-GPU always"]
        T3["Duration: Weeks to months\nfor large models"]
        T4["Best chips:\nNVIDIA H100/H200/B200\nAMD MI300X\nGoogle TPUv5p"]
    end

    subgraph Inference["AI Inference"]
        I1["Goal: Answer user queries\nRun trained model"]
        I2["Hardware needs:\n- Fast latency\n- Lower memory OK\n- High throughput\n- Can be single chip"]
        I3["Duration: Milliseconds\nper query"]
        I4["Best chips:\nNVIDIA H100 (TensorRT)\nAMD MI300X\nGoogle TPUv5e\nAWS Inferentia2\nGroq LPU"]
    end

    Training -->|"trained model"| Inference
```

| Dimension | Training | Inference |
|-----------|---------|----------|
| Compute precision | FP16, BF16, FP8 | INT8, INT4, FP8 |
| Memory needed | 80-192 GB per GPU | 16-80 GB (smaller models) |
| Multi-chip required? | Always | Often optional |
| Power per chip | 700W+ | 300-700W |
| Latency requirement | Doesn't matter | Critical (user waiting) |
| Optimization | Throughput | Tokens/second/dollar |

---

## NVIDIA's AI Chips: The Dominant Line

```mermaid
timeline
    title NVIDIA Data Center AI GPUs
    2017 : Tesla V100
         : First Tensor Cores
         : 14 TFLOPS FP16
         : 32 GB HBM2
         : 300W
    2020 : A100 (Ampere)
         : 3rd gen Tensor Cores
         : 312 TFLOPS TF32
         : 80 GB HBM2e
         : 400W
         : Trained GPT-3
    2022 : H100 (Hopper)
         : Transformer Engine
         : 3,958 TFLOPS FP8
         : 80 GB HBM3
         : 700W
         : Trains today's frontier models
    2024 : H200 (Hopper refresh)
         : Same compute as H100
         : 141 GB HBM3E
         : 1.4x memory bandwidth
         : Better for large models
    2024-25 : B100 / B200 (Blackwell)
            : 2-chip die design
            : FP4 support
            : 192 GB HBM3E (B200)
            : 4x H100 AI perf
            : 1,000W (!)
    2025 : GB200 NVL72
         : 72 B200 + 36 Grace CPUs
         : 13.5 TB total HBM3E
         : Rack-scale supercomputer
```

### The Transformer Engine (H100's Key Innovation)

Large Language Models are built on **transformers** (attention mechanisms). The Transformer Engine in Hopper specifically optimizes for this:

```mermaid
flowchart TD
    Attention["Attention Layer\nmost compute-intensive part of LLMs"]
    Attention --> FP8["FP8 Precision\n(8-bit floating point)\nHalves memory bandwidth vs FP16\nDoubles throughput"]
    FP8 --> Scale["Dynamic scaling\nAuto-adjusts to prevent\nnumeric overflow"]
    Scale --> Result["Net result:\n~2x training speedup\nfor transformer models\nvs A100"]
```

---

## AMD's AI Challenger: MI300X

AMD's Instinct MI300X launched in late 2023 and became the first credible alternative to NVIDIA's H100:

```mermaid
flowchart TD
    subgraph MI300X["AMD Instinct MI300X Chiplet Package"]
        XCD1["GPU Chiplet 1\n(XCD)\nCDNA3"]
        XCD2["GPU Chiplet 2\n(XCD)\nCDNA3"]
        XCD3["GPU Chiplet 3\n(XCD)\nCDNA3"]
        XCD4["GPU Chiplet 4\n(XCD)\nCDNA3"]
        XCD5["GPU Chiplet 5"]
        XCD6["GPU Chiplet 6"]
        XCD7["GPU Chiplet 7"]
        XCD8["GPU Chiplet 8\n(XCD)\nCDNA3"]

        HBM_A["HBM3\n24 GB Stack A"]
        HBM_B["HBM3\n24 GB Stack B"]
        HBM_C["HBM3\n24 GB Stack C"]
        HBM_D["HBM3\n24 GB Stack D"]
        HBM_E["HBM3\n24 GB Stack E"]
        HBM_F["HBM3\n24 GB Stack F"]
        HBM_G["HBM3\n24 GB Stack G"]
        HBM_H["HBM3\n24 GB Stack H"]

        IOD["IO Chiplet"]

        XCD1 & XCD2 & XCD3 & XCD4 & XCD5 & XCD6 & XCD7 & XCD8 --> IOD
        IOD <--> HBM_A & HBM_B & HBM_C & HBM_D & HBM_E & HBM_F & HBM_G & HBM_H
    end
```

**Result**: 192 GB HBM3 (vs H100's 80 GB) — game-changing for large model inference.

| Spec | NVIDIA H100 SXM | AMD MI300X |
|------|-----------------|-----------|
| Architecture | Hopper (monolithic) | CDNA3 (chiplets) |
| VRAM | 80 GB HBM3 | **192 GB HBM3** |
| Memory BW | 3.35 TB/s | **5.3 TB/s** |
| FP8 TFLOPS | 3,958 | 2,614 |
| TDP | 700W | 750W |
| Price | ~$30-40K | ~$10-15K |
| CUDA compat | Native | Via HIP/ROCm |

**AMD wins**: Large models that barely fit in one GPU (needs 192 GB), or price-sensitive buyers.  
**NVIDIA wins**: Ecosystem, software, training workloads, enterprise support.

---

## Google TPU: Training Giants Without NVIDIA

Google has been making its own AI chips (TPUs) since 2016 to avoid paying NVIDIA:

```mermaid
timeline
    title Google TPU Evolution
    2016 : TPU v1
         : Inference only
         : Runs AlphaGo
    2017 : TPU v2
         : Training + inference
         : 45 TFLOPS
    2018 : TPU v3
         : Liquid cooled
         : Used in BERT training
    2021 : TPU v4
         : 275 TFLOPS BF16
         : Used in PaLM, LaMDA
    2023 : TPU v5e (efficiency)
         : 197 TFLOPS
         : Best price/TFLOPS
    2023 : TPU v5p (performance)
         : 459 TFLOPS
         : Frontier model training
    2024 : Trillium (TPU v6)
         : 4.7x TPU v5e perf
```

**TPUs are only available on Google Cloud (GCP)** — Google doesn't sell them externally. They're deeply integrated with JAX and TensorFlow.

**How a TPU works differently from a GPU**:

```mermaid
flowchart LR
    subgraph GPU_Style["GPU: General SIMD"]
        Many_Cores["Thousands of general cores\nEach can do anything\nScheduled dynamically"]
    end

    subgraph TPU_Style["TPU: Systolic Array"]
        SA["Systolic Array\n128x128 or 256x256 grid\nof multiply-accumulate units\nData flows like a wave"]
        SA --> Benefit["Perfect for matrix multiply\n(which is ALL of AI)\nHardware pipeline = no scheduling overhead"]
    end
```

---

## AWS Trainium & Inferentia: Amazon's Chips

Amazon is the world's largest cloud provider. Paying NVIDIA for every GPU operation is expensive. So AWS built its own:

```mermaid
flowchart TD
    AWS_Chips["AWS Custom Silicon"]
    AWS_Chips --> Trainium["AWS Trainium 1/2\nTraining accelerator\n~50% cheaper than GPU equivalent\nfor Transformer training\nUsed: Alexa, Titan models"]
    AWS_Chips --> Inferentia["AWS Inferentia 1/2\nInference accelerator\nHighly optimized for\nspecific model sizes\nUsed: Rekognition, Alexa, SageMaker"]
    AWS_Chips --> Graviton["AWS Graviton 1/2/3/4\nARM-based CPU\nGeneral compute, cheaper\nthan Intel/AMD instances"]
    AWS_Chips --> Nitro["AWS Nitro\nI/O offload chip\nFrees CPU from networking/storage"]
```

**Trainium vs NVIDIA H100**:
- ~50% cheaper per FLOP at scale for supported models
- But: requires AWS-specific SDK (Neuron SDK), porting existing PyTorch code
- Less flexible than NVIDIA — optimized for specific workloads
- Limited availability: only on AWS (no hardware purchase)

---

## Apple Neural Engine: On-Device AI

Apple's **Neural Engine** is an NPU (Neural Processing Unit) integrated into every Apple chip since A11 (iPhone X, 2017). It's optimized for on-device inference:

```mermaid
flowchart TD
    Apple_AI["Apple AI Silicon"]
    Apple_AI --> ANE["Neural Engine (ANE)\nA11: 2 TOPS\nA17 Pro: 35 TOPS\nM4: 38 TOPS"]
    Apple_AI --> UMA2["Unified Memory\nGPU+CPU+NPU share memory\nNo copies needed\nM4 Max: 546 GB/s BW"]
    Apple_AI --> CoreML["Core ML Framework\nDeploys models to ANE\nAutomatically"]

    ANE --> Usecases["Use Cases:\nFace ID\nSiri\nCamera processing\nReal-time translation\nApple Intelligence"]
```

**M4 vs NVIDIA H100 for inference**:
- M4 Ultra: ~38 TOPS NPU + powerful GPU for LLMs
- Can run Llama 70B locally on 192 GB unified memory M4 Ultra
- But: $8,000 Mac Studio vs single H100 performance is orders of magnitude different
- Apple wins for: local/private inference, power efficiency, no internet needed

---

## Groq: The Deterministic LPU

**Groq** (not to be confused with "Grok") has a fundamentally different architecture:

```mermaid
flowchart LR
    Groq_Chip["Groq LPU\n(Language Processing Unit)"]
    Groq_Chip --> Sram["750 MB SRAM on-chip\n(no external DRAM)"]
    Groq_Chip --> Deterministic["Deterministic execution\nNo dynamic scheduling\nCompiler handles everything"]
    Groq_Chip --> Throughput["Result: 800+ tokens/second\nfor Llama 70B (inference)\nvs ~50 tok/s on H100"]

    Groq_Chip --> Limits["Limitations:\nSmall on-chip SRAM limits model size\nTraining not supported\nLimited model support\nExpensive to scale"]
```

Groq shows that specialized architectures can beat NVIDIA at specific tasks — but the ecosystem problem remains.

---

## The Full AI Chip Landscape

```mermaid
flowchart TD
    subgraph Compute["AI Compute Market"]
        Training["Training\n(weeks-months)"]
        Inference["Inference\n(milliseconds)"]
    end

    subgraph Chips["Chip Options"]
        NV_Train["NVIDIA H100/H200/B200\n★ Dominant in training"]
        AMD_Train["AMD MI300X/MI325X\nGrowing alternative"]
        Google_Train["Google TPU v5p\nGCP only"]
        AWS_Train["AWS Trainium 2\nAWS only"]

        NV_Inf["NVIDIA H100 + TensorRT\n★ Dominant in inference"]
        AMD_Inf["AMD MI300X\n192 GB for large models"]
        AWS_Inf["AWS Inferentia 2\nCheap on AWS"]
        Google_Inf["Google TPU v5e\nGCP only"]
        Groq_LPU["Groq LPU\nFastest tokens/sec"]
        Apple_M4["Apple M4/M4 Max\nOn-device inference"]
        Qualcomm_AI["Qualcomm AI 100\nEdge inference"]
    end

    Training --> NV_Train & AMD_Train & Google_Train & AWS_Train
    Inference --> NV_Inf & AMD_Inf & AWS_Inf & Google_Inf & Groq_LPU & Apple_M4 & Qualcomm_AI
```

---

## Precision Matters: FP32, FP16, INT8, FP4

AI training uses different numeric formats. Lower precision = faster + cheaper:

```mermaid
flowchart LR
    FP64["FP64\n64-bit float\nScientific computing\nHigh precision"] --> FP32
    FP32["FP32\n32-bit float\nTraditional ML\nGold standard accuracy"] --> TF32
    TF32["TF32\nNVIDIA's 19-bit hybrid\nSame exponent as FP32\nUsed in A100/H100\nauto-enabled"] --> FP16
    FP16["FP16\n16-bit float\nMixed precision training\n2x FP32 throughput"] --> BF16
    BF16["BF16\nBrain Float 16\nGoogle's format\nSame exponent range as FP32\nBetter numeric stability than FP16"] --> FP8
    FP8["FP8\n8-bit float\nH100 Transformer Engine\n2x BF16 throughput"] --> INT8
    INT8["INT8\nInteger, 8-bit\nPost-training quantization\nInference only"] --> INT4
    INT4["INT4\n4-bit integer\nExtreme compression\nLarge LLMs on consumer GPUs"] --> FP4
    FP4["FP4\nBlackwell B200 support\n2x INT8 throughput\nCutting edge"]
```

**Mixed precision training** (standard since 2018):
- Store weights in FP32 (precision)
- Compute in FP16/BF16 (speed)
- NVIDIA Tensor Cores are designed for FP16/BF16/FP8 matrix ops

---

## Power: The New Bottleneck

AI data centers are hitting power limits that no chip can solve:

```mermaid
flowchart TD
    Power_Crisis["Power Crisis in AI Data Centers"]
    Power_Crisis --> GPU_Power["Single GPU: 300-1000W\n8-GPU server: 2.4-8 kW\n1000-server cluster: 2.4-8 MW"]
    Power_Crisis --> Grid["US data center power demand\n2023: ~17 GW\n2030 projected: ~35-50 GW\n(~5% of US electricity)"]
    Power_Crisis --> Cooling["Cooling challenges:\nAir cooling maxed out at 100W/U\nLiquid cooling now standard\nGB200 NVL72: liquid-cooled per GPU"]
    Power_Crisis --> Nuclear["Nuclear power deals:\nMicrosoft + Three Mile Island (2023)\nAmazon + nuclear sites\nGoogle + nuclear startups"]
```

**Efficiency metrics** the industry cares about:
- **TFLOPS/Watt** — compute per watt
- **Tokens/second/$ ** — inference efficiency
- **PUE** (Power Usage Effectiveness) — how efficiently a data center uses power

---

## The Interconnect Problem at Scale

Training GPT-5 class models requires synchronizing gradients across 100,000+ GPUs. Networking is critical:

```mermaid
flowchart TD
    Scale["Scale: 100,000 GPUs"]
    Scale --> InfiniBand_Scale["InfiniBand NDR/XDR\n400 Gb/s - 1.6 Tb/s per port\nNVIDIA Quantum-3 switches\nMost advanced clusters"]
    Scale --> Ethernet_Scale["400G/800G Ethernet\n(RoCE - RDMA over Ethernet)\nLower cost, wider availability\nMeta, Google, AWS prefer"]
    Scale --> NVLink_Scale["NVLink within a server\n900 GB/s per GPU\nNVSwitch fabric\nThen InfiniBand between servers"]

    InfiniBand_Scale & Ethernet_Scale --> Topology["Topology: Fat-Tree\nFully non-blocking\nBisection bandwidth preserved\nat scale"]
```

---

## What's Next in AI Silicon

```mermaid
timeline
    title AI Chip Roadmap
    2024 : Blackwell (NVIDIA B200)
         : MI300X mainstream
         : Google Trillium
    2025 : Rubin (NVIDIA R100)
         : AMD MI350
         : Google TPU v7
         : Intel Gaudi 4
    2026 : Rubin Ultra (NVLink 6)
         : AMD MI400
         : Custom silicon from every major cloud
    2027+ : 2nm and below
           : Optical interconnects
           : 3D stacked compute+memory
           : Neuromorphic experiments
```

---

## Next: [Chapter 07 — Storage Deep Dive](./Chapter_07_Storage_Deep_Dive.md) | [Back to Chapter 04 — NVIDIA Ecosystem](./Chapter_04_NVIDIA_Ecosystem.md)
