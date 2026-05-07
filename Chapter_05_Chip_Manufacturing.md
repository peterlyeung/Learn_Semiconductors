# Chapter 05: How Chips Are Made

## Overview

Making a semiconductor chip is arguably the most complex manufacturing process humanity has ever developed. A modern chip fab requires:
- The cleanest rooms on Earth (Class 1: fewer than 1 particle per cubic foot)
- Lasers with wavelengths shorter than visible light
- ~1,000 precise manufacturing steps
- 3+ months per wafer
- $15-20B+ to build the facility

```mermaid
flowchart TD
    Design["1. Chip Design\n(months to years)"]
    Tape_Out["2. Tape-Out\n(send design to fab)"]
    Wafer_Prep["3. Wafer Preparation\n(pure silicon)"]
    Litho["4. Lithography\n(print circuit pattern)"]
    Etch["5. Etch\n(remove material)"]
    Deposit["6. Deposition\n(add materials)"]
    Repeat["7. Repeat ~100 times\n(layers of transistors + wiring)"]
    Test_W["8. Wafer Test\n(find working dies)"]
    Dicing["9. Dicing\n(cut wafer into individual chips)"]
    Package["10. Packaging\n(add substrate, pins, heatspreader)"]
    Final_Test["11. Final Test\n(sort by speed/power)"]
    Ship["12. Ship to OEM/Customer"]

    Design --> Tape_Out --> Wafer_Prep --> Litho --> Etch --> Deposit --> Repeat
    Repeat --> Test_W --> Dicing --> Package --> Final_Test --> Ship
```

---

## Step 1: Chip Design

Before a single wafer is touched, chip designers spend **2-4 years** designing the chip in software.

```mermaid
flowchart LR
    Spec["Architecture Spec\n(what the chip should do)"] --> RTL
    RTL["RTL Design\n(Register Transfer Level)\nWritten in Verilog/VHDL\nDescribes logic behavior"]
    RTL --> Synth["Logic Synthesis\nConvert RTL → gate-level netlist\n(EDA tools: Synopsys Design Compiler)"]
    Synth --> PnR["Place & Route\nArrange gates on die area\nRoute metal wires between them\n(Cadence Innovus, Synopsys ICC2)"]
    PnR --> STA["Static Timing Analysis\nVerify timing at worst-case conditions\n(hold/setup margins)"]
    STA --> DRC["DRC/LVS\nDesign Rule Check\nLayout vs Schematic\n(Mentor Calibre)"]
    DRC --> GDSII["GDSII / OASIS file\nThe 'blueprint' sent to fab\n(fab-specific rules embedded)"]
    GDSII --> Fab["Tape-Out!\nSend to TSMC/Samsung"]
```

**Key EDA companies**: Synopsys, Cadence, Siemens EDA (Mentor)  
**Key IP companies**: ARM (CPU cores), NVIDIA (used to buy others' IP), Synopsys (standard cells)

---

## Step 2: The Silicon Wafer

```mermaid
flowchart LR
    Sand["Silica sand\n(SiO₂, ~25% Silicon)"] --> Reduction["High-temp reduction\nSiO₂ + C → Si + CO₂"]
    Reduction --> Purify["Purification\n99.9999999% pure Si\n(9 nines!)"]
    Purify --> Crystal["Czochralski Process\nSeed crystal pulled\nfrom molten silicon\n→ Silicon Boule"]
    Crystal --> Slice["Diamond wire saw\nSlice into 300mm wafers\n~1mm thick"]
    Slice --> Polish["Chemical Mechanical\nPolishing (CMP)\nAtomic-level smooth"]
    Polish --> Ready["Ready for fab\n~$200 per wafer"]
```

**300mm (12 inch)** wafers are the industry standard for advanced chips. Larger wafer = more chips per wafer = lower cost per chip.

---

## Step 3: Lithography — The Critical Step

**Lithography** (from Greek: "stone writing") is how the circuit pattern is transferred onto the wafer. It's like photography at atomic scale.

```mermaid
flowchart TD
    Photoresist["1. Coat wafer with\nphotoresist\n(light-sensitive material)"]
    Mask["2. Load photomask\n(quartz glass with circuit pattern\nlike a photographic negative)"]
    Light["3. Shine light through mask\nLight exposes photoresist\nwhere circuit features are"]
    Develop["4. Develop photoresist\nExposed areas wash away\n(or unexposed, depending on type)"]
    Etch2["5. Etch exposed silicon\nRemove material from\nnon-protected areas"]
    Strip["6. Strip remaining photoresist\nRepeat for next layer"]

    Photoresist --> Mask --> Light --> Develop --> Etch2 --> Strip
```

### The Wavelength Problem

To print smaller features, you need shorter wavelength light. Physics limits this:

```mermaid
flowchart LR
    I_Line["i-line (1980s-90s)\n365 nm wavelength\n~500 nm features"] --> KrF
    KrF["KrF DUV (1990s-2000s)\n248 nm laser\n~250 nm features"] --> ArF
    ArF["ArF DUV (2000s-2010s)\n193 nm laser\n~90 nm features"] --> ArF_Immersion
    ArF_Immersion["ArF Immersion DUV\n193 nm + water lens\nMultiple patterning\n~7 nm features possible\nbut with many steps"] --> EUV
    EUV["EUV (2019-now)\n13.5 nm wavelength\nExtreme UV, made in plasma\n~3 nm features in 1 pass"]
    EUV --> HiNA_EUV["High-NA EUV (2025+)\nLarger lens aperture\n~2 nm and below"]
```

---

## ASML: The Only EUV Machine Maker

**ASML** (Netherlands) is one of the most important companies in the world that most people have never heard of. They make the **only** EUV lithography machines that can manufacture chips at 5nm and below.

```mermaid
flowchart TD
    ASML["ASML\nNetherlandsEUV Machine\n(NXE:3600D)"]
    
    ASML --> Zeiss["Carl Zeiss SMT\n(Germany)\nOptical elements\nMirrors polished to\n0.1 nm accuracy"]
    ASML --> Cymer["Cymer (ASML subsidiary)\n(San Diego, USA)\nLaser source\nIR laser hits tin droplets\n→ 13.5 nm EUV plasma"]
    ASML --> VDL["VDL Groep\n(Netherlands)\nMechanical components"]
    ASML --> Trumpf["Trumpf (Germany)\nHigh-power laser source"]

    ASML --> Machine["One EUV Machine:"]
    Machine --> Weight["~180 tonnes"]
    Machine --> Parts["~100,000 components"]
    Machine --> Price["~$200M per machine\n(High-NA: ~$380M)"]
    Machine --> Delivery["18+ month lead time"]
    Machine --> Speed["Exposes 170 wafers/hour"]
```

**Why ASML matters geopolitically**: The US, Netherlands, and Japan have restricted ASML from exporting EUV machines to China. Without EUV, China cannot manufacture chips below 7nm at scale. This is the central battlefield in the US-China semiconductor war.

---

## The Major Foundries

A **foundry** is a chip factory that manufactures chips designed by fabless companies:

```mermaid
flowchart TD
    Foundries["Global Foundries"]

    Foundries --> TSMC["TSMC\n(Taiwan Semiconductor Mfg Co)\nFounded 1987, Morris Chang\nWorldwide: 60%+ market share\nMakes: Apple, NVIDIA, AMD, Qualcomm, Intel"]
    Foundries --> Samsung_F["Samsung Foundry\n(South Korea)\n~15% market share\nMakes: Qualcomm, Google, AMD (some)\nAlso: integrated with Samsung memory"]
    Foundries --> IFS["Intel Foundry Services (IFS)\n(USA)\nExpanding via CHIPS Act\nMakes: Intel chips + external\nProcess: Intel 3, Intel 18A"]
    Foundries --> GF["GlobalFoundries\n(USA, facilities in NY, VT, Germany, Singapore)\nMature nodes: 14nm and above\nFocus: aerospace, defense, automotive"]
    Foundries --> SMIC["SMIC\n(China)\nChina's largest foundry\nStuck at ~7nm (no EUV)\nEntity List restrictions"]
    Foundries --> UMC["UMC\n(Taiwan)\nMature nodes, specialty"]
```

### TSMC: The World's Most Important Factory

TSMC is unique — nearly every advanced chip is made there:

```mermaid
pie title TSMC Revenue by Customer (~2024 estimate)
    "Apple" : 25
    "NVIDIA" : 11
    "AMD" : 8
    "Qualcomm" : 8
    "MediaTek" : 6
    "Broadcom" : 6
    "Intel" : 3
    "Others" : 33
```

**TSMC's process nodes (as of 2025)**:

| Process | EUV? | Key Customers | Status |
|---------|------|---------------|--------|
| N7 (7nm) | No (DUV multi-pattern) | Older products | Mature |
| N5 (5nm) | Yes | iPhone 14, M3 | High volume |
| N4P (4nm enhanced) | Yes | RTX 4090, Ryzen 7000 | High volume |
| N3E (3nm) | Yes | iPhone 15, M3 | Ramping |
| N2 (2nm) | Yes (HiNA) | iPhone 17+, expected | 2025 |
| A16 (1.6nm) | Yes + BackSide Power | iPhone 18+, expected | 2026 |

---

## The Manufacturing Process in Detail

### Transistor Building: FinFET and Gate-All-Around

Modern transistors are 3D structures, not flat:

```mermaid
flowchart LR
    subgraph FinFET["FinFET Transistor (7nm-5nm era)"]
        Fin["Silicon Fin\n(vertical fin of silicon)"]
        Gate_fin["Gate wraps\naround 3 sides\nof the fin"]
        Fin --> Gate_fin
    end

    subgraph GAA["Gate-All-Around / MBCFET (3nm+)"]
        Nanosheets["Stacked horizontal\nsilicon nanosheets\n(2-4 sheets)"]
        Gate_wrap["Gate wraps\ncompletely around\nall nanosheets"]
        Nanosheets --> Gate_wrap
    end

    FinFET -->|"smaller → less control"| GAA
```

- **FinFET**: Gate wraps 3 sides — better control than flat (planar)
- **GAA/MBCFET (Samsung) / NSFET (TSMC N2)**: Gate wraps all 4 sides — even better control
- Better gate control = lower leakage current = lower power consumption

### Back End of Line (BEOL): Wiring the Chip

After transistors are built, they need to be connected with copper wires (metal layers):

```mermaid
flowchart TD
    Transistors["Transistors built\n(Front End of Line, FEOL)"]
    Transistors --> M1["Metal Layer 1 (M1)\nThinest wires\nLocal connections"]
    M1 --> M2["Metal Layer 2 (M2)\nSlightly thicker"]
    M2 --> Mx["Metal Layers 3-15\nIntermediate routing"]
    Mx --> Top_Metal["Top Metal Layers\nThickest wires\nGlobal power distribution"]
    Top_Metal --> Pads["Bond Pads\nConnection points\nfor packaging"]
```

Modern chips have **15+ metal layers** — like a 15-story building where the transistors are the ground floor and each floor is a routing layer.

---

## Advanced Packaging: The New Frontier

As transistor scaling slows, **packaging** is becoming the new battlefield. Instead of putting everything on one die, chipmakers are connecting multiple smaller dies in a package:

```mermaid
flowchart TD
    subgraph Packaging_Types["Advanced Packaging Approaches"]
        CoWoS["CoWoS\n(Chip on Wafer on Substrate)\nTSMC's approach\nUsed in H100: GPU + HBM\non silicon interposer"]

        SoIC["SoIC\n(System on Integrated Chips)\nFace-to-face die stacking\n3D IC integration"]

        FOVEROS["FOVEROS\n(Intel's 3D stacking)\nUsed in Meteor Lake\nlogic + compute tiles"]

        MCM["Multi-Chip Module\n(basic version)\nMultiple dies on\norganic substrate\nAMD EPYC chiplets"]

        2pt5D["2.5D: Dies side by side\non interposer\n(CoWoS, EMIB)"]

        3D["3D: Dies stacked\nvertically\n(SoIC, FOVEROS, HBM)"]
    end
```

### Why Advanced Packaging Matters

```mermaid
flowchart LR
    Why["Why Advanced Packaging?"]
    Why --> Yield["Yield economics:\nSmall dies = higher yield\nConnect 4 small good dies\nvs 1 large die with defects"]
    Why --> Bandwidth["Bandwidth:\nDies connected with\nthousands of micro-bumps\nvs slow PCIe bus"]
    Why --> Hetero["Heterogeneous integration:\nBest-in-class for each function\nLogic at 3nm + Memory at different node"]
    Why --> Power["Power delivery:\nBackside Power Delivery Network\n(TSMC A16, Intel 18A)\nputs power rails below transistors"]
```

---

## The CHIPS Act: US Manufacturing Revival

The US CHIPS and Science Act (2022) allocated **$52.7 billion** to rebuild US semiconductor manufacturing:

```mermaid
flowchart TD
    CHIPS["US CHIPS and Science Act\n$52.7 billion (2022)"]
    
    CHIPS --> TSMC_AZ["TSMC Arizona\nPhoenix, AZ\n$65B total investment\nFab 21: N4P → N3 → N2\nFirst wafers: 2024"]
    CHIPS --> Intel_OH["Intel Ohio\nColumbus, OH\n$20B+\nIntel 18A / 14A\n2026+"]
    CHIPS --> Micron_NY["Micron New York\nClay, NY\n$100B over 20 years\nDRAM fabs\nFirst production: 2025"]
    CHIPS --> Samsung_TX["Samsung Texas\nTaylor, TX\n$17B\n4nm → 2nm fabs"]
    CHIPS --> GF_NY["GlobalFoundries\nMalta, NY\n$1.5B\nMature nodes for defense"]
```

**Why the US is doing this:**
1. National security — 92% of advanced chips made in Taiwan (geopolitical risk)
2. Supply chain resilience — COVID exposed vulnerabilities
3. Economic competition with China
4. Defense applications need domestic supply

---

## Yield: The Economic Reality

**Yield** = percentage of dies on a wafer that pass testing.

```mermaid
flowchart TD
    Wafer["300mm Wafer"]
    Wafer --> Dies["~500-700 dies depending on die size"]
    Dies --> Good["Good dies: 70-95%\n(depending on node/design)"]
    Dies --> Bad["Bad dies: 5-30%\n(defects from particles, EUV errors)"]
    
    Good --> Binning["Speed Binning:\nFastest dies → premium SKU\nAverage → mainstream\nSlowest → low-tier or disabled"]
    Bad --> Scrap["Scrapped / recycled"]
```

**Example: RTX 4090 die size vs yield**
- AD102 die: 608 mm² at 4nm TSMC
- Larger die = more area exposed to defects = lower yield
- This is why the RTX 4090 is so expensive — large dies are inherently expensive to manufacture
- AMD's chiplets solve this: multiple smaller dies, each with better yield

---

## Next: [Chapter 06 — The AI Silicon Race](./Chapter_06_AI_Silicon_Race.md)
