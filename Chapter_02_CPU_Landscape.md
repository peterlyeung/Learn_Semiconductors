# Chapter 02: CPUs — Intel, AMD, and the ARM Revolution

## What Is a CPU?

The **CPU (Central Processing Unit)** is the "brain" of a computer. Unlike a GPU's thousands of simple cores, a CPU has fewer but far more powerful cores — each capable of handling complex, branching logic quickly. It runs your operating system, applications, and coordinates all other hardware.

---

## Two Instruction Sets: x86 vs ARM

Almost every CPU in the world uses one of two instruction set architectures (ISAs). This is the "language" the hardware speaks:

```mermaid
flowchart TD
    ISA["Instruction Set Architecture (ISA)"]
    ISA --> x86["x86 / x86-64\n(CISC - Complex)"]
    ISA --> ARM["ARM / AArch64\n(RISC - Simplified)"]
    ISA --> RISC5["RISC-V\n(Open-source, emerging)"]

    x86 --> Intel["Intel\nCore, Xeon"]
    x86 --> AMD_x86["AMD\nRyzen, EPYC, Threadripper"]

    ARM --> Apple["Apple\nM1/M2/M3/M4 (Mac, iPad)"]
    ARM --> Qualcomm["Qualcomm\nSnapdragon (phones, laptops)"]
    ARM --> AWS["Amazon\nGraviton (cloud)"]
    ARM --> Ampere["Ampere Computing\n(cloud servers)"]
    ARM --> MediaTek["MediaTek\nDimensity (phones)"]
    ARM --> Samsung_Ex["Samsung\nExynos (phones)"]

    RISC5 --> SiFive["SiFive"]
    RISC5 --> Alibaba["T-Head (Alibaba)"]
```

### CISC vs RISC: The Key Difference

| Aspect | CISC (x86) | RISC (ARM) |
|--------|-----------|-----------|
| Instructions | Complex, variable-length | Simple, fixed-length |
| Power consumption | Higher | Lower |
| Performance/watt | Lower | Higher |
| Compatibility | Windows/Linux ecosystem | Mobile + growing PC |
| Backwards compat | 40+ years (from 8086!) | Newer, less legacy |

> **Why does backwards compatibility matter?** x86 software written in 1990 can still run on a modern Intel Core. That's a massive ecosystem lock-in. But it also means Intel carries 40 years of complexity "baggage" in every chip.

---

## Intel

Intel was the dominant CPU maker for decades, fabricating its own chips (IDM model).

### Product Lines

```mermaid
flowchart LR
    Intel["Intel CPUs"]
    Intel --> Consumer["Consumer\n(Desktop & Laptop)"]
    Intel --> Server["Server\n(Data Center)"]
    Intel --> Embedded["Embedded\n(IoT, Industrial)"]

    Consumer --> Core_i["Core i-series\ni3, i5, i7, i9\n(legacy naming)"]
    Consumer --> Core_U["Core Ultra\n(new branding, 2023+)\nSeries 1, 2, 3"]
    Consumer --> Celeron["Celeron / Pentium\n(budget)"]

    Server --> Xeon["Xeon\n(server/workstation)\nScalable line"]
    Server --> Xeon_Max["Xeon Max\n(HBM-equipped)"]

    Embedded --> Atom["Atom / Pentium Silver\n(low power)"]
```

### Intel's Architecture Strategy: Hybrid Cores

Since **Alder Lake (12th Gen, 2021)**, Intel uses a hybrid design with two types of cores:

```mermaid
flowchart TD
    Intel_Die["Intel Core i9-13900K Die"]
    Intel_Die --> PCore["P-Cores (Performance)\n8 cores, high clock speed\nHyper-Threading: 16 threads\nRuns demanding tasks"]
    Intel_Die --> ECore["E-Cores (Efficiency)\n16 cores, lower power\nNo HT\nBackground tasks, multitasking"]
    Intel_Die --> GPU_iGPU["Intel UHD Graphics\n(integrated GPU)"]
    Intel_Die --> NPU["NPU (Neural Processing Unit)\nAI acceleration on-chip"]
    Intel_Die --> IMC["Memory Controller\nDDR5 / LPDDR5"]
    Intel_Die --> PCIe["PCIe Controller\nPCIe 5.0"]
```

### Intel's Struggles (2015–2023)

Intel fell behind in process technology:
- **2015**: Intel announces 10nm, repeatedly delays
- **2019**: AMD releases 7nm Ryzen 3000, outperforms Intel
- **2021**: Intel finally ships 10nm (renamed "Intel 7"), competitive again
- **2022**: Intel launches 10nm "Intel 7" 12th Gen — competitive
- **2024**: Intel 3 (Intel 4 process, confusing naming) — Intel Gaudi 3, Lunar Lake
- **Problem**: Intel's own foundry is still catching up to TSMC's 3nm

---

## AMD

AMD went fabless in 2009, spinning off its fabs into what became **GlobalFoundries**. This was risky — but it freed AMD to use TSMC's leading-edge nodes.

### AMD's Chiplet Revolution

AMD's biggest strategic innovation was the **chiplet architecture** (2017+). Instead of making one giant die, they split the chip into smaller "chiplets" and link them together:

```mermaid
flowchart TD
    subgraph Ryzen9_9950X["AMD Ryzen 9 9950X Package"]
        CCD1["CCD 1\n(Core Chiplet Die)\n8 cores @ 4nm TSMC"]
        CCD2["CCD 2\n(Core Chiplet Die)\n8 cores @ 4nm TSMC"]
        IOD["IOD\n(I/O Die)\nMemory controller\nPCIe, USB\n6nm TSMC"]
        CCD1 <-->|Infinity Fabric| IOD
        CCD2 <-->|Infinity Fabric| IOD
    end

    IOD --> DDR5["DDR5 Memory\n(2 channels)"]
    IOD --> PCIe5["PCIe 5.0\n(GPU, NVMe)"]
    IOD --> USB["USB / SATA"]
```

**Why chiplets are clever:**
- Smaller dies have higher yield (fewer defects per die)
- Can use different manufacturing nodes for different parts
- Mix and match: 4 CCDs = 64-core EPYC server chip; 1 CCD = 8-core desktop chip
- Same CCD design across product families — economics of scale

### AMD Product Lines

```mermaid
flowchart LR
    AMD["AMD CPUs"]
    AMD --> Consumer["Consumer"]
    AMD --> Server["Server / Workstation"]
    AMD --> Embedded2["Embedded / APU"]

    Consumer --> Ryzen["Ryzen 5/7/9\n(desktop)"]
    Consumer --> Ryzen_M["Ryzen AI\n(laptop, NPU built-in)"]
    Consumer --> Threadripper["Threadripper\n(desktop workstation\n64-core)"]

    Server --> EPYC["EPYC\n(server)\nGenoa: 96 cores\nTurin: 192 cores"]

    Embedded2 --> APU["APU\n(CPU + GPU same chip)\nRyzen AI 9 HX"]
    Embedded2 --> RDNA_APU["Ryzen + RDNA GPU\n(no discrete GPU needed)"]
```

### AMD vs Intel CPU Comparison

| Area | AMD Ryzen 9000 | Intel Core Ultra 200 |
|------|----------------|----------------------|
| Manufacturing | 4nm TSMC | Intel 3 / TSMC N3 |
| Architecture | Zen 5 | Lion Cove + Skymont |
| Top cores | 16 (desktop) | 24 (8P + 16E) |
| Gaming | Excellent | Excellent |
| Multi-thread | Excellent | Excellent |
| Power efficiency | Better at load | Better at idle |
| Platform | AM5 socket | LGA1851 socket |
| Server | EPYC (market leader) | Xeon (declining share) |

---

## ARM: The Silent Winner

ARM Holdings (now owned by SoftBank, ~90% IPO'd in 2023) **does not make chips**. They design CPU core IP and license it to everyone else.

```mermaid
flowchart TD
    ARM["ARM Holdings\n(IP Licensor)"]

    ARM --> Apple2["Apple Silicon\nM1/M2/M3/M4\n(Mac, iPad, iPhone)"]
    ARM --> Qualcomm2["Qualcomm Snapdragon\n(Android phones, Windows PCs)"]
    ARM --> AWS2["Amazon Graviton\n(AWS servers)\n40% cheaper than Intel"]
    ARM --> Google_T["Google Tensor\n(Pixel phones)"]
    ARM --> NVIDIA_G["NVIDIA Grace\n(data center CPU\nin GB200)"]
    ARM --> MediaTek2["MediaTek Dimensity\n(mid-range phones)"]
    ARM --> Samsung_Ex2["Samsung Exynos\n(phones, TVs)"]
    ARM --> Raspberry["Raspberry Pi\nBCM2712"]
    ARM --> Automotive["Automotive SoCs\nTesla, NXP, TI"]
```

### Apple Silicon: The Gold Standard

Apple's M-series chips show what's possible when you control hardware + software + OS:

```mermaid
flowchart TD
    subgraph M4Pro["Apple M4 Pro (2024)"]
        P_Cores["12 Performance Cores\n(Firestorm class)"]
        E_Cores2["4 Efficiency Cores\n(Icestorm class)"]
        iGPU["20-core GPU\n(Apple RDNA-like)"]
        NPU2["16-core Neural Engine\n38 TOPS AI"]
        UMem["Unified Memory\n24-48 GB\nShared by CPU+GPU+NPU"]
        ISP["ISP, Secure Enclave\nMedia Engine (H.264/H.265)"]

        P_Cores & E_Cores2 & iGPU & NPU2 --> UMem
    end
```

**Unified Memory Architecture (UMA)** — Apple's key innovation:
- CPU and GPU share the **same physical memory pool**
- No copying data between CPU RAM and GPU VRAM — zero copy overhead
- Memory bandwidth is incredibly high (~400 GB/s on M4 Max)
- This is why Apple Macs punch above their weight for AI inference

---

## Server CPUs: AMD EPYC Dominates

In data centers, AMD's EPYC has taken massive market share from Intel Xeon:

```mermaid
pie title x86 Server CPU Market Share (2024 estimate)
    "AMD EPYC" : 34
    "Intel Xeon" : 66
```

### Why EPYC Wins in Data Centers

- **More cores**: EPYC Genoa = 96 cores. Intel Xeon = 60 max (lower-end competition)
- **More memory bandwidth**: 12-channel DDR5 vs Intel's 8-channel
- **More PCIe lanes**: 128 PCIe 5.0 lanes (more GPUs/NVMe)
- **Lower price**: Often 20-40% cheaper for equivalent performance
- **Chiplets work perfectly here**: Servers don't have the same frequency demands as gaming

---

## Instruction Set Architecture Future: RISC-V

RISC-V is an **open-source ISA** — anyone can build a chip using it without paying royalties. It's growing fast in embedded systems, and China is investing heavily in it to reduce ARM dependence.

```mermaid
flowchart LR
    RISCV["RISC-V"]
    RISCV --> Today["Today (2025)"]
    RISCV --> Future["Future"]

    Today --> Emb["Embedded: microcontrollers\nESP32-based, WCH chips"]
    Today --> China["China: Alibaba T-Head\nXiangshan architecture"]
    Today --> SiFive2["SiFive HiFive boards"]

    Future --> Server2["Server potential\n(no royalties = cheaper)"]
    Future --> GPU_ish["Neural Network accelerators\nwith RISC-V control cores"]
```

---

## Summary

| Dimension | Intel | AMD | ARM |
|-----------|-------|-----|-----|
| x86 compatibility | Yes | Yes | No |
| Market: Consumer | Competitive | Leading | Growing (laptops) |
| Market: Server | ~66% share | ~34% share | ~5% (Graviton, etc) |
| Market: Mobile | Irrelevant | Irrelevant | **Dominant (99%)** |
| Manufacturing | Own fabs (Intel 3/4/7) | TSMC 3/4nm | Via licensees |
| Efficiency | Moderate | High | Highest |
| GPU integration | Intel Arc (discrete) | RDNA in APUs | Apple: best iGPU |

---

## Next: [Chapter 03 — Memory (Micron, SanDisk)](./Chapter_03_Memory_Micron_SanDisk.md)
