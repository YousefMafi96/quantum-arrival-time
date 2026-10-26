# Arrival-time distributions in monitored quantum circuits

Simulation code for the paper

> Y. Mafi, K. G. S. H. Gunawardana, K. Pöyhönen, and T. Ojanen,
> *Arrival-time problem in the quantum information era – observation of the quantum Zeno crossover in monitored arrival-time dynamics.*

A particle (an up-spin) starts at one end of a spin chain or lattice and propagates under a brickwork circuit of XXZ gates. A detector qubit at the other end is measured every monitoring period Δt with mid-circuit measurements, and the arrival-time distribution is built from the first detection. The notebooks reproduce the simulations behind the results of the paper and can be run for any parameters you choose.

## Contents

| File | Description |
|---|---|
| `arrival_time_1D.ipynb` | Open chain of `L` qubits, `K` up-spins on the first sites, detector on the last qubit |
| `arrival_time_2D.ipynb` | `LX × LY` square lattice, up-spin in one corner, detector in the opposite corner |
| `requirements.txt` | Python packages |

Both notebooks have the same structure:

1. **Parameters** – the only cell you need to edit
2. **Circuit** – brickwork XXZ circuit with mid-circuit measurements of the detector
3. **Post-processing** – U(1) post-selection and first-detection probability
4. **Simulation** – ideal and/or noisy simulation for every monitoring period
5. **Plots**
   - cumulative arrival probability P_Σ(t_n), ideal vs. noisy, with statistical error band
   - local magnetization map (1D: qubit vs. time; 2D: lattice snapshots)
   - error distribution: fraction of shots in each particle-number sector before post-selection

## Installation

```bash
git clone <this repository>
cd <this repository>
pip install -r requirements.txt
jupyter notebook
```

No IBM Quantum account is needed. Simulations run locally with

- `"aer"` – the ideal, noiseless `AerSimulator`, and
- `"fake"` – `AerSimulator.from_backend(...)` with the noise model, native gates and coupling map of an IBM fake backend from `qiskit_ibm_runtime.fake_provider` (default: `FakeKingston` for 1D, `FakeBerlin` for 2D).

## Usage

Open a notebook, edit the parameter cell and run all cells. The main parameters are:

| Parameter | Meaning |
|---|---|
| `L` (1D) / `LX, LY` (2D) | system size |
| `K` | number of up-spins in the initial state |
| `DELTA` | anisotropy Δ of the XXZ gate |
| `TAU` | gate parameter τ in exp[−iτ(XX + YY + ΔZZ)] |
| `LAYER_TIME` | time per brickwork layer (1D: 1/2, 2D: 1/4, so one full cycle of layers takes time 1) |
| `DT_LIST` | monitoring periods Δt to simulate (multiples of `LAYER_TIME`) |
| `T` | final time; one circuit is run for every t_n = nΔt ≤ T |
| `MODES` | `["aer"]`, `["fake"]` or both |
| `FAKE_BACKEND` | name of the fake backend used for `"fake"` |
| `SHOTS` | shots per circuit |
| `SEED` | `None` for independent random seeds, or an integer for reproducible runs |
| `MAP_MODE`, `MAP_DT` | which run is shown in the magnetization map and the error distribution |

## Method in brief

- **Circuits.** For every monitoring time t_n a separate circuit is executed, as on hardware. The detector is measured after every monitoring period; at t_n all qubits are measured.
- **First detection.** P_a(t_n) is the fraction of shots in which all earlier detector records are 0 (down) and the detector is 1 (up) at t_n. The cumulative arrival probability is P_Σ(t_n) = Σ_{i≤n} P_a(t_i).
- **Post-selection.** The ideal dynamics conserves the number of up-spins, so shots whose final particle number differs from `K` must come from noise and are discarded. The kept fraction N_i/N_shot is printed below the error-distribution plot.
- **Statistical error.** σ_a²(t_i) = P_a(1 − P_a)/N_i for each monitoring time and σ_Σ²(t_n) = Σ_{i≤n} σ_a²(t_i), shown as a band around the noisy curves.
- **Conventions.** Up-spin = |1⟩. Qiskit bitstrings are little-endian: classical bits 0…N−1 hold the final measurement of the qubits, and the following bits hold the detector records in chronological order.

## Runtime

Circuits with mid-circuit measurements are simulated shot by shot, so the runtime grows with `SHOTS`, the number of monitoring times `T/Δt` and the number of qubits. The noisy fake-backend simulation is slower than the ideal one. With the default parameters each notebook takes about 5–10 minutes on a laptop, most of it for the noisy simulation; use `MODES = ["aer"]` for a quick ideal run. For the paper-size runs (e.g. 20,000 shots, or the 4 × 4 lattice), expect considerably longer runtimes.

## Citation

If you use this code, please cite the paper above.

