# ParaLSM

A GPU-accelerated in-memory LSM-Tree key-value store with Size-Tiered Compaction, CUDA Merge Path, and OpenMP concurrency.

**CMU 15-418/618 Final Project, Spring 2026**

**Team:** Hua-Yo Wu(huayow), Pi-Jung Chang(pijungc)

**[Project Page](https://neo1202.github.io/ParaLSM/)**

---

## Overview

ParaLSM is an in-memory LSM-Tree using **Size-Tiered Compaction** that parallelizes compaction via **CUDA Merge Path** (two-phase: Global Memory partitioning + Shared Memory merging) with **chunk-based pipelining** to hide PCIe latency, and serves concurrent read/write requests via **OpenMP** task scheduling.

Size-Tiered compaction produces exponentially growing SSTables at deeper levels, providing the large, contiguous merge workloads that maximize GPU throughput. All data lives in pre-allocated memory pools — no disk I/O, no runtime allocation — so we isolate and measure pure computational parallelism.

## Architecture

```
CPU (8 cores) — OpenMP Tasks             GPU (RTX 2080) — Chunk Pipeline
┌──────────────────────────────┐         ┌───────────────────────────┐
│ Task pool (dynamic):         │         │ Cascaded 2-way merge:     │
│   ├── get/put requests       │         │   Stream 0: merge A+B     │
│   ├── flush (memtable→SST)   │         │   Stream 1: merge C+D     │
│   └── compaction dispatch ───┼────────►│   Stream 2: merge AB+CD   │
│                              │         │                           │
│ Co-rank partitioning (CPU)   │         │ Each stream: overlap      │
│ Pinned memory buffers        │         │   H2D / Kernel / D2H      │
└──────────────────────────────┘         └───────────────────────────┘
```

## Building

```bash
mkdir build && cd build
cmake ..
make -j$(nproc)
```

### Dependencies

- C++20 compiler (g++ 10+)
- CUDA Toolkit 11+
- OpenMP
- (Stretch) CPU with AVX2 support

## Usage

```bash
# Run all tests
./build/test_lsm

# Run benchmarks
./build/bench_merge --size 10000000
./build/bench_system --workload mixed --threads 8
```

## Project Structure

```
ParaLSM/
├── docs/               # GitHub Pages project website
│   └── index.html
├── src/
│   ├── memtable.h      # Active + immutable memtable
│   ├── sstable.h       # Sorted array (in-memory SSTable)
│   ├── lsm_tree.h      # Core LSM-Tree logic (size-tiered compaction)
│   ├── version.h       # Atomic version snapshot (std::atomic<shared_ptr>)
│   ├── memory_pool.h   # Pre-allocated host/device memory pools
│   ├── merge.h          # Serial merge baseline
│   ├── merge.cu         # CUDA merge path kernel (shared memory two-phase)
│   └── pipeline.cu      # Chunk-based pipeline with Co-rank partitioning
├── bench/
│   ├── bench_merge.cpp  # Merge micro-benchmark (H2D/kernel/D2H breakdown)
│   └── bench_system.cpp # End-to-end YCSB-style workload benchmark
├── test/
│   └── test_lsm.cpp    # Correctness tests
├── CMakeLists.txt
├── .gitignore
└── README.md
```

## References

- Odeh, Green, Miklau. *Merge Path — A Visually Intuitive Approach to Parallel Merging.* IEEE IPDPS, 2012.
- O'Neil et al. *The Log-Structured Merge-Tree (LSM-Tree).* Acta Informatica, 1996.
