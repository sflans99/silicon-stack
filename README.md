# Semiconductor Industry Supply Chain Dataset

**Version:** 0.6.2
**Last updated:** 2026-04-28
**Stage:** render-ready

A graph-oriented dataset describing the global semiconductor supply chain — companies, materials, and the relationships that connect them — built to power an interactive two-page web visualization.

---

## Overview

| Metric | Count |
|---|---|
| Companies (nodes) | 239 |
| Materials (taxonomy) | 115 |
| Relationships (edges) | 1,110 |
| Orphan edges | 0 |
| Node types | 10 |
| Geo clusters (metros + country fallbacks) | 69 |
| Companies in metro clusters | 206 |
| Single-source-risk nodes flagged | 16 |
| Geographic-risk nodes flagged | 9 |
| Export-control-exposed nodes flagged | 10 |

Every company has a `page` field controlling which visualization page it appears on. There are no orphan edges; every relationship has both endpoints present in `companies` and a valid `material_id`.

---

## Visualization plan

### Page 1 — Raw materials

Bubble map of the world's specialty material suppliers, sized by importance (`scale_indicator`, 0–100), with relationship lines to the foundries, IDMs, and memory makers that consume them. Suppliers are **circles**. Sole-source nodes (Ajinomoto for ABF film, Hoya + AGC for EUV mask blanks, Mitsui Chemicals for EUV pellicles, Shin-Etsu + SUMCO for 300mm wafers, JSR + TOK for EUV resist) are visually emphasized via the `single_source_risk` flag.

### Page 2 — Foundries + Equipment + Design + Customers

The full chip-creation diagram, with distinct shape types per the user's spec:

| node_type | count | shape | examples |
|---|---|---|---|
| `foundry` | 11 | large circle | TSMC, Samsung Foundry, Intel, SMIC, GlobalFoundries, UMC |
| `equipment` | 52 | **diamond** | ASML, AMAT, Lam, TEL, KLA, Disco, BESI, Hanmi, FormFactor |
| `eda` | 5 | hexagon | Synopsys (incl. Ansys), Cadence, Siemens EDA, Empyrean, Keysight |
| `ip` | 6 | small hexagon | Arm, Imagination, Ceva, SiFive, Andes, Rambus |
| `osat` | 10 | trapezoid | ASE, Amkor, JCET, Tongfu, PTI, Huatian, KYEC, ChipMOS |
| `memory` | 8 | rounded square | SK hynix, Micron, Kioxia, ChangXin, YMTC, Western Digital, Winbond, Nanya |
| `idm` | 10 | square | Intel, TI, Infineon, ST, Renesas, NXP, ADI, Microchip, ON, Sony Semi |
| `fabless` | 17 | small circle | NVIDIA, AMD, Apple, Broadcom, Qualcomm, MediaTek, Marvell, Alchip, Realtek, Cirrus Logic, Lattice, MaxLinear, Ambarella, Cerebras, Groq, Socionext, HiSilicon |
| `oem` | 5 | rounded rectangle | Tesla, Microsoft Azure, Google TPU, AWS, Meta |
| `supplier` | 115 | circle | (page 1 + a few cross-page like Ajinomoto, Resonac) |

Equipment makers explicitly use a **different shape** (diamond) from consumable suppliers (circle) per the user's distinction — they sell tools, not chemicals.

---

## Schema

```jsonc
{
  "metadata": {
    "version": "0.6.1",
    "...": "..."
  },
  "materials": [
    {
      "id": "mat_eu_resist",
      "name": "EUV Photoresist",
      "category": "Photolithography",
      "process_stage": "litho",
      "criticality": "high",
      "notes": "..."
    }
  ],
  "companies": [
    {
      "id": "co_tsmc",
      "node_type": "foundry",
      "page": 2,                          // 1 | 2 | "both"
      "name": "TSMC (Taiwan Semiconductor)",
      "country": "Taiwan",
      "hq_city": "Hsinchu",
      "lat": 24.78, "lng": 120.99,
      "geo_cluster_id": "metro_hsinchu",   // metro-area grouping for de-clustering
      "tier": 1,
      "categories": ["Foundry", "Advanced Packaging"],
      "tickers": ["TSM", "2330.TW"],
      "approx_revenue_usd_b": 87.7,
      "scale_indicator": 100,
      "market_positions": ["~64% global foundry revenue", "sole 3nm volume producer"],
      "key_facilities": [
        {"site": "Fab 18 (Tainan)", "node": "5/3nm", "country": "Taiwan"}
      ],
      "single_source_risk": true,         // optional flag
      "geographic_risk": "Taiwan strait",  // optional flag
      "export_control_exposed": true,      // optional flag
      "notes": "..."
    }
  ],
  "relationships": [
    {
      "id": "rel_0042",
      "supplier_id": "co_asml",
      "customer_id": "co_tsmc",
      "material_id": "mat_euv_scanner",
      "strength": "high",                 // high | medium | low | estimated
      "evidence": "ASML 10-K + TSMC capex disclosures",
      "revenue_weight_pct_2024": 24,      // optional, customer-side weight
      "revenue_weight_pct_2026e": 26,     // optional forward
      "revenue_weight_pct_hbm_2025": 27,  // optional HBM-specific
      "cowos_2026_pct": 60                // optional CoWoS allocation
    }
  ]
}
```

---

## Top 10 by total degree (centrality validation)

| Rank | Company | Degree |
|---|---|---|
| 1 | TSMC | 161 |
| 2 | Samsung Foundry | 97 |
| 3 | Intel (incl. Intel Foundry) | 86 |
| 4 | Synopsys | 84 |
| 5 | Cadence Design Systems | 78 |
| 6 | SK hynix | 74 |
| 7 | Applied Materials (AMAT) | 57 |
| 8 | Siemens EDA | 56 |
| 9 | SMIC | 39 |
| 10 | NVIDIA | 38 |

This is the expected shape: foundries dominate the center; the EDA triopoly (Synopsys, Cadence, Siemens EDA) ranks alongside top equipment makers; SK hynix's HBM centrality lifts it above all other memory makers.

---

## Customer revenue weights (where captured)

The `revenue_weight_pct_*` fields on relationships encode customer-side weights — i.e. how much of supplier X's revenue comes from customer Y. Where data is publicly available:

**TSMC customers (2024, % of TSMC revenue):**
Apple ~24%, Broadcom ~12%, NVIDIA ~11%, MediaTek ~9%, Qualcomm ~8%, AMD ~7%, Intel ~7%, Marvell ~3%

**TSMC CoWoS-L 2026 allocation (advanced packaging, by customer):**
NVIDIA ~60%, Broadcom ~15%, AMD ~11%, Google TPU ~5%, AWS ~4%, Meta ~3%, Microsoft Azure ~2%

**SK hynix HBM (Q2 2025):**
~62% global HBM share. NVIDIA receives ~90% of SK hynix HBM output, making HBM ~27% of total SK hynix revenue in 1H 2025.

**HBM market shares (Q2 2025):**
SK hynix 62%, Micron 21%, Samsung 17%

**HBM TC bonders (SK hynix supply mix, 2024 → 2026):**
Hanmi declining 100% → 30%, Hanwha Semitech rising to ~45%, ASMPT rising to ~50% by 2026 (Hanmi-Hanwha patent dispute ongoing).

---

## Risk flags

### `single_source_risk: true` (16 nodes)
ASML (EUV scanners), Zeiss SMT (EUV optics), Trumpf (EUV laser), Lasertec (EUV mask inspection), NuFlare (mask writers), Disco (precision dicing), Ajinomoto (ABF film), Mitsui Chemicals (EUV pellicle membranes), Hoya + AGC (EUV mask blanks), Shin-Etsu + SUMCO (300mm wafers), JSR + TOK (EUV resist), Arm (CPU IP), BESI (hybrid bonder for HBM4).

### `geographic_risk` (9 nodes)
Taiwan strait — TSMC, UMC, Vanguard, Powerchip, ASE, KYEC.
Korean peninsula — Samsung Foundry, SK hynix.
Japan seismic — Shin-Etsu / Mitsui / Kioxia (called out in notes).

### `export_control_exposed` (10 nodes)
ASML, Applied Materials, Lam Research, KLA, Tokyo Electron, Synopsys, Cadence, Siemens EDA, Arm, Empyrean (added to US Entity List Dec 2024).

---

## Coverage by category

**Raw materials & specialty chemicals:** silicon wafers (Shin-Etsu, SUMCO, GlobalWafers, Siltronic, SK Siltron, Wafer Works), polysilicon (Wacker, OCI, Tokuyama, GCL, Daqo, Xinte, TBEA), photoresists & ancillaries (JSR, TOK, Shin-Etsu, Sumitomo, Fujifilm, Merck/EMD, DuPont), wet chemicals & slurry (BASF, Cabot, DuPont, Fujimi, Versum, Resonac, Stella), specialty gases (Linde, Air Liquide, Air Products, Taiyo Nippon Sanso, Messer, Versum, SK Materials, Kanto Denka, Showa Denko/Resonac), ALD/CVD precursors (Adeka, Tri Chemical, UP Chemical, Air Liquide Electronics, SK Trichem), sputter targets (JX, Tosoh, Materion, Mitsui Mining), masks & blanks (Hoya, AGC, Photronics, Toppan, DNP, SK-Electronics), bonding & assembly (Heraeus, Tanaka, Sumitomo Metal Mining, Possehl, Mitsui High-tec, Enomoto, Shinko, Hitachi Metals/Proterial), substrates (Unimicron, Ibiden, AT&S, Nan Ya PCB, Shinko, Kinsus, SEMCO, Corning), rare earths (Lynas, MP Materials, China Rare Earth Group, Shenghe, JL Mag, Ningbo Yunsheng).

**Equipment (52 nodes — diamond-shaped):** Big-5 (ASML, AMAT, Lam, TEL, KLA), Japanese specialty (SCREEN, Hitachi High-Tech, Kokusai, Tokyo Seimitsu/Accretech, NuFlare, Lasertec, Advantest, Teradyne, Disco, Ebara, Daifuku, Yaskawa), bonding/packaging (BESI, ASMPT, Hanmi, Hanwha Semitech, Shinkawa, Kulicke & Soffa, Ajinomoto Fine-Techno), test (FormFactor, Technoprobe, MJC, JEM, MPI, Cohu, Yamaichi, Enplas, LEENO, ISC, Yokowo, Smiths Interconnect, Johnstech), Chinese equipment (NAURA, AMEC, Piotech, ACM Research, SMEE, Kingsemi, Hwatsing, JSGSEMI), ASML's own sub-suppliers (Zeiss SMT, Trumpf, Cymer/IPG components).

**EDA & IP triopoly:**
- Synopsys 31% global EDA share (~$6.1B FY24, +Ansys $35B acq Jul 2025).
- Cadence 30% (~$4.6B FY24).
- Siemens EDA 13%.
- Empyrean 5% (China, on Entity List Dec 2024).
- IP: Arm ~40% IP share (post-IPO Sep 2023), Imagination, Ceva, SiFive, Andes, Rambus.

**HBM ecosystem (full stack captured):**
HBM makers (SK hynix, Micron, Samsung) → HBM TC bonders (Hanmi, Hanwha, BESI, ASMPT, Shinkawa) → MR-MUF (Resonac/Showa Denko) → underfill/molding compounds (Namics, Henkel) → CoWoS at TSMC → AI accelerator customers (NVIDIA, AMD, Google TPU, AWS Trainium, Microsoft Azure, Meta).

**ABF substrate cartel + Ajinomoto chokepoint:**
Ajinomoto licenses ABF film IP to all 7 major substrate makers (Unimicron, Ibiden, AT&S, Nan Ya PCB, Shinko, Kinsus, SEMCO). Single-point-of-failure for advanced packaging.

---

## Files

| File | Purpose |
|---|---|
| `semicon_supply_chain.json` | Master dataset (v0.6.1) — single source of truth |
| `README.md` | This document |
| `expand_pass1.py` … `expand_pass6_polish.py` | Reproducible build scripts (one per pass) |
| `patch_risk_flags.py` | Applied risk flag annotations |

To rebuild from scratch: run `expand_pass1.py` → `expand_pass2.py` → … → `expand_pass6_polish.py` in order.

---

## Sourcing & methodology

Primary sources include company 10-Ks and annual reports (TSMC 2022 & 2024, ASML, Synopsys, Cadence, SK hynix newsroom, Arm IPO docs), industry analysts (TrendForce, Counterpoint, TECHCET, Bernreuter, Astute Group, SemiAnalysis), and Korean / Taiwanese trade press (The Elec, DigiTimes).

Chinese supply chain data is flagged with `strength: "estimated"` due to lower transparency. All quotes from sources are kept short and paraphrased; no source is quoted more than once.

`relationships[].evidence` records the basis for each edge (10-K, analyst report, press release, etc.) so the graph can be audited.

---

## Known limitations

- Western Digital / SanDisk has limited inbound edges; modeled mostly via Kioxia JV.
- Customer revenue weights are most complete for TSMC; Samsung Foundry and Intel Foundry have only partial public disclosure.
- Passive components (MLCC: Murata, TDK, SEMCO) and AI networking optics (Coherent, Lumentum) are out of scope for v0.6.2.
- Some Chinese equipment makers' actual customer mix is opaque; edges modeled with `strength: "estimated"`.

---

## v0.6.2 changelog

Pre-front-end render-readiness pass:
- Removed incorrect Sony Semi → Samsung Foundry "CMOS Image Sensor" edge (Sony is a CIS competitor, not a supplier to Samsung Foundry).
- Downgraded Ceva → Qualcomm DSP IP edge to `strength: "estimated"` (Qualcomm uses its own Hexagon DSP).
- Added `geo_cluster_id` to every company — metro-area grouping with 69 clusters (e.g. `metro_tokyo` covers Greater Kanto with 51 companies). Front end can de-cluster these into expandable bubbles instead of overlapping at HQ coordinates.
- Filled missing `scale_indicator` on Mitsubishi Materials.
- Dropped 5 unused material taxonomy entries (BT Substrate, Foundry Service Mature, Advanced Packaging service, Ultra-high-density PCB, TSV Equipment).
