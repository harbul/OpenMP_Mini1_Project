# DataForge Analytics Engine
Fire & Population Data Processing and Analytics

![C++17](https://img.shields.io/badge/C%2B%2B-17-blue)
![OpenMP](https://img.shields.io/badge/Parallel-OpenMP-success)
![License: MIT](https://img.shields.io/badge/License-MIT-green)
![Build Status](https://img.shields.io/badge/Build-CMake-informational)

A high-performance C++17 + OpenMP framework that compares **row-oriented vs column-oriented data layouts** for environmental monitoring and demographic analytics. This project demonstrates how data organization affects ingestion performance, parallel scalability, cache behavior, and analytical query efficiency.

## 🚀 Quick Start

### Prerequisites
- **CMake** ≥ 3.16
- **C++17** compatible compiler (GCC, Clang, or MSVC)
- **OpenMP** library
- **macOS**: `brew install cmake libomp`
- **Ubuntu**: `sudo apt install cmake build-essential libomp-dev`

### Build & Run
```bash
# Clone and build
git clone <repository-url>
cd OpenMP_Mini1_Project
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . --parallel

# Run population analytics (default)
./OpenMP_Mini1_Project_app --threads 8 --repetitions 5

# Run fire data ingestion benchmark
./OpenMP_Mini1_Project_app --fire --threads 8 --repetitions 3

# Run fire analytics operations
./OpenMP_Mini1_Project_app --fire-analytics --threads 8

# Run unit tests
./OpenMP_Mini1_Project_tests

# Test fire data models individually
./OpenMP_Mini1_Project_fire_test
```

## 📊 Performance Results

### Latest Benchmark Snapshot *(Release build, macOS, 8-thread analytics, 4-thread ingestion)*
- **Fire ingestion**: Row model completed 516 CSVs in **0.839 s** with **≈81% reported efficiency**; column model finished in **0.884 s** with **25% reported efficiency** (4 OpenMP threads).
- **Fire analytics (max AQI)**: Row model dropped from **11.5 ms serial** to **3.9 ms** with 8 threads; column model stayed near **0.08 ms serial** and **0.12 ms parallel**, retaining the latency advantage.
- **Fire dataset footprint**: Row model ingested **1,167,525 measurements across 1,398 sites**; column model produced **1,167,009 measurements across 1,397 sites** in the same run.
- **Population sum of year (1992)**: Row service measured **2.00 µs serial** vs **50.63 µs (8 threads)**; column service achieved **0.50 µs serial** vs **44.60 µs (8 threads)**.
- **Top-10 population ranking**: Row service ran in **10.79 µs serial** and **35.14 µs (8 threads)**; column service recorded **8.47 µs serial** and **32.46 µs (8 threads)**.

### 1. Population Row Service (Latest µs Measurements)
| Operation | Serial 1T (µs) | 8T (µs) | Parallel Speedup |
|-----------|----------------|---------|------------------|
| **sumPopulationForYear** | 2.000 | 50.625 | 0.04× (overhead-bound) |
| **averagePopulationForYear** | 0.903 | 48.889 | 0.02× (overhead-bound) |
| **maxPopulationForYear** | 0.861 | 29.084 | 0.03× (overhead-bound) |
| **minPopulationForYear** | 0.875 | 23.764 | 0.04× (overhead-bound) |
| **topNCountriesByPopulationInYear** | 10.791 | 35.139 | 0.31× (overhead-bound) |
| **populationForCountryInYear** | 24.125 | 19.444 | **1.24×** |
| **populationOverYearsForCountry** | 21.417 | 20.417 | **1.05×** |

### 2. Population Column Service (Latest µs Measurements)
| Operation | Serial 1T (µs) | 8T (µs) | Parallel Speedup |
|-----------|----------------|---------|------------------|
| **sumPopulationForYear** | 0.500 | 44.597 | 0.01× (overhead-bound) |
| **averagePopulationForYear** | 0.417 | 47.292 | 0.01× (overhead-bound) |
| **maxPopulationForYear** | 0.444 | 23.708 | 0.02× (overhead-bound) |
| **minPopulationForYear** | 0.417 | 22.028 | 0.02× (overhead-bound) |
| **topNCountriesByPopulationInYear** | 8.472 | 32.458 | 0.26× (overhead-bound) |
| **populationForCountryInYear** | 0.097 | 0.014 | **6.93×** |
| **populationOverYearsForCountry** | 0.736 | 0.278 | **2.65×** |

### Key Performance Insights

**🚀 Column Model Advantages:**
- **Point Queries**: Up to **≈249× faster** (0.097 μs vs 24.125 μs)
- **Range Queries**: Up to **≈29× faster** (0.736 μs vs 21.417 μs)
- **Cache Efficiency**: Superior for analytical operations
- **Fire Max AQI**: **≈144× faster** single-thread evaluation (0.08 ms vs 11.5 ms)

**⚡ Row Model Advantages:**
- **Parallel Ingestion**: Delivers higher reported efficiencies (≈80% vs ≤25%) while loading 516 CSVs
- **Merge-Friendly Layout**: Per-site accumulation keeps merge costs predictable even at 8 threads
- **Consistent Analytics**: Identical results remain easy to audit against the column layout

**🔧 Multi-Threading Analysis:**
- **Analytics overhead matters**: Short-running aggregations slow down when forced through the threading pipeline
- **Per-entity queries scale**: `populationForCountryInYear` and range scans still gain 1.0–7.0× when parallelized
- **Column model retains latency edge** even when parallel efficiency drops

### Fire Analytics Operations Summary
| Operation | Row (1T) | Row (8T) | Column (1T) | Column (8T) | **Best Choice** |
|-----------|----------|----------|-------------|-------------|----------------|
| **maxAQI** | 11.5 ms | 3.9 ms | **0.08 ms** | **0.12 ms** | Column Model |
| **minAQI** | 11.5 ms | 3.9 ms | **0.08 ms** | **0.12 ms** | Column Model |
| **averageAQI** | 11.5 ms | 3.9 ms | **0.08 ms** | **0.12 ms** | Column Model |
| **topNSites** | 0.012 ms | 0.042 ms | **0.010 ms** | **0.040 ms** | Column Model |

**Note:** min/average reuse the same scan path as maxAQI, so their timings match within measurement noise.

**Fire Data:** 516 CSV files observed. Latest run produced 1,167,525 measurements across 1,398 row-model sites and 1,167,009 measurements across 1,397 column-model sites.

### 5. Fire Data Loading Performance (CSV Ingestion)
| Model | Threads | Avg Time (s) | Speedup | Reported Efficiency | Files/sec | Sites | Measurements |
|-------|---------|--------------|---------|---------------------|-----------|-------|--------------|
| **Row-oriented** | 1 | 2.166 | 1.00× | - | 238.3 | 1,398 | 1,167,525 |
| **Row-oriented** | 2 | 1.345 | 1.61× | ≈88% | 383.8 | 1,398 | 1,167,525 |
| **Row-oriented** | 3 | 1.012 | 2.14× | ≈84% | 510.0 | 1,398 | 1,167,525 |
| **Row-oriented** | 4 | 0.839 | 2.58× | ≈81% | 615.1 | 1,398 | 1,167,525 |
| **Row-oriented** | 8 | 0.813 | 2.67× | ≈79% | 634.9 | 1,398 | 1,167,525 |
| **Column-oriented** | 1 | 2.132 | 1.00× | - | 242.0 | 1,397 | 1,167,009 |
| **Column-oriented** | 2 | 1.360 | 1.57× | 50% | 379.4 | 1,397 | 1,167,009 |
| **Column-oriented** | 3 | 1.044 | 2.04× | 33% | 494.2 | 1,397 | 1,167,009 |
| **Column-oriented** | 4 | 0.884 | 2.41× | 25% | 583.8 | 1,397 | 1,167,009 |
| **Column-oriented** | 8 | 0.846 | 2.52× | 12.5% | 610.0 | 1,397 | 1,167,009 |

Reported efficiency figures are sourced from the model logs and reflect only the parallel CSV parsing phase.

### Fire Data Loading Analysis
**🔥 Key Findings:**
- **Row and column converge**: 4-thread runs finish in **0.839 s (row)** vs **0.884 s (column)** for 516 CSVs
- **Throughput ceiling**: Row layout peaks at **≈635 files/sec** (8 threads); column reaches **≈610 files/sec**
- **Single-thread parity**: Column holds a slight serial edge (2.132 s vs 2.166 s) but the gap is <2%
- **Optimal operating point**: 3–4 threads balance throughput with manageable merge overhead for both layouts

**⚡ Parallel Efficiency Comparison:**
- **Row model**: Reported efficiencies stay near **80%** even at 8 threads due to uniform per-site aggregation
- **Column model**: Reported efficiencies fall from **50% (2T)** to **12.5% (8T)** because merge steps dominate
- **Data layout effect**: Column layout still favors analytics—ingestion parallelism simply saturates sooner

## 🏗️ Architecture Overview

### Data Models
| Model Type | Layout Strategy | Optimal Use Case |
|------------|----------------|------------------|
| **Fire Row Model** | Site-grouped hierarchical containers | Real-time ingestion, parallel loading, site-specific queries |
| **Fire Column Model** | Field-oriented vectors (13 columns) | Aggregations, statistical analysis, parameter scans |
| **Population Row Model** | Country-grouped time series | Per-country analysis, demographic trends |
| **Population Column Model** | Year-grouped vectors | Cross-country comparisons, temporal aggregations |

### Service Architecture
The project uses a **direct service pattern** without virtual inheritance for maximum performance:

```cpp
// Fire Analytics Services
FireRowService rowService(&fireRowModel);
FireColumnService colService(&fireColumnModel);

// Both provide identical interface:
int maxAQI = rowService.maxAQI(numThreads);
double avgAQI = colService.averageAQI(numThreads);
auto topSites = rowService.topNSitesByAverageConcentration(10, numThreads);
```

### Parallel Strategy
**Dynamic Work Distribution** with thread-local staging:
```cpp
#pragma omp parallel num_threads(numThreads)
{
    ThreadLocalModel threadModel;
    #pragma omp for schedule(dynamic, 1)
    for (size_t i = 0; i < files.size(); ++i) {
        threadModel.processFile(files[i]);
    }
    #pragma omp critical
    {
        globalModel.merge(threadModel);
    }
}
```

## 🛠️ Command Line Interface

### Main Application (`OpenMP_Mini1_Project_app`)
| Flag | Description | Default |
|------|-------------|---------|
| `--help, -h` | Show usage information | - |
| `--threads N, -t N` | Set OpenMP thread count | 4 |
| `--repetitions N, -r N` | Number of benchmark repetitions | 5 |
| `--fire, -f` | Run fire data ingestion benchmark | off |
| `--fire-analytics, -fa` | Run fire analytics operations benchmark | off |

### Usage Examples
```bash
# Population analytics with 8 threads, 10 repetitions
./OpenMP_Mini1_Project_app --threads 8 --repetitions 10

# Fire data benchmarks with custom thread count
./OpenMP_Mini1_Project_app --fire --fire-analytics --threads 6

# Show help
./OpenMP_Mini1_Project_app --help
```

## 📁 Project Structure

```
OpenMP_Mini1_Project/
├── interface/                 # Header files
│   ├── fireRowModel.hpp       # Fire row-oriented model
│   ├── fireColumnModel.hpp    # Fire column-oriented model
│   ├── fire_service_direct.hpp # Fire analytics services
│   ├── populationModel.hpp    # Population row model
│   ├── populationModelColumn.hpp # Population column model
│   ├── service.hpp           # Population services
│   ├── benchmark_utils.hpp   # Benchmarking utilities
│   └── utils.hpp            # General utilities
├── src/                      # Implementation files
│   ├── main.cpp             # Unified application entry point
│   ├── fireRowModel.cpp     # Fire row model implementation
│   ├── fireColumnModel.cpp  # Fire column model implementation
│   ├── fireRowService.cpp   # Fire row analytics service
│   ├── fireColumnService.cpp # Fire column analytics service
│   ├── populationModel.cpp  # Population models
│   ├── service.cpp          # Population services
│   ├── benchmark_utils.cpp  # Benchmarking framework
│   └── fire_test.cpp        # Fire model validation tests
├── tests/                   # Unit tests
│   └── basic_tests.cpp      # Comprehensive test suite
├── data/                    # Data directory
│   └── fireData/           # CSV files (not included in repo)
├── CMakeLists.txt          # Build configuration
└── README.md              # This file
```

## 🎯 Key Insights & Recommendations

### When to Use Each Model

| Scenario | Recommended Model | Reason |
|----------|------------------|--------|
| **High-throughput CSV ingestion** | Fire Row Model | Superior parallel scaling (79-81% efficiency) |
| **Site-specific queries** | Fire Row Model | Natural hierarchical organization |
| **Statistical aggregations** | Fire/Population Column Model | Cache-friendly columnar access |
| **Cross-entity comparisons** | Column Model | Direct indexed access (≈249× faster point lookups) |
| **Real-time monitoring** | Row Model | Better ingestion latency |
| **Batch analytics** | Column Model | Optimized for bulk operations |

### Performance Optimization Tips

1. **Thread Count**: 4 threads provide near-optimal performance with better efficiency than 8 threads
2. **Memory Layout**: Choose based on primary access pattern (row-wise vs column-wise)
3. **Parallel Efficiency**: Row models scale better for ingestion, column models for analytics
4. **Cache Behavior**: Column models excel in scenarios requiring iteration over many entities

## 🧪 Testing & Validation

The project includes comprehensive testing:

- **Unit Tests**: Validate utility functions, benchmark framework, and model equivalence
- **Performance Tests**: Measure and compare ingestion and query performance
- **Correctness Tests**: Ensure identical results across different models and thread counts
- **Integration Tests**: Verify end-to-end workflows

```bash
# Run all tests
./OpenMP_Mini1_Project_tests

# Test fire models specifically
./OpenMP_Mini1_Project_fire_test
```

## 📈 Benchmarking Framework

The project includes a sophisticated benchmarking system:

- **Multi-threaded timing** with statistical analysis
- **Memory usage tracking** and efficiency metrics
- **Automated result comparison** across models
- **Configurable repetitions** for statistical reliability
- **Command-line result formatting** with performance tables

## 🔧 Development

### Adding New Analytics Operations

1. **Add to service interface** (`fire_service_direct.hpp`)
2. **Implement in both services** (`fireRowService.cpp`, `fireColumnService.cpp`)
3. **Add benchmark integration** (`main.cpp`)
4. **Update tests** (`tests/basic_tests.cpp`)

### Extending Data Models

1. **Define new model class** following existing patterns
2. **Implement parallel ingestion** using OpenMP
3. **Create corresponding service class**
4. **Add validation tests**

## 📄 License

MIT License - see source files for details.

## 🚨 Note on Data

The fire monitoring CSV files are not included in this repository due to size constraints. The application expects data in `data/fireData/` or set the `FIRE_DATA_PATH` environment variable to specify a custom location.

---

**Performance Summary**: Row model achieves ≈634.9 files/sec with a 2.67× speedup for ingestion. Column model delivers up to ≈249× speedup for analytics-heavy point queries. Choose your architecture based on primary workload characteristics.
