# Spatial Engine

A fully hardware-based spatial accelerator built using pure digital logic in Proteus. Given a set of 2D points stored in EEPROM, the engine computes the closest point pair using Euclidean distance, resolves ties deterministically via a spatial metric, and displays the product of the winning pair's indices in BCD on a 7-segment display.

> **DigiSim'26** — Digital Electronics Event, EES, IIT BHU

---

## Problem Statement

Design a digital system (no microcontrollers) that:

1. Reads **n** points (4 ≤ n ≤ 8) with coordinates in the range [0, 16] from an EEPROM.
2. Finds the **closest pair** of points by Euclidean distance.
3. Breaks ties **deterministically** using a defined spatial metric.
4. Outputs the **product of the two point indices** in BCD on a 7-segment display.

---

## EEPROM Data Format

| Address | Content |
|---------|---------|
| `0x00`  | Number of points, **n** |
| `0x01 … 0x2n` | Encoded point data (one byte per coordinate entry) |

### Byte Encoding

```
Bit [7] [6] [5]   [4]        [3] [2] [1] [0]
    └─ Point Index ─┘  Axis Sel   └─ Coordinate ─┘
```

| Field | Bits | Description |
|-------|------|-------------|
| Coordinate | `[3:0]` | 4-bit value (0–16) |
| Axis Selector | `[4]` | `0` → X-coordinate, `1` → Y-coordinate |
| Point Index | `[7:5]` | Identifies which point this entry belongs to |

Each point is defined by exactly one X entry and one Y entry sharing the same index.

---

## Algorithm

### 1. Distance Computation

For every unique pair (i, j) the engine computes the **squared Euclidean distance**:

$$d_{ij}^2 = (x_i - x_j)^2 + (y_i - y_j)^2$$

The square root is unnecessary since only relative ordering matters.

### 2. Minimum Distance Selection

The pair with the smallest $d^2$ is selected. If multiple pairs share the same minimum distance, tie-breaking is applied.

### 3. Deterministic Tie Resolution

For tied pairs, the engine evaluates a spatial metric:

$$M_{ij} = (x_i \cdot y_i + x_j \cdot y_j) \mod (i \cdot j)$$

All indices and coordinates are guaranteed non-zero, so the modulo operation is always safe. The pair with the **smallest metric value** wins.

### 4. Output

The product of the selected pair's indices:

$$\text{Output} = i \times j$$

is converted to **BCD** and displayed on a **7-segment display**.

---

## Architecture Overview

The system is built entirely from combinational and sequential digital logic blocks — no microcontrollers or programmable processors are used.

```
┌──────────┐     ┌───────────────┐     ┌──────────────────┐
│  EEPROM  │────▶│  Data Decoder │────▶│  Point Registers │
│ (I²C/SPI)│     │  & Sequencer  │     │  X[i], Y[i]      │
└──────────┘     └───────────────┘     └────────┬─────────┘
                                                │
                                    ┌───────────▼───────────┐
                                    │  Pairwise Distance    │
                                    │  Computation Unit     │
                                    │  (subtract, square,   │
                                    │   accumulate)         │
                                    └───────────┬───────────┘
                                                │
                                    ┌───────────▼───────────┐
                                    │  Minimum Finder &     │
                                    │  Tie-Break Logic      │
                                    │  (metric comparator)  │
                                    └───────────┬───────────┘
                                                │
                                    ┌───────────▼───────────┐
                                    │  Index Multiplier     │
                                    │  & BCD Converter      │
                                    └───────────┬───────────┘
                                                │
                                    ┌───────────▼───────────┐
                                    │  7-Segment Display    │
                                    └───────────────────────┘
```

### Key Functional Blocks

| Block | Function |
|-------|----------|
| **Sequencer** | Drives EEPROM read addresses and routes decoded data to point registers |
| **Subtractor & Squarer** | Computes $(x_i - x_j)^2$ and $(y_i - y_j)^2$ for each pair |
| **Accumulator** | Sums squared differences to produce $d^2$ |
| **Comparator Tree** | Finds the minimum $d^2$ across all pairs |
| **Tie-Break Unit** | Evaluates the spatial metric $M_{ij}$ for tied pairs |
| **Multiplier** | Computes $i \times j$ for the winning pair |
| **Binary-to-BCD** | Converts the product to BCD for display |
| **7-Segment Driver** | Drives the display segments |

---

## How to Run

1. Open `Spatial Engine.pdsprj` in **Proteus Design Suite**.
2. Load the EEPROM with point data following the encoding format above.
3. Run the simulation.
4. The 7-segment display will show the BCD-encoded product of the closest pair's indices.

---

## Constraints

| Parameter | Range |
|-----------|-------|
| Number of points (n) | 4 – 8 |
| Coordinate values | 0 – 16 |
| Point indices | Non-zero |
| Coordinates | Non-zero (for tie-break safety) |

## License

This project was built as part of DigiSim'26, organised by the Electronics Engineering Society (EES), IIT BHU.
