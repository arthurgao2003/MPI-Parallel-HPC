# MPI-Parallel-Histogram: High-Performance Distributed Computing

This project implements a high-performance parallel histogram generator using **C** and **MPI (Message Passing Interface)**. It is designed to process massive datasets (up to 10 billion items) across a distributed compute cluster by leveraging parallel data decomposition and reduction strategies.

## 🏎️ Parallel Architecture & Logic
The system optimizes data processing by distributing the workload across multiple CPU ranks (processes) to minimize total computation time and maximize memory throughput.

### 1. Data Distribution (Scatter Phase)
* **Load Balancing**: Since the dataset size ($N$) may not be perfectly divisible by the number of processes ($P$), the implementation manually calculates `sendcounts` and `displs` (displacements).
* **Irregular Partitioning**: Utilizes `MPI_Scatterv` to handle remainder data points, ensuring the workload is distributed as evenly as possible across the cluster.

### 2. Local Histogram Computation
Each process independently computes a local histogram for its assigned data chunk:
* **Quantization**: Data items in the range $[0.0, 20.0)$ are mapped to specific bins using a calculated `binrange`.
* **Parallel Execution**: Local bins are allocated and processed in parallel across all ranks, drastically reducing execution time compared to serial implementations.

### 3. Global Aggregation (Reduce Phase)
* **MPI_Reduce**: Once local histograms are complete, the system performs a global **Sum Reduction**. All local bin counts are aggregated into a final global histogram stored exclusively on **Process 0**.
* **Synchronization**: A global `MPI_Barrier` is utilized to ensure precise wall-clock timing measurements of the parallel execution.

## 📊 Performance & Benchmarking
* **Timing**: Utilizes `MPI_Wtime()` to measure the precise duration of the parallel section.
* **Scalability**: Designed to scale with the number of processes. The final elapsed time is captured using the `MPI_MAX` operator to identify the bottleneck (the slowest process).
* **Validation**: Includes a `check_correctness` function that validates the parallel output against a serial reference histogram to ensure numerical accuracy and data integrity.

## 🛠 Tech Stack
* **Language**: C
* **Framework**: MPI (Message Passing Interface)
* **Tools**: High-Performance Computing (HPC) Clusters, GCC
* **Concepts**: Distributed Memory, Load Balancing, Collective Communication (`Scatterv`, `Reduce`), Data Decomposition.

## 🔧 How to Run
Compile the code with an MPI wrapper and execute it by specifying the number of processes, total data items, and number of bins:

```bash
# Compile
mpicc -o histogram lab1.c

# Run with 4 processes
# Arguments: [Number of items] [Number of bins]
mpirun -np 4 ./histogram 1000000 10
