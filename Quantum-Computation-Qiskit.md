# Quantum Computation with Qiskit

A comprehensive guide to Qiskit — the world's most popular and performant open-source software stack for quantum computing and algorithms research.

---

## Overview

| Property | Details |
|----------|---------|
| **Name** | Qiskit |
| **Type** | Open-source quantum computing SDK |
| **Language** | Python (with C/Rust acceleration) |
| **License** | Apache 2.0 |
| **Repository** | [github.com/Qiskit/qiskit](https://github.com/Qiskit/qiskit) |
| **Documentation** | [docs.quantum.ibm.com](https://quantum.cloud.ibm.com/docs) |
| **Current Version** | v2.5+ |
| **Downloads** | 13M+ |
| **Community** | 7K+ dependent projects, 1,700+ advocates across 74+ countries |

### Key Statistics
- **69%** of developers prefer Qiskit as primary SDK (Unitary Foundation Survey 2025)
- **83x** faster transpilation than next leading SDK (Benchpress toolkit)
- **29% fewer** two-qubit gates in compiled circuits

---

## Installation

### Basic Installation
```bash
pip install qiskit
```

### With Visualization Support
```bash
pip install qiskit[visualization]
```

### With Qiskit Runtime (IBM Quantum)
```bash
pip install qiskit-ibm-runtime
```

### Development Installation
```bash
git clone https://github.com/Qiskit/qiskit
cd qiskit
pip install -e .[dev]
```

---

## Core Architecture

Qiskit is organized into four main functional layers:

```
┌─────────────────────────────────────────────────────────┐
│                    QISKIT APPLICATIONS                   │
│  (Addons, Function Templates, Algorithm Libraries)      │
├─────────────────────────────────────────────────────────┤
│                  QISKIT PRIMITIVES                       │
│  (Sampler, Estimator, Circuit Runners)                  │
├─────────────────────────────────────────────────────────┤
│                 TRANSPILATION & COMPILATION             │
│  (Pass Manager, Synthesis, Optimization, Layout)        │
├─────────────────────────────────────────────────────────┤
│               CIRCUIT & QUANTUM INFORMATION             │
│  (QuantumCircuit, Gates, Operators, Statevectors)       │
└─────────────────────────────────────────────────────────┘
```

---

## Core Modules

### 1. Circuit Construction (`qiskit.circuit`)

The foundation for building quantum programs.

#### Key Classes
| Class | Purpose |
|-------|---------|
| `QuantumCircuit` | Main container for quantum circuits |
| `QuantumRegister` / `ClassicalRegister` | Register definitions |
| `Gate` / `Instruction` | Quantum operations |
| `Parameter` / `ParameterVector` | Parameterized circuits |
| `ControlFlowOp` | Classical control flow (if/for/while) |

#### Circuit Library (`qiskit.circuit.library`)
Pre-built circuits and gates:
- **Standard Gates**: H, X, Y, Z, S, T, CX, CZ, CCX, RX, RY, RZ, U3, etc.
- **N-Local Ansätze**: `EfficientSU2`, `TwoLocal`, `RealAmplitudes`, `NLocal`
- **QAOA**: `QAOAAnsatz`
- **QFT**: `QFT`, `QFTGate`
- **Arithmetic**: Adders, multipliers, comparators
- **State Preparation**: `Initialize`, `StatePreparation`
- **Feature Maps**: `ZZFeatureMap`, `PauliFeatureMap`, `ZFeatureMap`
- **VQE Ansätze**: `UCCSD`, `ExcitationPreserving`, `HamiltonianVariationalAnsatz`

#### Control Flow (v2.0+)
```python
from qiskit.circuit import QuantumCircuit, IfElseOp, ForLoopOp, WhileLoopOp

qc = QuantumCircuit(2, 2)
qc.h(0)
qc.measure(0, 0)

with qc.if_test((qc.clbits[0], 1)):
    qc.x(1)

with qc.for_loop(range(3)):
    qc.ry(0.5, 1)
```

### 2. Quantum Information (`qiskit.quantum_info`)

Tools for quantum state analysis, operators, and quantum information theory.

#### State Representations
| Class | Description |
|-------|-------------|
| `Statevector` | Pure quantum states |
| `DensityMatrix` | Mixed states |
| `StabilizerState` | Stabilizer formalism |

#### Operators
| Class | Description |
|-------|-------------|
| `Operator` | General linear operators |
| `Pauli` / `PauliList` | Pauli operators |
| `SparsePauliOp` | Sparse Pauli sums |
| `Clifford` | Clifford group operations |
| `Choi` / `SuperOp` / `Kraus` | Quantum channels |

#### Quantum Information Tools
- Entanglement measures
- State fidelity and distance metrics
- Pauli basis conversions
- Process tomography support

### 3. Transpilation (`qiskit.transpiler`)

Compiles circuits for specific hardware constraints.

#### Core Components
| Component | Purpose |
|-----------|---------|
| `PassManager` | Orchestrates transpilation passes |
| `CouplingMap` | Hardware connectivity graph |
| `Target` | Full backend specification |
| `Layout` | Virtual-to-physical qubit mapping |
| `InstructionDurations` | Gate timing for scheduling |

#### Transpilation Stages
1. **Layout Selection** - Map virtual to physical qubits
2. **Routing** - Insert SWAPs for connectivity
3. **Translation** - Convert to basis gates
4. **Optimization** - Gate cancellation, commutation
5. **Scheduling** - Add delays, dynamical decoupling

#### Preset Pass Managers
```python
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager

pm = generate_preset_pass_manager(backend=backend, optimization_level=3)
transpiled_circuit = pm.run(circuit)
```

#### Optimization Levels
| Level | Description |
|-------|-------------|
| 0 | No optimization |
| 1 | Light optimization |
| 2 | Heavy optimization |
| 3 | Maximum optimization (default for hardware) |

#### Advanced Features
- **Approximate Quantum Compiler (AQC)**: `qiskit.synthesis.unitary.aqc`
- **High-Level Synthesis (HLS)**: Plugin-based synthesis
- **Solovay-Kitaev**: Approximate arbitrary unitaries
- **Clifford+T Synthesis**: For fault-tolerant circuits

### 4. Primitives (`qiskit.primitives`)

Unified interface for executing quantum programs (v1.0+).

#### Sampler
```python
from qiskit.primitives import Sampler

sampler = Sampler()
job = sampler.run([circuit])
result = job.result()
counts = result[0].data.meas.get_counts()
```

#### Estimator
```python
from qiskit.primitives import Estimator
from qiskit.quantum_info import SparsePauliOp

estimator = Estimator()
observable = SparsePauliOp.from_list([("ZZ", 1.0), ("XX", 0.5)])
job = estimator.run([circuit], [observable])
result = job.result()
expectation_values = result[0].data.evs
```

#### Primitive Types
| Version | Sampler | Estimator |
|---------|---------|-----------|
| V1 | `BaseSamplerV1` | `BaseEstimatorV1` |
| V2 (Current) | `BaseSamplerV2` | `BaseEstimatorV2` |
| Statevector | `StatevectorSampler` | `StatevectorEstimator` |
| Backend | `BackendSamplerV2` | `BackendEstimatorV2` |

---

## Execution Workflow

### Basic Workflow
```python
from qiskit import QuantumCircuit
from qiskit.primitives import Sampler
from qiskit.visualization import plot_histogram

# 1. Build circuit
qc = QuantumCircuit(2, 2)
qc.h(0)
qc.cx(0, 1)
qc.measure([0, 1], [0, 1])

# 2. Execute
sampler = Sampler()
job = sampler.run([qc], shots=1024)
result = job.result()

# 3. Analyze
counts = result[0].data.meas.get_counts()
plot_histogram(counts)
```

### Hardware Execution (IBM Quantum)
```python
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler

service = QiskitRuntimeService(channel="ibm_quantum", token="YOUR_TOKEN")
backend = service.least_busy(operational=True, simulator=False)

sampler = Sampler(mode=backend)
job = sampler.run([transpiled_circuit], shots=1024)
result = job.result()
```

### Serverless Execution
```python
from qiskit_serverless import QiskitServerless

serverless = QiskitServerless()
job = serverless.run("my-program", arguments={"circuit": circuit})
result = job.result()
```

---

## Key Application Domains

### 1. Optimization
- **QAOA** (Quantum Approximate Optimization Algorithm)
- **VQE** (Variational Quantum Eigensolver)
- **Quantum Annealing** formulations

### 2. Quantum Simulation
- **Chemistry**: Molecular ground states, reaction dynamics
- **Physics**: Condensed matter, lattice gauge theories
- **Quantum Diagonalization**: Sample-based methods

### 3. Quantum Machine Learning
- **Quantum Kernels**: `QuantumKernel`, `QuantumSVC`
- **Variational Classifiers**: `VQC`
- **Neural Networks**: `QuantumNeuralNetwork`

### 4. PDEs & Scientific Computing
- **QUICK-PDE**: Quantum algorithms for PDEs
- **Fluid Dynamics**: Variational quantum solvers

### 5. Quantum Error Mitigation
- **Zero-Noise Extrapolation (ZNE)**
- **Probabilistic Error Cancellation (PEC)**
- **Measurement Error Mitigation (M3)**
- **Dynamical Decoupling**

---

## Qiskit Addons (v2.0+)

Modular algorithmic building blocks for utility-scale experiments.

| Addon | Category | Purpose |
|-------|----------|---------|
| **MPF** | Map | Measurement-padded factorization |
| **AQC-Tensor** | Map | Tensor-network AQC |
| **Transpiler** | Map/Optimize | Advanced transpilation |
| **OBP** | Optimize | Observable-based pruning |
| **Circuit Cutting** | Optimize | Divide large circuits |
| **Primitives** | Execute | Unified execution interface |
| **SQD** | Post-process | Subspace quantum diagonalization |
| **M3** | Post-process | Measurement mitigation |
| **Quantum Info** | Post-process | Quantum information tools |

### Using Addons
```python
# Install specific addons
pip install qiskit-addon-mpf qiskit-addon-sqd

# Use in workflow
from qiskit_addon_mpf import mpf_transpile
from qiskit_addon_sqd import sqd_postprocess

transpiled = mpf_transpile(circuit, backend)
result = sampler.run(transpiled).result()
mitigated = sqd_postprocess(result, observables)
```

---

## Qiskit Function Templates

Pre-built code templates for common research workflows:
- **QAOA for MaxCut**: Complete optimization pipeline
- **VQE for Chemistry**: Ground state energy calculation
- **Quantum Kernels**: Classification workflows
- **Error Mitigation**: ZNE, PEC, M3 implementations
- **Circuit Cutting**: Large circuit decomposition

[Browse Templates](https://quantum.cloud.ibm.com/docs/guides/qiskit-function-templates)

---

## Visualization (`qiskit.visualization`)

```python
from qiskit.visualization import (
    circuit_drawer,
    plot_histogram,
    plot_bloch_multivector,
    plot_state_city,
    plot_state_qsphere,
    plot_circuit_layout,
    plot_coupling_map,
    plot_error_map,
    timeline_drawer
)

# Circuit diagrams
circuit_drawer(qc, output='mpl', style='iqp')

# State visualization
plot_state_qsphere(statevector)
plot_bloch_multivector(statevector)

# Results
plot_histogram(counts)

# Hardware
plot_coupling_map(backend.coupling_map)
plot_error_map(backend)
```

---

## Serialization & Interoperability

### OpenQASM 2.0
```python
from qiskit import qasm2
qasm_str = qasm2.dumps(circuit)
circuit = qasm2.loads(qasm_str)
```

### OpenQASM 3.0
```python
from qiskit import qasm3
qasm3_str = qasm3.dumps(circuit)
circuit = qasm3.loads(qasm3_str)
```

### QPY (Qiskit Python Serialization)
```python
from qiskit import qpy
import io

# Save
buffer = io.BytesIO()
qpy.dump(circuit, buffer)

# Load
buffer.seek(0)
circuit = qpy.load(buffer)[0]
```

---

## Hardware Support

Qiskit is backend-agnostic with plugins for major providers:

| Provider | Plugin | Access |
|----------|--------|--------|
| **IBM Quantum** | Built-in / `qiskit-ibm-runtime` | Cloud |
| **IonQ** | `qiskit-ionq` | Cloud |
| **Amazon Braket** | `qiskit-braket` | AWS |
| **Azure Quantum** | `qiskit-azure` | Azure |
| **IQM** | `iqm-qiskit` | Cloud/On-prem |
| **Rigetti** | `qiskit-rigetti` | Cloud |
| **Quantinuum** | `qiskit-quantinuum` | Cloud |
| **Superstaq** | `qiskit-superstaq` | Cloud |
| **AQT** | `qiskit-aqt` | Cloud |
| **Alice & Bob** | `qiskit-alice-bob` | Cloud |

---

## Heterogeneous Orchestration

### Quantum Resource Management Interface (QRMI)
Standardized control of quantum resources across HPC environments.

### Slurm Integration
```bash
# Submit quantum job via Slurm
sbatch --qos=quantum --gres=quantum:1 job_script.sh
```

### Qiskit Serverless
```python
from qiskit_serverless import QiskitServerless, ServerlessClient

serverless = QiskitServerless(token="...", url="...")
function = serverless.upload("my_quantum_function", "function.py")
job = function.run(arguments={"shots": 1024})
```

---

## Learning Resources

### Official
- **Documentation**: [quantum.cloud.ibm.com/docs](https://quantum.cloud.ibm.com/docs)
- **Learning Path**: [quantum.cloud.ibm.com/learning](https://quantum.cloud.ibm.com/learning)
- **Tutorials**: [quantum.cloud.ibm.com/docs/tutorials](https://quantum.cloud.ibm.com/docs/tutorials)
- **API Reference**: [docs.quantum.ibm.com/api/qiskit](https://docs.quantum.ibm.com/api/qiskit)

### Community
- **Qiskit Slack**: [Join here](https://airtable.com/appkU8W0NtENifBxX/shrv2nz4kGxpCAsOp)
- **YouTube**: [youtube.com/@qiskit](https://www.youtube.com/@qiskit)
- **Events**: [quantum.ibm.com/events](https://quantum.ibm.com/events)
- **Advocate Program**: 1,700+ members globally

### Courses
- **Quantum Computing Foundations**
- **Qiskit for Quantum Machine Learning**
- **Quantum Algorithms with Qiskit**
- **Error Mitigation Techniques**

---

## Version History (Major Releases)

| Version | Date | Key Features |
|---------|------|--------------|
| **v2.5** | 2025 | Enhanced addons, improved transpiler, Rust acceleration |
| **v2.0** | 2024 | Primitives V2, control flow, modular architecture |
| **v1.0** | 2023 | Stable API, primitives V1, improved transpiler |
| **v0.46** | 2023 | Last v0 release, migration guides |
| **v0.25** | 2021 | Pulse-level control, backend v2 |

---

## Contributing

```bash
# Clone and set up
git clone https://github.com/Qiskit/qiskit
cd qiskit

# Run tests
pytest test/python

# Code style
pre-commit run --all-files
```

### Repositories
| Repository | Purpose |
|------------|---------|
| `Qiskit/qiskit` | Core SDK |
| `Qiskit/qiskit-ibm-runtime` | IBM Quantum Runtime |
| `Qiskit/qiskit-serverless` | Serverless execution |
| `Qiskit/qiskit-addon-*` | Algorithm addons |
| `Qiskit/benchpress` | Performance benchmarking |
| `qiskit-community/*` | Community plugins |

---

## References

- **Original Paper**: [Qiskit: An Open-Source Framework for Quantum Computing](https://arxiv.org/abs/1711.10289) (2017)
- **Benchpress**: [github.com/Qiskit/benchpress](https://github.com/Qiskit/benchpress)
- **Roadmap**: [Quantum Computing Roadmap](https://quantum.ibm.com/roadmap)
- **Unitary Foundation Survey**: [unitaryfoundation.github.io/survey-2025](https://unitaryfoundation.github.io/survey-2025/)

---

## License

Apache License 2.0 — Free for commercial and academic use.

---

*Document generated: 2026 | Qiskit v2.5+*