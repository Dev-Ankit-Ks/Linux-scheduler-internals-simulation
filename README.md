# 🚀 CFS Scheduler Simulation

> *A sophisticated implementation of the Linux Completely Fair Scheduler (CFS) algorithm with real-time visualization and comprehensive analysis*

[![C++](https://img.shields.io/badge/C++-17-blue.svg)](https://isocpp.org/)
[![Python](https://img.shields.io/badge/Python-3.8+-green.svg)](https://python.org/)
[![CMake](https://img.shields.io/badge/CMake-3.10+-red.svg)](https://cmake.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 📖 Overview

The **Completely Fair Scheduler (CFS)** is the default process scheduler in the Linux kernel, designed to provide fair CPU time distribution among all running tasks. This project presents a detailed simulation that demonstrates the core principles of CFS through an interactive C++ implementation with Python-based visualization.

### 🎯 Key Features

- ✅ **Authentic CFS Implementation** - Faithful recreation of Linux kernel scheduling logic
- ✅ **Dual Task Support** - Comprehensive handling of CPU-bound and I/O-bound processes
- ✅ **Real-time Visualization** - Dynamic plotting of scheduling decisions and vruntime evolution  
- ✅ **Priority-based Scheduling** - Weighted fair queuing with configurable priorities
- ✅ **Performance Analytics** - Detailed metrics and scheduling behavior analysis

---

## 🧠 Core Concepts

### Virtual Runtime (vruntime)
The heart of CFS lies in its virtual runtime mechanism:

```
vruntime += (executed_time × NICE_0_LOAD) / weight
```

Where:
- **NICE_0_LOAD** = 1024 (normalization constant)
- **weight** = 1024 / (priority + 1)
- Tasks with **lower vruntime** get scheduled first

### Task Weighting System
Priority determines CPU share allocation:

| Priority | Weight | CPU Share |
|----------|--------|-----------|
| 0 (Highest) | 1024 | ~50% |
| 1 | 512 | ~25% |
| 2 | 341 | ~17% |
| 3 | 256 | ~12% |

### Red-Black Tree Scheduling
CFS maintains tasks in a self-balancing red-black tree ordered by vruntime, ensuring O(log n) insertion and deletion operations.

---

## 🔧 Architecture

### Task Types

#### 🖥️ CPU-Bound Tasks
```cpp
struct CPUTask {
    int pid;
    double vruntime;
    int cpu_burst_time;
    int priority;
    double weight;
};
```

**Characteristics:**
- Execute in continuous **1ms time slices**
- Minimal I/O operations
- vruntime grows linearly with execution time
- Higher priority = slower vruntime growth

#### 💾 I/O-Bound Tasks
```cpp
struct IOTask {
    int pid;
    double vruntime;
    int cpu_burst_time;
    int priority;
    double weight;
    int iowait_time = 10; // ms
};
```

**Characteristics:**
- Frequent **10ms I/O wait periods**
- Short **1ms CPU bursts** between I/O
- vruntime penalty during I/O wait
- Realistic simulation of interactive processes

### Scheduling Algorithm

```cpp
while (!runqueue.empty()) {
    Task* current_task = runqueue.top();
    runqueue.pop();
    
    if (current_task->type == IO_BOUND) {
        // Simulate I/O wait
        sleep(iowait_time);
        penalize_vruntime(current_task, iowait_time);
    }
    
    // Execute task
    execute_timeslice(current_task, 1ms);
    update_vruntime(current_task);
    
    if (current_task->remaining_time > 0) {
        runqueue.push(current_task);
    }
}
```

---

## 🚀 Getting Started

### Prerequisites

```bash
# System Requirements
- C++17 compatible compiler (GCC 7+, Clang 5+)
- CMake 3.10 or higher
- Python 3.8+ with matplotlib
- Git for version control
```

### 📦 Installation

```bash
# Clone the repository
git clone <repository-url>
cd cfs-scheduler-simulation

# Create Python virtual environment
python3 -m venv myvenv
source myvenv/bin/activate  # On Windows: myvenv\Scripts\activate

# Install Python dependencies
pip install matplotlib numpy pandas

# Build the project
mkdir -p build && cd build
cmake ..
make -j$(nproc)

# Run simulation
./cfs-scheduler

# Generate visualizations
cd ..
python3 plot.py
```

### 🎮 Quick Start

```bash
# Run complete simulation pipeline
./run_simulation.sh
```

---

## 📊 Simulation Results

### I/O-Bound Process Scheduling
![I/O Bound Scheduling](https://github.com/user-attachments/assets/193d8a9d-fd70-4a32-8a82-d55c0cc1a287)

**Key Observations:**
- I/O tasks experience **wait penalties** in vruntime
- Higher priority tasks get **more frequent scheduling**
- Clear visualization of **I/O wait periods** and CPU bursts

### CPU-Bound Process Scheduling  
![CPU Bound Scheduling](https://github.com/user-attachments/assets/d5f5367a-0a17-4ac6-9663-1b06e96e9677)

**Key Observations:**
- **Process 1** (highest priority) completes fastest
- **Proportional CPU sharing** based on task weights
- **Smooth vruntime progression** without I/O interruptions

---

## 🔬 Technical Deep Dive

### CFS Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| `NICE_0_LOAD` | 1024 | Base weight for priority 0 tasks |
| `CPU_TIMESLICE` | 1ms | Execution quantum for CPU tasks |
| `IO_WAIT_TIME` | 10ms | Simulated I/O operation duration |
| `MIN_GRANULARITY` | 1ms | Minimum scheduling granularity |

### Fairness Metrics

The simulation tracks several fairness indicators:

- **Proportional CPU Share**: Actual vs. expected CPU allocation
- **Latency Distribution**: Task response time analysis  
- **vruntime Spread**: Variance in virtual runtime across tasks
- **Throughput**: Tasks completed per time unit

### Performance Characteristics

```
Time Complexity:
- Task Selection: O(1) - min-heap top access
- Task Insertion: O(log n) - heap insertion
- vruntime Update: O(1) - arithmetic operation

Space Complexity: O(n) where n = number of tasks
```

---

## 📈 Visualization Features

### Real-time Metrics
- **Task Timeline**: Gantt chart of process execution
- **vruntime Evolution**: Dynamic virtual runtime tracking
- **CPU Utilization**: System-wide CPU usage patterns
- **Wait Time Analysis**: I/O wait and scheduling delays

### Interactive Analysis
```python
# Generate custom visualizations
python3 analyze.py --metric vruntime --tasks 1,2,3
python3 analyze.py --metric utilization --timerange 0-100
```

---

## 🎓 Educational Value

This simulation serves as an excellent learning tool for:

- **Operating Systems Courses**: Understanding modern scheduling algorithms
- **Systems Programming**: Real-world kernel concepts implementation  
- **Performance Analysis**: Measuring and optimizing system behavior
- **Algorithm Design**: Fair resource allocation strategies

---

## 🔍 Advanced Features

### Custom Task Generation
```cpp
// Create custom workload scenarios
TaskGenerator generator;
generator.add_cpu_intensive_tasks(5, priority_range={0,2});
generator.add_interactive_tasks(3, io_ratio=0.8);
generator.run_simulation();
```

### Configurable Parameters
```bash
# Run with custom parameters
./cfs-scheduler --timeslice 2 --io-wait 15 --nice-load 2048
```

---

## 📚 Further Reading

- 🔗 **Detailed Blog Post**: [Dissecting Linux Schedulers & Implementing CFS Simulation](https://singhdevhub.bearblog.dev/dissecting-linux-schedulers-implementing-our-toy-cfs_scheduler-simulation/)
- 📖 **Linux Kernel Documentation**: [CFS Scheduler Design](https://www.kernel.org/doc/Documentation/scheduler/sched-design-CFS.txt)
- 🎥 **Video Tutorial**: Coming soon...

---

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

### Development Setup
```bash
# Install development dependencies  
pip install -r requirements-dev.txt

# Run tests
make test

# Code formatting
make format
```

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- **Linux Kernel Community** for the original CFS implementation
- **Academic Research** on fair scheduling algorithms
- **Open Source Contributors** who made this project possible

---

<div align="center">

**⭐ Star this repository if you found it helpful!**

*Built with ❤️ for the systems programming community*

</div>
