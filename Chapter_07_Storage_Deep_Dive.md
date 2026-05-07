# Chapter 07: Storage — SSDs, NVMe, and the Full Picture

## The Storage Hierarchy Revisited

Storage is where data lives when it's not actively being computed. The key tension is always **speed vs cost vs capacity**:

```mermaid
flowchart TD
    subgraph Hierarchy["Full Storage Hierarchy"]
        CPU_Regs["CPU Registers\n~KB, sub-ns, $$$$$\nInside the CPU"]
        L1L2L3["CPU Cache (SRAM)\n~MB, 1-40 ns, $$$$\nInside the CPU die"]
        DRAM2["DRAM\n8 GB - 2 TB, ~100 ns, $$\nDIMMs/LPDDR modules"]
        NVMe2["NVMe SSD\n512 GB - 16 TB, ~100 µs, $0.10/GB\nM.2/U.2 slots, PCIe"]
        SATA_SSD["SATA SSD\n256 GB - 4 TB, ~200 µs, $0.07/GB\n2.5\" bay or M.2 B+M key"]
        HDD["HDD\n1 TB - 24 TB, ~5-10 ms, $0.02/GB\n3.5\" bay, spinning"]
        Tape["Tape Archive\nPB scale, minutes, <$0.001/GB\nAWS Glacier / enterprise backup"]

        CPU_Regs --> L1L2L3 --> DRAM2 --> NVMe2 --> SATA_SSD --> HDD --> Tape
    end
```

---

## NVMe: The Modern SSD Interface

**NVMe** (Non-Volatile Memory Express) is the protocol that connects SSDs to the CPU via PCIe. It replaced the old AHCI/SATA protocol designed for spinning disks.

### Why NVMe is Better Than SATA

```mermaid
flowchart LR
    subgraph SATA_IF["SATA / AHCI (old)"]
        SATA_Q["1 command queue\nMax 32 commands deep\nDesigned for HDDs\n600 MB/s max"]
    end

    subgraph NVMe_IF["NVMe (modern)"]
        NVMe_Q["65,535 queues\n65,536 commands each\nDesigned for NAND flash\nNo seek time assumption"]
        NVMe_Speed["PCIe 4.0 x4: 7 GB/s\nPCIe 5.0 x4: 14 GB/s\nLatency: ~20-100 µs"]
    end

    SATA_IF -->|"10x faster"| NVMe_IF
```

| Interface | Max Bandwidth | Latency | Queues | Use Case |
|-----------|-------------|---------|--------|----------|
| SATA III | 600 MB/s | ~200 µs | 1 (32 deep) | Budget SSDs, legacy |
| PCIe 3.0 x4 (NVMe) | 3.5 GB/s | ~100 µs | 65K | Gen 3 NVMe (2015-2020) |
| PCIe 4.0 x4 (NVMe) | 7 GB/s | ~50 µs | 65K | Current mainstream |
| PCIe 5.0 x4 (NVMe) | 14 GB/s | ~20 µs | 65K | Enthusiast/workstation |
| PCIe 5.0 x4 enterprise | 14 GB/s | ~10 µs | 65K | Data center |

### M.2 Form Factor

Most consumer NVMe SSDs use the M.2 form factor — a small PCB that plugs directly into the motherboard:

```
M.2 2280 (22mm wide, 80mm long) — most common consumer SSD
┌─────────────────────────────────────────────────────────────┐
│  [Controller] [NAND] [NAND] [NAND] [NAND]  [DRAM cache]   │
└─────────────────────────────────────────────────────────────┘
                                                    M-key connector
```

**M.2 key types**:
- **M-key** — supports PCIe x4 NVMe AND SATA (check your SSD!)
- **B+M key** — usually SATA or PCIe x2 (slower)

### Enterprise Form Factors

| Form Factor | Description | Use |
|-------------|-------------|-----|
| U.2 (2.5") | Larger than M.2, hot-swappable | Enterprise servers |
| EDSFF E1.S | Thin, high density | Cloud data center |
| EDSFF E3.S | Wider, more NAND | Cloud data center |
| CXL Memory | CPU-attached pool memory | Hyperscaler memory expansion |

---

## SanDisk Deep Dive: Products and Technology

### SanDisk's Full Product Line (2025)

```mermaid
flowchart TD
    SanDisk2["SanDisk (now independent again, 2025)"]

    SanDisk2 --> Consumer3["Consumer Products"]
    SanDisk2 --> Pro["SanDisk Professional"]
    SanDisk2 --> Enterprise3["Enterprise"]
    SanDisk2 --> Raw_NAND3["Raw NAND (OEM)"]

    Consumer3 --> MicroSD["MicroSD Cards\nUltra / Extreme / Extreme Pro\n32 GB - 1.5 TB\nFor phones, cameras, Switch"]
    Consumer3 --> SD_Full["Full-size SD Cards\nUHS-I / UHS-II / SD Express\nFor cameras, drones"]
    Consumer3 --> USB_Drives["USB Flash Drives\nUltra / Extreme Go\n32 GB - 512 GB"]
    Consumer3 --> Portable_SSD["Portable SSDs\nExtreme V2 / Extreme Pro\n1-8 TB, USB-C 3.2"]
    Consumer3 --> Internal_SSD["Internal SSDs\nSanDisk Ultra 3D SATA\nSanDisk Extreme M.2 NVMe"]

    Pro --> Pro_Grade["G-Drive / ArmorATD\nRugged portable drives\nFor creative professionals"]
    Pro --> CFexpress["CFexpress Type B\nProfessional cameras\n(Nikon Z9, Canon R3)"]

    Enterprise3 --> Ent_SSD["Ultrastar DC SN650/SN861\nNVMe U.2 enterprise SSDs\nSold under WD brand"]
    Enterprise3 --> QLC_Arch["QLC-heavy designs\nHigh capacity per rack\nRead-heavy workloads"]
```

### SD Card Speed Classes: Decoded

SD cards have a confusing array of speed ratings:

```mermaid
flowchart LR
    subgraph Classes["SD Card Speed Classes"]
        Class10["Class 10 / U1\n10 MB/s minimum write\nFull HD video"]
        U3["UHS Speed Class 3 (U3)\n30 MB/s minimum write\n4K video capture"]
        V30["Video Speed V30\n30 MB/s write\nSame as U3"]
        V60["Video Speed V60\n60 MB/s write\nHigh-res 4K/8K"]
        V90["Video Speed V90\n90 MB/s write\nProRes 4K RAW, 8K"]
        A1["Application Performance A1\n1500 read IOPS, 500 write IOPS\nFor running Android apps"]
        A2["Application Performance A2\n4000 read IOPS, 2000 write IOPS\nFaster app performance"]
    end
```

**Practical guide**:
- Phone storage: **A2 UHS-I U3** (fast app launches + video)
- GoPro / action cam: **V30 or V60** (sustained 4K write)
- Mirrorless camera (burst mode): **V90** (fastest burst buffer clearing)
- Nintendo Switch: **A1 or A2 U3** (game loading)

### CFexpress: Professional Camera Storage

```mermaid
flowchart LR
    CFexpress_Types["CFexpress"]
    CFexpress_Types --> TypeA["Type A (Sony)\nSmall, single PCIe lane\n~800 MB/s\nSony α7/α9 cameras"]
    CFexpress_Types --> TypeB["Type B (most popular)\nMedium, dual PCIe lane\n~1800 MB/s\nNikon Z, Canon EOS R"]
    CFexpress_Types --> TypeC["Type C (emerging)\nLarger, 4 PCIe lanes\n~4000 MB/s\nCinema cameras"]
```

---

## The WD + SanDisk + Kioxia Triangle

This relationship is complicated but important:

```mermaid
flowchart TD
    History["History"]
    History --> WD_Buy["Western Digital acquires SanDisk\n2016, $19B\nGoal: become NAND + HDD company"]
    WD_Buy --> JV2["Joint Venture with Kioxia\n(formerly Toshiba Memory)\nFabs in Yokkaichi & Kitakami, Japan"]
    JV2 --> Products_Made["Products manufactured:\n- Flash chips (BiCS NAND)\n- Used in SanDisk + WD branded products\n- Sold as raw NAND to others"]

    subgraph Spin_Out["2024-2025: The Split"]
        WD_HDD["WD (Western Digital)\n→ keeps HDD business\nWD Gold, WD Red, WD Purple\nWD Blue (SATA SSD)"]
        SanDisk_New["SanDisk Corp (new)\n→ flash business\nConsumer flash + enterprise flash\nKeeps Kioxia JV relationship"]
    end

    JV2 --> Spin_Out
```

**Why the split?**
- HDD and NAND flash are different businesses with different customers, cycles, and capital needs
- Activist investors pushed for separation
- HDD is mature/declining (cloud → NVMe SSDs)
- NAND flash is volatile but high-growth (AI storage, smartphones)

---

## Kioxia: Japan's Flash Giant

**Kioxia** (formerly Toshiba Memory, spun off in 2018) is the other half of the SanDisk JV:

| Fact | Detail |
|------|--------|
| Founded | 2018 (spun from Toshiba) |
| HQ | Tokyo, Japan |
| Ownership | Bain Capital (PE), Toshiba, SK Hynix (small) |
| Revenue | ~$10-12B (varies with NAND pricing) |
| Technology | BiCS NAND (co-invented with SanDisk) |
| Fabs | Yokkaichi (Mie Prefecture), Kitakami (Iwate Prefecture) |
| Brands | Kioxia (enterprise), Exceria (consumer) |

---

## NAND Pricing: Boom and Bust Cycles

NAND flash is notorious for dramatic price swings — this affects everything from SSD prices to company profits:

```mermaid
flowchart TD
    Oversupply["Oversupply\n(too many chips, not enough buyers)"]
    Oversupply --> PriceFall["Prices collapse\nManufacturers lose money\n(sometimes $0.02/GB → $0.03/GB → $0.05/GB)"]
    PriceFall --> CutProduction["Companies cut production\nDelay new fabs\nFire workers"]
    CutProduction --> Shortage["Supply shortage"]
    Shortage --> PriceRise["Prices spike\n(back to $0.08-0.12/GB)"]
    PriceRise --> CapEx["Companies invest in new fabs\nHire again, expand"]
    CapEx --> Oversupply
```

**Recent cycles:**
- 2021-2022: Supply crunch (COVID disruptions + demand surge) → High prices
- 2022-2023: Oversupply, prices crashed 60%+ — Micron, Samsung, SK Hynix all lost money
- 2024: Recovery, AI storage demand + discipline → Prices recovered
- This cycle repeats every 3-5 years

---

## Hard Drives: Still Relevant

HDDs (spinning magnetic disks) aren't dead — they're the cheapest storage at scale:

```mermaid
flowchart LR
    HDD_Makers["HDD Manufacturers\n(only 3 left)"]
    HDD_Makers --> Seagate["Seagate\n(Fremont, CA)\nLargest capacity\n30 TB+ enterprise\nIronWolf NAS, Barracuda desktop"]
    HDD_Makers --> WD2["Western Digital\nWD Gold, WD Purple\nWD Red (NAS)"]
    HDD_Makers --> Toshiba_HDD["Toshiba\nEnterprise HDDs\nNAS/surveillance"]

    HDD_Makers --> Tech["HDD Technologies"]
    Tech --> CMR["CMR\n(Conventional Magnetic Recording)\nFastest writes, most reliable"]
    Tech --> SMR["SMR\n(Shingled Magnetic Recording)\nHigher density, slower writes\nArchival / cold storage"]
    Tech --> HAMR["HAMR\n(Heat-Assisted Magnetic Recording)\nSeagate's laser-assist tech\n30+ TB drives"]
    Tech --> MAMR["MAMR\n(Microwave-Assisted)\nWD's approach\nSimilar capacity to HAMR"]
```

**Where HDDs still win:**
- Bulk cold storage: ~$0.02/GB vs SSD's ~$0.05-0.10/GB
- Video surveillance (write-optimized, continuous)
- Backup / archival
- Cloud storage tiers (hyperscalers use billions of HDDs)

---

## Enterprise SSD vs Consumer SSD: What's Different

```mermaid
flowchart TD
    subgraph Consumer_SSD["Consumer SSD (e.g. SanDisk Extreme)"]
        C_NAND["TLC or QLC NAND\n(cheap, higher density)"]
        C_Cache["Large DRAM cache\n(1 GB per 1 TB)"]
        C_Controller["Budget controller\nPhison, Silicon Motion"]
        C_Endurance["300-600 TBW endurance\n(1-3 years heavy use)"]
        C_Warranty["3-5 year warranty"]
        C_Power["No power loss protection"]
    end

    subgraph Enterprise_SSD["Enterprise SSD (e.g. WD Ultrastar DC)"]
        E_NAND["SLC or MLC NAND\n(or TLC with more overprovisioning)"]
        E_Controller["Enterprise controller\n(Broadcom, Marvell, HGST)"]
        E_Endurance["3-100 DWPD endurance\n(Drive Writes Per Day for 5 years)"]
        E_Warranty["5-year warranty, 24/7 support"]
        E_Power["Power Loss Protection (PLP)\nCapacitors hold power for flush"]
        E_Features["Encryption, sanitize\nEnd-to-end data protection\nT10 Protection Information"]
        E_QoS["QoS guarantees\n<1 ms 99.99th percentile latency"]
    end
```

**Power Loss Protection (PLP)** is critical for enterprise SSDs: if power cuts out mid-write without PLP, the SSD could corrupt data. Enterprise SSDs have capacitors that store enough energy to flush the write buffer to NAND safely.

---

## CXL: The Future of Memory

**CXL** (Compute Express Link) is an emerging standard built on PCIe 5.0 that will blur the line between memory and storage:

```mermaid
flowchart TD
    CXL["CXL (Compute Express Link)"]
    CXL --> CXL1["CXL 1.0/2.0\nMemory expansion\nAttach extra DRAM\nvia PCIe slot\n(no code change needed)"]
    CXL --> CXL3["CXL 3.0\nMemory pooling\nMultiple servers share\na memory pool\n(disaggregated memory)"]

    CXL1 --> UseCases2["Use Cases:\nServer memory expansion\n(cheap vs DRAM DIMMs)\nMemory bandwidth\naggregation for AI"]

    CXL --> Vendors["Vendors:\nSamsung CXL DRAM\nMicron CXL modules\nSK Hynix CXL\nMontage, Astera Labs (controllers)"]
```

CXL essentially lets you add memory "drives" that the CPU treats like RAM — bridging the DRAM / SSD gap.

---

## Summary: The Full Storage Stack

```mermaid
flowchart TD
    subgraph Phone["Smartphone Storage"]
        UFS["UFS 3.1 / 4.0\n(Universal Flash Storage)\nNAND inside the phone\nMicron, Samsung, SK Hynix supply"]
        MicroSD2["MicroSD (optional)\nSanDisk, Samsung, Lexar"]
    end

    subgraph PC["PC / Laptop Storage"]
        M2_SSD["M.2 NVMe SSD\nSanDisk, WD, Crucial (Micron)\nSamsung 990 Pro, SK Hynix P41"]
        SATA_SSD2["SATA SSD (budget)\nSanDisk Ultra 3D\nCrucial MX500"]
    end

    subgraph Server["Server / Cloud Storage"]
        NVMe_Ent["NVMe U.2/E3.S SSDs\nWD Ultrastar, Micron 9400\nSamsung PM9A3, Kioxia CM7"]
        HDD2["HDDs - cold/warm data\nSeagate Exos, WD Gold"]
        Tape2["Tape - archive\nLTO-9 (18 TB/cartridge)"]
    end

    subgraph Memory_AI["AI GPU Memory"]
        HBM3_AI["HBM3E\nMicron, SK Hynix, Samsung\nOn the GPU package itself"]
    end
```

---

## Key Takeaways

| Company | What They Make | Key Products |
|---------|---------------|-------------|
| **Micron** | DRAM + NAND + HBM | Crucial SSDs/RAM, HBM3E for H100/B200, enterprise SSDs |
| **SanDisk** | NAND flash products | MicroSD, USB drives, portable SSDs, enterprise SSDs |
| **Samsung** | DRAM + NAND + HDD components | 990 Pro, PM9A3, HBM2e, LPDDR6 |
| **SK Hynix** | DRAM + NAND | Gold P31, HBM3E (dominant supplier to NVIDIA) |
| **Kioxia** | NAND flash | Exceria consumer, CM7 enterprise (JV with SanDisk) |
| **Seagate** | HDD | Exos enterprise, IronWolf NAS, Barracuda desktop |
| **Western Digital** | HDD + NAND | WD Gold/Red/Purple, WD Black NVMe |

---

## Next Steps

You've completed all 8 chapters! Here's a review path:

1. **[Chapter 00](./Chapter_00_Overview.md)** — Industry structure (read again with context)
2. **[Chapter 05](./Chapter_05_Chip_Manufacturing.md)** — Manufacturing (re-read knowing what products need it)
3. **[Chapter 06](./Chapter_06_AI_Silicon_Race.md)** — The AI race (most current/fast-moving)
4. **[Chapter 04](./Chapter_04_NVIDIA_Ecosystem.md)** — Why NVIDIA won (the software story)

For further learning:
- **Semiconductors Explained** (IEEE Spectrum)
- **TSMC's annual report** — incredibly educational about manufacturing
- **NVIDIA GTC keynotes** — Jensen Huang explains their roadmap
- **AnandTech archives** (RIP, but archived) — deep technical chip reviews
- **Patrick Moorhead, Ian Cutress** — industry analysts on YouTube/Twitter
