# Diva
[![Build and Run Unit Tests](https://github.com/n3slami/Diva/actions/workflows/build_and_test.yml/badge.svg)](https://github.com/n3slami/Diva/actions/workflows/build_and_test.yml)

Diva is the first range filter to simultaneously support **d**ynam**i**c
operations, **va**riable-length keys, range queries of any length, and high
performance while providing a theoretical false positive rate guarantee. This
false positive guarantee holds when the input data comes from a "well-behaved"
distribution, which encompasses the common distributions seen in practice.

Our paper describes the inner workings of Diva in detail. It also presents
an in-depth theoretical analysis as well as empricial evaluations.

# Getting Started
To use Diva in your own project, simply add the files in the `include`
directory and include the `diva.hpp` header file.

The file `examples/example.cpp` presents an example of how to use Diva's APIs.

# Building
The following software are prerequisites for building Diva and its evaluation
suite:
- cmake 3.5 (or later)
- gcc-11 (or later)
- Boost 1.67.0 (or later)
- Git 2.13 (or later)
- Python 3.8 (or later)
- Bash 4.4 (or later)
  - realpath 8.28 (or later)
  - wget 1.19.4 (or later)
  - zsdt 0.0.1 (or later)
  - md5sum 8.28 (or later)

To build Diva with its examples, unit tests, and benchmarks, navigate to the
project's root directory and use 
```Bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . -j8
```
You can control which parts are configured and compiled with the following
CMake options:
- `BUILD_TESTS`: Builds the tests.
- `BUILD_EXAMPLES`: Builds the examples.
- `BUILD_BENCHMARKS`: Builds the benchmark suite. It will also clone the
  repositories of the baselines and WiredTiger, followed by building
  WiredTiger, if not done previously.
All these options are set by default. To turn one off, for example,
`BUILD_BENCHMARKS`, substitute the second line in the build script above with:
```Bash
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_BENCHMARKS=0
```

# Global Slab (multi-instance optimization)

By default, each Wormhole instance maintains its own slab allocator. In applications that host many instances (e.g., an LSM tree where each sorted run builds a separate Diva), this per-instance design adds measurable setup/teardown overhead.

The **global slab** optimization lets all active Wormhole instances share a single, process-wide slab allocator. The shared allocator is created on the first Wormhole initialization and destroyed after the last instance is torn down, amortizing allocator lifecycle costs across instances.

Enable this feature with the following preprocessor macros:

```c++
// Enable a process-wide shared slab allocator for all Wormhole instances.
#define WORMHOLE_GLOBAL_SLAB

// Disable QSBR (Quiescent State Based Reclamation) and use a read–write lock instead.
#define WORMHOLE_DISABLE_QSBR
```

### Notes

* **`WORMHOLE_GLOBAL_SLAB`**: Activates the shared allocator so multiple instances reuse the same slabs instead of each managing its own.

* **`WORMHOLE_DISABLE_QSBR`**: Turns off QSBR and switches synchronization to a read–write lock. This is useful when instances are effectively **read-only** or **write-only** and do not require concurrent reclamation (e.g., RocksDB’s `FilterBitsReader` and `FilterBitsWriter` operate on separate Diva instances).

To fully enable the Global Slab optimization, define **both** `WORMHOLE_GLOBAL_SLAB` and `WORMHOLE_DISABLE_QSBR`.

# Running Unit Tests
After building Diva, run the following command from the project's root
directory to execute the unit tests:
```Bash
ctest -VV
```

# Benchmarks
We advise to use a Linux machine with at least 64GB of RAM to run our
benchmarks. Note that the following is required for plotting the experiment
results:
- Python 3.8 (or later)
  - Matplotlib 3.2.1 (or later)
- Latex full (for generating the plots)

## TL;DR
You can automate all the following steps by running the `evaluate.sh` script
from the project's root directory: 
```Bash
bash evaluate.sh
```
The plots from the experiments will be placed in the `paper_results/figures`,
with the `paper_results` neighboring the project's root directory. Pictorially,
the directory structure will be like so:


## Setting up the Environment
To keep the project clean and separate out the datasets from the code, we
create a work directory. That is, from the project's root directory, we use the
following command to create a `paper_results` directory and navigate into it:
```Bash
cd .. && mkdir paper_results && cd paper_results
```
In what follows, we refer to the name of the root directory of the project by
`<project-root>`.

## Downloading and Generating Datasets and Workloads
From the newly created directory, we run the following commands to download the
datasets used and generate our workloads:
```Bash
bash ../<project-root>/bench/scripts/download_datasets.sh
bash ../<project-root>/bench/scripts/generate_datasets.sh ../<project-root>/build real_datasets
```

## Running the Benchmarks
Next, we run the `run_benchmarks.py` Python script to run all benchmarks and
gather their data:
```Bash
python3 ../<project-root>/bench/scripts/run_benchmarks.py ../<project-root>/build workloads
```
You can also change which experiments are run using command-line options. For
more information, run 
```Bash
python3 ../<project-root>/bench/scripts/run_benchmarks.py -h
```

## Plotting
Finally, once the results are gathered, the following command will generate
plots from the experimental data gathered:
```Bash
python3 ../<project-root>/bench/scripts/plot.py
```
You can also change the directory where this script reads the experiment data from,
the directory where it saves its plots in, and which plots it generates by
using command-line options. For more information, run 
```Bash
python3 ../<project-root>/bench/scripts/plot.py -h
```

# Citation
Please use the following BibTeX entry to cite our paper presenting Diva:
```{bibtex}
@article{}
```

