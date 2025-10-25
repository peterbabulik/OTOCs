# Quantum Dynamics Simulation: An Exploration of OTOCs

This repository documents a hands-on journey into the physics of **quantum chaos** and **information scrambling** by simulating **Out-of-Time-Order Correlators (OTOCs)**. Using Google's **Cirq** framework, this project progresses from simple, exact simulations to advanced, hardware-ready techniques, exploring the core challenges and phenomena of many-body quantum dynamics.

The central piece of this work is the `OTOCs.ipynb` notebook, which contains all the code, experiments, and analysis.

## Gallery of Key Experiments & Results

This project is a series of computational experiments, each designed to reveal a different aspect of quantum physics and simulation.

---

### 1. The Ideal Case: Exact Simulation (6 Qubits)
The first experiment calculates the OTOC `C(t) = <|[X₀(t), Z₃]²|>` for a small, 6-qubit system using exact matrix exponentiation. This provides a perfect, noise-free baseline.

![Information Scrambling in a 6-Qubit Ising Chain](https://github.com/peterbabulik/OTOCs/raw/main/picture1.png)
*   **Observation:** The plot shows a clear "S-curve": an initial delay (the light cone), a period of rapid growth (scrambling), and saturation (thermal equilibrium). This is the classic signature of an OTOC in a chaotic system.

---

### 2. Scaling Up: State-Vector Simulation with Trotterization (12 Qubits)
To simulate a larger system, we switched to a Trotter-based method, which approximates the time evolution. With a sufficiently small time step (`dt=0.01`), we reproduced the same physical behavior for a 12-qubit chain.

![Information Scrambling in a 12-Qubit Chain (Trotterized, dt=0.01)](https://github.com/peterbabulik/OTOCs/raw/main/picture6.png)
*   **Observation:** The characteristic S-curve is preserved, demonstrating that our scalable algorithm is physically accurate. The light cone is now longer due to the larger system size.

---

### 3. Preparing for Real Hardware: The Ancilla-Based Measurement
On a real quantum computer, we can't measure the squared commutator directly. Instead, we use an ancilla qubit in an interferometric circuit to measure a proxy value: `Re(<W(t)V>)`.

![Optimized OTOC Sweep for a 12-Qubit Chain](https://github.com/peterbabulik/OTOCs/raw/main/picture7.png)
*   **Observation:** This plot shows the expected physical signature for this measurement: an initial build-up of correlations followed by a **decay to an equilibrium value** as information scrambles and the initial correlation is lost. This experiment was also optimized using Cirq's `run_sweep` for vastly improved performance.

---

### 4. The Control Experiment: A System Without Chaos
What happens if the system isn't chaotic? We ran the hardware-ready simulation on a 1D chain but set the chaos-inducing parameter `h=0`.

![Hardware-Ready OTOC Measurement for a 12-Qubit Chain](https://github.com/peterbabulik/OTOCs/raw/main/picture4.png)
*   **Observation:** The signal remains at zero (within the statistical noise floor). **Information does not scramble.** This crucial control experiment proves that the decay and scrambling we observed is a direct consequence of quantum chaos.

---

### 5. The Reality of Quantum Computing: Introducing Noise
Real quantum computers are noisy. We simulated our experiment on an 8-qubit chain with a tiny `0.1%` error probability after each gate.

![Noisy OTOC Measurement (N=8, p=0.001)](https://github.com/peterbabulik/OTOCs/raw/main/picture2.png)
*   **Observation:** The signal **decays rapidly to zero**. The delicate quantum correlations are destroyed by decoherence. This powerfully illustrates the primary challenge facing today's quantum hardware. The simulation also demonstrated the immense classical resources required to simulate noise, as it was orders of magnitude slower.

---

### 6. Lessons from a "Failed" Experiment: The Importance of Accuracy
During development, a simulation of a 2D qubit grid with a Trotter step of `dt=0.1` produced unphysical oscillations.

![Hardware-Ready OTOC on a 4x3 Qubit Grid](https://github.com/peterbabulik/OTOCs/raw/main/picture3.png)
*   **Observation:** The result is dominated by **Trotter error**. This served as a critical lesson: as system complexity (e.g., connectivity) increases, the simulation's accuracy (e.g., a smaller `dt`) must also increase to obtain physically meaningful results.

## How to Use This Repository

The best way to explore this project is to run the Jupyter Notebook yourself.

### Prerequisites
You will need Python 3 and the following libraries:
- `cirq`
- `numpy`
- `scipy`
- `matplotlib`
- `sympy`

### Running the Notebook
1.  Clone the repository:
    ```bash
    git clone https://github.com/peterbabulik/OTOCs.git
    cd OTOCs
    ```
2.  Install the dependencies:
    ```bash
    pip install cirq numpy scipy matplotlib sympy
    ```
3.  Open and run the `OTOCs.ipynb` notebook in Google Colab, VS Code, or a local Jupyter environment. The notebook is fully commented and guides you through each experiment.

## License
This project is licensed under the MIT License. See the `LICENSE` file for details.
