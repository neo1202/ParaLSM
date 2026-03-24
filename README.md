# ParaLSM

A GPU-accelerated in-memory LSM-Tree key-value store with CUDA, OpenMP, and SIMD parallelism.

**CMU 15-418/618 Final Project, Spring 2026**

**Team:** [Your Name], [Partner Name]

**[Project Page](https://YOUR_USERNAME.github.io/ParaLSM/)**

---

## Overview

ParaLSM is an in-memory LSM-Tree that parallelizes compaction (sorted array merge) using CUDA merge path, serves concurrent read/write requests via OpenMP thread pools, and accelerates key comparisons with AVX2 SIMD intrinsics.

All data lives in memory — no disk I/O — so we can isolate and measure pure computational parallelism.

## Architecture

```
CPU (8 cores)                          GPU (RTX 2080)
┌────────────────────────────┐         ┌─────────────────────┐
│ Core 0-5: OpenMP thread    │         │ Stream 0: merge     │
│   pool (get/put requests)  │         │ Stream 1: merge     │
│ Core 6: flush thread       │         │ Stream 2: transfer  │
│ Core 7: compaction dispatch├────────►│                     │
└────────────────────────────┘         └─────────────────────┘
```

## Building

```bash
mkdir build && cd build
cmake ..
make -j$(nproc)
```

### Dependencies

- C++17 compiler (g++ 9+)
- CUDA Toolkit 11+
- OpenMP
- CPU with AVX2 support

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
├── docs/              # GitHub Pages project website
│   └── index.html
├── src/
│   ├── memtable.h     # Active + immutable memtable
│   ├── sstable.h      # Sorted array (in-memory SSTable)
│   ├── lsm_tree.h     # Core LSM-Tree logic
│   ├── version.h      # Lock-free version snapshot
│   ├── merge.h         # Serial merge baseline
│   ├── merge.cu        # CUDA merge path kernel
│   └── merge_simd.h    # AVX2 merge + comparison
├── bench/
│   ├── bench_merge.cpp # Merge micro-benchmark
│   └── bench_system.cpp# End-to-end workload benchmark
├── test/
│   └── test_lsm.cpp   # Correctness tests
├── CMakeLists.txt
├── .gitignore
└── README.md
```

## References

- Odeh, Green, Miklau. *Merge Path — A Visually Intuitive Approach to Parallel Merging.* IEEE IPDPS, 2012.
- O'Neil et al. *The Log-Structured Merge-Tree (LSM-Tree).* Acta Informatica, 1996.
