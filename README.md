# Flash Translation Layer (FTL) Simulator

A C simulation of an SSD's Flash Translation Layer — modeling NAND flash constraints, logical-to-physical address mapping, garbage collection, and wear leveling, with instrumentation for write amplification and storage utilization.

## Overview

NAND flash cannot be overwritten in place: writes happen at page granularity, but erases only happen at block granularity, and each block tolerates a finite number of erase cycles. An FTL is the firmware layer that hides these constraints from the OS, making flash behave like a normal freely-overwritable block device.

This project implements that layer in software: a simulated NAND array enforcing physical program/erase rules, an L2P (logical-to-physical) mapping table, out-of-place writes, garbage collection, and wear-leveling-aware block allocation — driven by a configurable read/write workload and reporting on write amplification, erase counts, and utilization.

## Features

- **NAND flash model** — page-level state machine (`FREE` → `VALID` → `INVALID`), block-level erase, illegal-overwrite prevention
- **Logical-to-Physical mapping** — page-level L2P table with `ftl_read(lba)` / `ftl_write(lba, data)` API
- **Out-of-place updates** — writes never overwrite in place; stale pages are marked invalid, not reused directly
- **Garbage collection** — victim block selection (fewest valid pages), valid-page migration, block erase and reclamation
- **Wear leveling** — erase-count tracking per block, allocation biased to even out wear
- **Workload-driven simulation** — feed a trace of LBA read/write operations and observe FTL behavior
- **Metrics** — write amplification (physical writes ÷ logical writes), per-block erase counts, storage utilization

## Architecture

```
ftl-sim/
├── Makefile
├── src/
│   ├── main.c       # workload driver / CLI entry point
│   ├── nand.c/.h    # NAND flash model: pages, blocks, program/erase
│   ├── ftl.c/.h     # L2P mapping table, address translation
│   ├── gc.c/.h      # garbage collection, victim selection
│   ├── stats.c/.h   # write amplification, erase count, utilization tracking
├── workloads/       # sample read/write traces
└── README.md
```

**Simulated flash geometry:** 16 blocks × 4 pages/block = 64 pages total (configurable via constants in `nand.h`).

## Building

```bash
git clone https://github.com/<your-username>/ftl-simulator.git
cd ftl-simulator
make
```

## Usage

```bash
./ftl_sim workloads/sample_trace.txt
```

Each line of a workload trace is a single operation:

```
W <lba> <data>     # write
R <lba>            # read
```

On completion, the simulator prints final flash state and summary statistics (write amplification, total erases per block, utilization).

## Example output

```
[GC] Reclaiming block 3 (2 valid pages migrated)
[STATS] Logical writes: 128 | Physical writes: 171
[STATS] Write amplification: 1.34
[STATS] Total erases: 12
[STATS] Storage utilization: 78%
```

## Why this project

Real SSD controllers are judged largely on how well their FTL minimizes write amplification while spreading wear evenly — it's a direct measure of both performance and drive longevity. This simulator reproduces the core algorithmic tradeoffs of that firmware layer (mapping, GC, wear leveling) at a small, inspectable scale, without requiring real hardware.

## Roadmap

- [ ] Hybrid (block + page level) mapping for larger address spaces
- [ ] Configurable GC trigger threshold
- [ ] Multiple wear-leveling policies (static vs. dynamic)
- [ ] Trace-driven benchmarking against realistic workloads

## License

MIT
