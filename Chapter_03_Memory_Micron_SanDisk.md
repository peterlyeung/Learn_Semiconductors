# Chapter 03: Memory — DRAM, NAND Flash, Micron & SanDisk

## The Memory Hierarchy

Modern computers use several different types of memory arranged in a hierarchy. Speed goes up as you get closer to the CPU; capacity goes up as you move further away.

```mermaid
flowchart TD
    subgraph Hierarchy["Memory Hierarchy (top = fastest, most expensive per GB)"]
        direction TB
        L1["L1 Cache\n~32 KB per core\n~1 ns latency\nSRAM, embedded in CPU die"]
        L2["L2 Cache\n~256 KB - 1 MB per core\n~4 ns\nSRAM"]
        L3["L3 Cache\n~8 MB - 192 MB shared\n~20-40 ns\nSRAM"]
        DRAM["Main Memory (DRAM)\n8 GB - 2 TB\n~80 ns latency, ~100 GB/s BW\nDDR5, LPDDR5"]
        NVMe["NVMe SSD (NAND)\n500 GB - 8 TB\n~100 µs latency, ~7 GB/s\nNAND Flash"]
        SATA["SATA SSD / HDD\n500 GB - 20 TB\n~200-500 µs / 10 ms\nNAND or spinning disk"]

        L1 --> L2 --> L3 --> DRAM --> NVMe --> SATA
    end
```

There are two fundamentally different memory technologies: **DRAM** and **NAND Flash**.

---

## DRAM: Dynamic RAM

**DRAM** is your computer's main working memory. Every program you run lives here while running.

### How DRAM Works

```mermaid
flowchart LR
    subgraph DRAM_Cell["One DRAM Cell = 1 bit"]
        Transistor["Transistor\n(switch)"]
        Capacitor["Capacitor\n(stores charge = 1 or 0)"]
        Transistor --> Capacitor
    end
    
    Row["Row address\n(wordline)"] --> Transistor
    Col["Column address\n(bitline)"] --> Transistor
```

- **1 transistor + 1 capacitor per bit** — very dense, very cheap
- **Dynamic** = the capacitor slowly loses charge. Must be **refreshed** thousands of times per second (this is why DRAM consumes power even when idle)
- Much **denser** than SRAM (cache) but much **slower** to access
- **Volatile** = loses data when power is cut

### DRAM Generations

```mermaid
timeline
    title DRAM Evolution
    2000 : DDR1
         : 200-400 MT/s
         : PC desktops
    2004 : DDR2
         : 400-1066 MT/s
         : Lower voltage
    2007 : DDR3
         : 800-2133 MT/s
         : Still in old systems
    2014 : DDR4
         : 1600-3200 MT/s
         : Most PCs until 2022
    2021 : DDR5
         : 4800-8400 MT/s
         : Current standard
         : On-die ECC
    2015 : LPDDR4
         : Low Power
         : Phones, laptops
    2022 : LPDDR5X
         : 8533 MT/s
         : Current phones & thin laptops
    2013 : HBM1
         : Stacked 3D DRAM
         : High Bandwidth Memory
    2020 : HBM2e
         : 460 GB/s per stack
         : AI GPU memory
    2023 : HBM3
         : 819 GB/s per stack
         : H100, MI300X
    2024 : HBM3E
         : 1.2 TB/s per stack
         : H200, MI325X
```

### HBM: High Bandwidth Memory (The AI Memory)

This is critical for AI chips. Instead of GDDR memory sitting on a separate chip connected by a bus, HBM **stacks memory dies vertically** right next to the GPU:

```mermaid
flowchart TD
    subgraph HBM_Stack["HBM Stack"]
        D8["DRAM Die 8"]
        D7["DRAM Die 7"]
        D6["DRAM Die 6"]
        D5["DRAM Die 5"]
        D4["DRAM Die 4"]
        D3["DRAM Die 3"]
        D2["DRAM Die 2"]
        D1["DRAM Die 1 (bottom)"]
        Base["Base Die / Logic"]
        D8 --> D7 --> D6 --> D5 --> D4 --> D3 --> D2 --> D1 --> Base
    end

    subgraph Package["Interposer Package"]
        GPU["GPU Die\n(e.g. NVIDIA GH100)"]
        HBM_Stack
        Interposer["Silicon Interposer\n(connects GPU ↔ HBM\nwith 1024-bit wide bus)"]
        GPU <-->|"1024-bit bus\n~1 TB/s"| Interposer
        HBM_Stack <--> Interposer
    end
```

HBM delivers **10-15x more bandwidth** than GDDR at similar power consumption — essential for AI training where GPUs are constantly hungry for data.

---

## Micron Technology

**Founded**: 1978, Boise, Idaho  
**Type**: IDM (makes both DRAM and NAND Flash)  
**Revenue**: ~$30B (FY2024)  
**Headquarters**: Boise, Idaho, USA

Micron is the **only major US-based memory manufacturer**. The other two dominant DRAM makers (Samsung, SK Hynix) are Korean.

### Micron's Product Portfolio

```mermaid
flowchart TD
    Micron["Micron Technology"]

    Micron --> DRAM["DRAM Division"]
    Micron --> NAND["NAND Flash Division"]
    Micron --> NOR["NOR Flash\n(small, for firmware/BIOS)"]

    DRAM --> DDR5_M["DDR5\nDesktop/Server"]
    DRAM --> LPDDR5_M["LPDDR5 / LPDDR5X\nMobile / Thin laptops"]
    DRAM --> HBM_M["HBM3 / HBM3E\nAI GPUs - biggest growth area"]
    DRAM --> GDDR6_M["GDDR6 / GDDR7\nDiscrete GPU memory"]
    DRAM --> RDIMM["RDIMMs / LRDIMMs\nServer memory"]

    NAND --> Consumer_N["Consumer SSDs\nCrucial brand\n(P3, P5, T700 NVMe)"]
    NAND --> Enterprise_N["Enterprise SSDs\nCrucial Pro, data center"]
    NAND --> Storage_N["Storage: eMMC, UFS\n(phones, embedded)"]
    NAND --> Raw_NAND["Raw NAND\nsold to SSD makers"]
```

### Crucial — Micron's Consumer Brand

When you buy a **Crucial** RAM kit or SSD at Best Buy, that's Micron. Crucial is Micron's consumer-facing brand.

| Product | Type | Example |
|---------|------|---------|
| Crucial T700 | NVMe PCIe 5.0 SSD | Up to 7.4 GB/s read |
| Crucial P3 Plus | NVMe PCIe 4.0 SSD | Budget option |
| Crucial Pro DDR5 | DDR5 DRAM | Up to DDR5-6000 |
| Crucial Pro DDR4 | DDR4 DRAM | Legacy systems |

### Micron's HBM Business (Strategic)

Micron's HBM is critical to the AI boom. NVIDIA H200 and GB200 use HBM3E, and Micron is one of only three suppliers:

```mermaid
pie title HBM Market Share (2024 estimate)
    "SK Hynix" : 53
    "Samsung" : 30
    "Micron" : 17
```

Micron is **ramping up HBM production aggressively** — it's their highest-margin product.

### The Memory Oligopoly

DRAM is controlled by 3 companies. This is intentional — the capital required to build a DRAM fab is so extreme that new entrants can't compete:

```mermaid
pie title Global DRAM Market Share (2024)
    "Samsung (Korea)" : 43
    "SK Hynix (Korea)" : 35
    "Micron (USA)" : 22
```

> **Geopolitical note**: If tensions with Korea escalated, the US and much of the world would face a severe DRAM shortage. This is why the US CHIPS Act (2022) invested heavily to encourage Micron to build fabs in New York ($100B+ planned over 20 years).

---

## NAND Flash: Non-Volatile Storage

Unlike DRAM, **NAND Flash retains data without power**. This is what's inside every SSD, phone, USB drive, and SD card.

### How NAND Works

```mermaid
flowchart LR
    subgraph NAND_Cell["NAND Flash Cell"]
        FG["Floating Gate\n(stores electrons = data)"]
        ControlGate["Control Gate"]
        Channel["Channel (source → drain)"]
        Oxide["Oxide layers\n(insulate floating gate)"]
        ControlGate --> Oxide --> FG --> Oxide --> Channel
    end
```

- Electrons are trapped in the floating gate by tunneling through oxide
- **No mechanical parts** — purely electronic (unlike HDDs)
- **Slow to erase** (must erase a full block before writing)
- **Wears out** over time (program/erase cycles)
- **Cheaper per GB than DRAM** — but much slower

### NAND Cell Types: Bits Per Cell

```mermaid
flowchart LR
    SLC["SLC\n1 bit/cell\n2 voltage levels\nFastest, most durable\n100K P/E cycles\nMost expensive/GB"]
    MLC["MLC\n2 bits/cell\n4 voltage levels\nEnterprise SSDs\n10K P/E cycles"]
    TLC["TLC\n3 bits/cell\n8 voltage levels\nConsumer SSDs\n1K P/E cycles\nSweet spot"]
    QLC["QLC\n4 bits/cell\n16 voltage levels\nCheapest/GB\n300 P/E cycles\nLow-write workloads"]
    PLC["PLC (emerging)\n5 bits/cell\n32 voltage levels\nArchival / read-heavy only"]

    SLC --> MLC --> TLC --> QLC --> PLC
```

**Most consumer SSDs today use TLC**. The SSD controller compensates for TLC's limitations using:
- **SLC cache**: a fast buffer zone using TLC cells in SLC mode
- **Error correction (ECC)**: fixes bit errors
- **Wear leveling**: spreads writes across the entire drive

### 3D NAND: Stacking Layers

Scaling 2D NAND hit limits around 2015. The solution: **stack layers vertically** (like a skyscraper):

```mermaid
flowchart TD
    subgraph ThreeD["3D NAND Structure"]
        L200["Layer 200-300\n(top string)"]
        L150["Layer 150-200"]
        L100["Layer 100-150"]
        L50["Layer 50-100"]
        L1_2["Layer 1-50\n(bottom string)"]
        Base2["Silicon Substrate"]

        L200 --> L150 --> L100 --> L50 --> L1_2 --> Base2
    end
```

| Year | Technology | Layers |
|------|-----------|--------|
| 2015 | First 3D NAND | 24-32 layers |
| 2019 | 3rd gen | 64-96 layers |
| 2022 | 5th gen | 176-232 layers |
| 2024 | 7th gen | 232-300+ layers |
| 2025+ | Next gen | 400+ layers (emerging) |

More layers = more storage per unit area = cheaper GB.

---

## SanDisk: The Flash Pioneer

**Founded**: 1988, Sunnyvale, California  
**Founders**: Eli Harari (inventor of flash memory cell), Sanjay Mehrotra, Jack Yuan  
**Acquired by**: Western Digital (WD) in 2016 for $19B  
**Status**: Being spun out as independent company again (2024–2025)

SanDisk didn't just use flash memory — they **invented the commercial NAND flash storage industry**.

### SanDisk's History

```mermaid
timeline
    title SanDisk History
    1988 : Founded by Eli Harari
         : Harari held key patents on flash memory cells
    1991 : First flash card product
         : 20 MB for $1000
    1994 : CompactFlash format
         : Became standard for cameras
    2000 : SD Card standard
         : Co-created with Matsushita and Toshiba
    2005 : 1 GB USB drive
         : Flash becoming mainstream
    2008 : 32 GB SDHC
         : High-speed cards
    2016 : Acquired by Western Digital
         : $19 billion deal
    2020 : WD + SanDisk = combined brand
         : "Western Digital | SanDisk"
    2024 : WD announces SanDisk spin-off
         : Separating HDD (WD) from Flash (SanDisk)
    2025 : SanDisk Corp relaunched
         : Independent flash company again
```

### SanDisk / WD NAND Operations

SanDisk/WD doesn't own its NAND fabs outright — it operates a joint venture with **Kioxia** (formerly Toshiba Memory):

```mermaid
flowchart TD
    JV["Flash Forward Joint Venture\n(Yokkaichi & Kitakami, Japan)"]
    WD["Western Digital / SanDisk\n~49.9% ownership"]
    Kioxia["Kioxia (Japan)\n~50.1% ownership\nFormerly: Toshiba Memory"]
    
    WD <--> JV
    Kioxia <--> JV
    
    JV --> BiCS["BiCS NAND\n(Bit Cost Scalable)\n3D NAND technology"]
    BiCS --> Products["Products"]
    Products --> Consumer2["Consumer:\nSanDisk Ultra SD cards\nSanDisk Extreme SSDs\nSanDisk USB drives"]
    Products --> Enterprise2["Enterprise:\nWD Gold, WD Purple (surveillance)\nUltraStar data center SSDs"]
    Products --> Raw_NAND2["Raw NAND wafers\nSold to SSD makers"]
```

### SanDisk's NAND Technology: BiCS

SanDisk/Kioxia pioneered **BiCS (Bit Cost Scalable)** — their version of 3D NAND:

| Generation | Layers | Year | Key Feature |
|-----------|--------|------|-------------|
| BiCS 1 | 48 | 2015 | First 3D NAND |
| BiCS 3 | 64 | 2017 | TLC mainstream |
| BiCS 5 | 112 | 2020 | QLC viable |
| BiCS 6 | 162 | 2022 | Higher density |
| BiCS 8 | 218+ | 2024 | Current gen |

### SanDisk Consumer Products

```mermaid
flowchart LR
    SanDisk["SanDisk Brand"]
    SanDisk --> Flash2["Flash Cards"]
    SanDisk --> USB["USB Drives"]
    SanDisk --> Portable["Portable SSDs"]
    SanDisk --> MSDSS["MicroSD\n(phones, cameras, Switch)"]

    Flash2 --> UltraSD["SanDisk Ultra SDXC\n128GB-1TB\nA2, UHS-I"]
    Flash2 --> ExtremeSD["SanDisk Extreme Pro\n4K video, cameras"]

    USB --> Ultra3["SanDisk Ultra\nBasic USB-A"]
    USB --> Extreme2["SanDisk Extreme Go\nUSB 3.2, fast"]

    Portable --> ExtremePSSD["SanDisk Extreme Portable SSD\n1-4TB, USB-C"]
    Portable --> ProElite["SanDisk Professional Pro-Elite G-Drive"]
```

---

## NAND Market: The Other Oligopoly

Like DRAM, NAND is controlled by just a few companies:

```mermaid
pie title Global NAND Flash Market Share (2024)
    "Samsung (Korea)" : 33
    "SK Hynix + Solidigm (Korea/US)" : 20
    "Kioxia/WD/SanDisk (Japan/US)" : 32
    "Micron (USA)" : 13
    "YMTC (China)" : 2
```

> **China's YMTC**: China's Yangtze Memory Technologies is aggressively building NAND capacity. They were added to the US Entity List in 2023, blocking them from buying US equipment and being used in US products. This is a major geopolitical flashpoint.

---

## Memory vs Storage: Quick Reference

| | DRAM (Main Memory) | NAND Flash (SSD) |
|-|--------------------|------------------|
| **Volatile?** | Yes (loses data on power off) | No (retains data) |
| **Speed (read)** | ~50 GB/s (DDR5) | ~7 GB/s (NVMe Gen5) |
| **Latency** | ~80 ns | ~100 µs (1000x slower) |
| **Cost/GB** | ~$3-5/GB | ~$0.05-0.10/GB |
| **Durability** | Effectively unlimited | Limited P/E cycles |
| **Use** | Running programs | Storing files |
| **Key makers** | Samsung, SK Hynix, Micron | Samsung, Kioxia/WD, Micron, SK Hynix |

---

## Next: [Chapter 04 — NVIDIA's Ecosystem](./Chapter_04_NVIDIA_Ecosystem.md) | [Chapter 07 — Storage Deep Dive](./Chapter_07_Storage_Deep_Dive.md)
