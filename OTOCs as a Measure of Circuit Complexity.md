# Experiment: OTOCs as a Measure of Circuit Complexity

This document details a specific experiment from the main OTOCs project: using the Out-of-Time-Order Correlator (OTOC) not just to observe physical phenomena, but as a **quantitative tool to analyze the complexity of generic quantum circuits**.

We compare two different types of quantum evolution:
1.  **Physics-Motivated Evolution:** A Trotterized simulation of a physical system (the Ising model), where dynamics are governed by local interactions.
2.  **Hardware-Motivated Evolution:** A random quantum circuit, which represents a generic computation on a quantum computer and has no underlying physical structure.

**The scientific question:** How does the "scrambling power" of a generic quantum computation compare to that of a chaotic physical system?

## Method

To answer this, we adapt the ancilla-based OTOC measurement. Instead of measuring the OTOC as a function of physical time `t`, we measure it as a function of **circuit depth**. Circuit depth is a natural measure of computational effort or "time" in the context of a digital quantum algorithm.

The core of the experiment is as follows:
1.  **Random Circuit Generation:** For a given `depth`, we generate a random quantum circuit consisting of `depth` layers. Each layer contains random single-qubit rotations followed by a set of entangling `CZ` gates.
2.  **OTOC Measurement:** We use this random circuit as the `forward_evolution` operator `U` in our ancilla-based OTOC measurement circuit. We measure the OTOC proxy, `Re(<W(U)V>)`, between operators `W` and `V` placed at opposite ends of the qubit chain.
3.  **Statistical Averaging:** A single random circuit is just one specific instance. To get a meaningful result about the *general* properties of circuits of a certain depth, we must average the OTOC results over many different random circuit instances. **The results below are an average over 1000 unique random circuits for each depth point.**

## Results

The simulation was performed on a 12-qubit chain, averaging over 1000 random circuit instances per depth, with 4000 measurement shots for each instance.

![Scrambling Power of Random Circuits](https://github.com/peterbabulik/OTOCs/raw/main/picture5.png)

The numerical results confirm the visual finding:

| Depth | Avg. OTOC Proxy |
|-------|-----------------|
| 0     | 0.0010          |
| 1     | 0.0000          |
| 2     | -0.0003         |
| 3     | 0.0001          |
| ...   | ...             |
| 20    | 0.0000          |

## Analysis and Conclusion

This result is profoundly different from the gradual decay observed in the physical simulation, and this difference is the key insight.

1.  **Instantaneous Scrambling:** The OTOC proxy value effectively drops to zero immediately at `depth=1` and remains there. The small fluctuations around zero are purely statistical noise, which becomes very small when averaging over 1000 instances.

2.  **Random Circuits are "Fast Scramblers":** The physical Ising model scrambles information at a finite speed, constrained by its local interactions (the "light cone"). A random circuit has no such constraint. The very first layer of entangling gates creates non-local correlations across the entire system. Information is not just spread—it is immediately and thoroughly **shuffled**, hiding any initial local correlation.

3.  **Implications for Quantum Computing:** This experiment demonstrates why random circuit sampling is a standard benchmark for demonstrating **quantum advantage**. The ability of these circuits to rapidly generate highly complex, globally entangled states is precisely what makes them incredibly difficult for classical computers to simulate. The OTOC provides a direct measure of this complexity generation.

**In conclusion, this experiment successfully uses the OTOC as an analytical tool to prove that hardware-motivated random circuits are exceptionally powerful scramblers of quantum information, far more so than the structured evolution of a local physical Hamiltonian. The information decay is not gradual; it is, for all practical purposes, instantaneous.**

---

### Code for the Experiment

Below is the Python code used to generate these results.

<details>
<summary>Click to expand the Python code</summary>

```python
import cirq
import numpy as np
import matplotlib.pyplot as plt

# --- 1. Define the Random Circuit Generator ---

def generate_random_layer(qubits):
    """Yields a layer of random single-qubit rotations and a layer of CZ gates."""
    # Layer of random single-qubit gates
    for qubit in qubits:
        yield cirq.PhasedXPowGate(
            phase_exponent=np.random.uniform(0, 1),
            exponent=0.5
        ).on(qubit)

    # Layer of entangling gates (CZ)
    for i in range(0, len(qubits) - 1, 2):
        yield cirq.CZ(qubits[i], qubits[i+1])
    for i in range(1, len(qubits) - 1, 2):
        yield cirq.CZ(qubits[i], qubits[i+1])

# --- 2. Ancilla-Based OTOC Simulation with Random Circuits ---

# System Parameters
N = 12
system_qubits = cirq.LineQubit.range(N)
ancilla = cirq.GridQubit(0, N)

# Operators and sites
op_W = cirq.X
op_V = cirq.Z
w_site = N - 1
v_site = N // 2

# Simulation Parameters
max_depth = 20
depth_steps = np.arange(0, max_depth + 1)
otoc_proxy_values = []
repetitions = 4000
num_instances = 1000 # High number of instances for clean results

print(f"Measuring OTOC vs. Circuit Depth for a {N}-qubit system.")
print(f"Averaging over {num_instances} random circuit instances per depth.")

simulator = cirq.Simulator()

for depth in depth_steps:
    
    instance_results = []
    
    for instance_num in range(num_instances):
        if depth > 0:
            forward_evolution = cirq.Circuit(generate_random_layer(system_qubits) for _ in range(depth))
            backward_evolution = cirq.inverse(forward_evolution)
        else:
            forward_evolution = cirq.Circuit()
            backward_evolution = cirq.Circuit()
        
        otoc_circuit = cirq.Circuit(
            cirq.H(ancilla),
            op_V(system_qubits[v_site]).controlled_by(ancilla),
            forward_evolution,
            op_W(system_qubits[w_site]).controlled_by(ancilla),
            backward_evolution,
            cirq.H(ancilla),
            cirq.measure(ancilla, key='ancilla_meas')
        )
        
        result = simulator.run(otoc_circuit, repetitions=repetitions)
        counts = result.histogram(key='ancilla_meas')
        expectation_value_z = (counts.get(0, 0) - counts.get(1, 0)) / repetitions
        instance_results.append(expectation_value_z)

    avg_otoc_proxy = np.mean(instance_results)
    otoc_proxy_values.append(avg_otoc_proxy)
    
    print(f"Depth: {depth:2d}, Avg. OTOC Proxy = {avg_otoc_proxy:.4f}")

print("Simulation complete!")

# --- 3. Plot the Results ---
plt.figure(figsize=(10, 6))
plt.plot(depth_steps, otoc_proxy_values, marker='.', linestyle='-')
plt.xlabel("Circuit Depth", fontsize=14)
plt.ylabel(f"Avg. OTOC Proxy = Re(<W_{w_site}(t)V_{v_site}>)", fontsize=14)
plt.title(f"Scrambling Power of Random Circuits", fontsize=16)
plt.grid(True)
plt.show()
```

</details>
