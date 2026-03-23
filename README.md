# Grover's Algorithm — 10-Qubit Implementation

A comprehensive quantum computing project implementing **Grover's search algorithm** on a
10-qubit system using Qiskit.  
The project demonstrates quantum speedup over classical search through two independently
built oracle implementations — all without using any built-in Grover library routines.

---

## Author & Course Information

| Field | Detail |
|---|---|
| **Author** | Sharanya C |
| **Course** | Quantum Computing |
| **Project Type** | Final Course Project |

---

## Project Overview

The algorithm searches a $2^{10} = 1024$-state space for two marked bit-strings using quantum
amplitude amplification.  Two different oracle implementations are compared:

| | Part A | Part B |
|---|---|---|
| **Oracle** | IBM-style — `MCMTGate(ZGate())` | Phase Kickback — MCX into ancilla $|{-}\rangle$ |
| **Diffuser** | Manual $H^{\otimes n}X^{\otimes n}\mathrm{MCZ}X^{\otimes n}H^{\otimes n}$ | Same as Part A |
| **Qubits** | 10 | 11 (10 data + 1 ancilla) |
| **Built-in Grover** | ✗ Not used | ✗ Not used |

### Key Metrics

| Parameter | Value |
|---|---|
| Search space | $N = 2^{10} = 1024$ states |
| Marked states | $M = 2$ |
| Optimal iterations | $k_{\text{opt}} = 17$ |
| Theoretical success probability | $\approx 99.9\,\%$ |
| Classical speedup | $\approx 30\times$ |

---

## Problem Statement

**Search task:** Find the two marked bit-strings in an unsorted database of 1024 quantum states.

| Marked state | Decimal |
|---|---|
| `"0110011010"` | 410 |
| `"1101010001"` | 849 |

- **Classical approach:** $O(N)$ queries — $\approx 512$ comparisons on average.  
- **Grover's algorithm:** $O(\sqrt{N})$ oracle calls — only **17 iterations** needed.

---

## Algorithm Theory

### Grover's Algorithm at a Glance

Starting from a uniform superposition of all $N$ states:

$$
|s\rangle = \frac{1}{\sqrt{N}}\sum_{x=0}^{N-1}|x\rangle
$$

each **Grover iteration** applies two operations:

1. **Oracle $U_f$** — applies a $-1$ phase to every marked state.  
2. **Diffuser $U_s$** — reflects all amplitudes about their mean (inversion about average).

After $k$ iterations the probability of measuring a marked state is:

$$
P_{\mathrm{marked}}(k) = \sin^2\!\bigl((2k+1)\,\theta\bigr),
\qquad
\theta = \arcsin\!\sqrt{\frac{M}{N}} \approx 0.0442\;\text{rad}
$$

The optimal number of iterations that maximises this probability is:

$$
k_{\mathrm{opt}} = \left\lfloor \frac{\pi}{4\theta} \right\rfloor = 17
$$

---

## Implementation Details

### Part A — IBM-Style Oracle

Adapted from the IBM Quantum Learning tutorial.  
Uses `MCMTGate(ZGate(), n-1, 1)` (multi-controlled-Z) together with the **open-control trick**:

1. Apply X gates on qubits where the target bit-string has `'0'` (temporarily making them $|1\rangle$).  
2. Apply MCZ — flips the phase when **all** qubits are $|1\rangle$.  
3. Undo the X gates.

**Reference:** <https://quantum.cloud.ibm.com/docs/en/tutorials/grovers-algorithm>

### Part B — Phase Kickback Oracle

Built entirely from scratch using elementary gates.

1. Prepare an ancilla qubit in state $|{-}\rangle = \frac{|0\rangle - |1\rangle}{\sqrt{2}}$.  
2. For each marked state: apply MCX with all 10 data qubits as controls and the ancilla as target.  
   When the data register holds the marked state, the MCX flips $|{-}\rangle \to -|{-}\rangle$,  
   kicking the $-1$ phase back into the data register.  
3. Uncompute the ancilla back to $|0\rangle$.

### Diffuser (both parts)

The same manually-constructed diffuser is used for both Part A and Part B:

$$
U_s = H^{\otimes n}\,X^{\otimes n}\;\underbrace{H_{n-1}\;\mathrm{MCX}\;H_{n-1}}_{\mathrm{MCZ}}\;X^{\otimes n}\,H^{\otimes n}
$$

No `grover_operator()` or any other built-in helper is used.

---

## Results

Both implementations achieve approximately **99.9 % success probability** at $k = 17$,
confirming the theoretical prediction $P_{\mathrm{theory}} = \sin^2(35\theta) \approx 0.999$.

Out of 8192 shots:

| State | Hits |
|---|---|
| `"0110011010"` (marked) | ≈ 4050 |
| `"1101010001"` (marked) | ≈ 4050 |
| All other 1022 states | ≈ 90 total |

---

## Circuit Complexity

| Metric | Part A (10 qubits) | Part B (11 qubits) |
|---|---|---|
| Oracle depth | ≈ 15–20 | ≈ 20–30 |
| Diffuser depth | ≈ 25–30 | ≈ 25–30 |
| Per-iteration depth | ≈ 40–50 | ≈ 45–60 |
| Total circuit depth ($k = 17$) | ≈ 700–800 | ≈ 800–1000 |

---

## Prerequisites

- Python 3.8+
- `qiskit` and `qiskit-aer`
- `numpy`, `matplotlib`
- Jupyter Notebook or JupyterLab

---

## Installation

```bash
pip install qiskit qiskit-aer numpy matplotlib jupyter
```

Or inside a virtual environment:

```bash
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install qiskit qiskit-aer numpy matplotlib
```

---

## Usage

1. Review the algorithm theory and mathematical framework.  
2. Run **Part A** (IBM-style oracle) simulation across $k = 1, 3, 5, 10, 17$.  
3. Run **Part B** (Phase Kickback oracle) simulation across the same iteration counts.  
4. Compare results, view histograms, and read the full summary.

---

## Important Notes

### Transpilation Requirement

`MCMTGate(ZGate())` is a high-level composite gate that Qiskit Aer cannot execute directly.
The notebook transpiles every circuit before running:

```python
qc_transpiled = transpile(qc, backend, optimization_level=1)
```

This decomposes the gate into Aer's supported basis gates (`cx`, `u`, `x`, `h`, `measure`).

### Ancilla Uncomputation (Part B)

The ancilla qubit in Part B is **not measured**.  
It is uncomputed back to $|0\rangle$ at the end of each oracle call:

$$
|{-}\rangle \;\xrightarrow{H}\; |1\rangle \;\xrightarrow{X}\; |0\rangle
$$

Only the 10 data qubits appear in the classical register.

---

## References

- [IBM Quantum Learning — Grover's Algorithm Tutorial](https://quantum.cloud.ibm.com/learning/en/modules/computer-science/grovers)
- [Qiskit Documentation](https://qiskit.org/documentation/)
- [Grover, L. K. (1996) — "A Fast Quantum Mechanical Algorithm for Database Search"](https://arxiv.org/abs/quant-ph/9605043)
- Nielsen & Chuang — *Quantum Computation and Quantum Information*, Cambridge University Press
- Lecture slides and notes provided in class

---

*Last updated: March 2026*
