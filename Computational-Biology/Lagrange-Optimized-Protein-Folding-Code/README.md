# Lagrange-Optimized-Protein-Folding-Code

# Constrained Lagrange Optimization of Protein Folding

A proof‑of‑concept pipeline that reconstructs protein Cα backbones by **minimizing a physically‑motivated energy function under hard Ramachandran constraints**.  The project demonstrates how classical constrained optimization can beat random guessing on small proteins *without* deep learning or massive training data. The Lennard-Jones potential is used to come up with the function we must optimize using Lagrange Constraints.

This is the Lennard-Jones Potential.

$$
V(r) = 4\varepsilon \left[ \left(\frac{\sigma}{r}\right)^{12} - \left(\frac{\sigma}{r}\right)^{6} \right]
$$

Where:

* $r$ is the distance between particles.
* $\varepsilon$ sets the depth of the potential well.
* $\sigma$ is the finite distance at which the inter-particle potential is zero.


---

## Problem Statement

Predict the 3‑D Cα coordinates of the first 50 residues of **PDB 1FME, chain A** using only dihedral angles (φ, ψ) and simple physical priors.

* **Variables:** 100 torsion angles (φ/ψ per residue)
* **Objective:** Minimize total energy
  $E = E_{torsion} + E_{steric} + E_{contact}$
* **Constraints:** φ/ψ for each residue restricted to a ±40° window around ideal (–60°, –40°) values from the Ramachandran map
* **Optimizer:** Sequential Least Squares Quadratic Programming (SLSQP)

---

## 🛠️ Methodology

| Term                  | Equation                                                                             | Role                         |                                     |                           |
| --------------------- | ------------------------------------------------------------------------------------ | ---------------------------- | ----------------------------------- | ------------------------- |
| Torsion prior         | $E_{torsion}=0.001\sum_i (\phi_i^2+\psi_i^2)$                                        | Keeps angles smooth/physical |                                     |                           |
| Steric repulsion      | $E_{steric}=\sum_{i<j}\mathbf{1}[r_{ij}<3.8]\,\big(1/(r_{ij}^{12}+\varepsilon)\big)$ | Penalizes clashes            |                                     |                           |
| Long‑range compaction | $E\_{contact}=-\sum\_{abs(i-j) >5},0.01/(r\_{ij}^{6}+\varepsilon)$                   | Rewards tertiary contacts |

The optimizer iteratively updates φ/ψ while respecting inequality bounds derived from the Ramachandran plot.

---

## Results

```
Mean worst‑case RMSD (random backbone): 12.45 Å
Model   — Mean RMSD: 8.87 ± 1.65 Å
           Best RMSD: 6.95 Å  (10 trials)
           Success   : 80 %
```

| RMSD (Å)  | Interpretation                 |
| --------- | ------------------------------ |
| **< 2**   | Near‑atomic match              |
| 2 – 4     | Correct fold, minor offsets    |
| 4 – 6     | Loose fold                     |
| **6 – 9** | Weak approximation (this work) |
| > 9       | Random‑like                    |

Our best run (6.95 Å) **outperforms random baselines (\~14 Å) but falls short of functional accuracy**, underscoring the need for richer physics (hydrogen bonds, side‑chains).

---

## Limitations & Future Work

* Energy lacks hydrogen bonding, electrostatics, and explicit bond‑length/angle constraints.
* SLSQP is prone to local minima; global heuristics (basin hopping, CMA‑ES) could improve convergence.
* Only Cα atoms—full backbone/side‑chain reconstruction would require additional geometric constraints.

---

## License

This project is released under the MIT License; see `LICENSE` for details.

---

## Acknowledgements

* **BioPython** for PDB parsing
* **SciPy** for SLSQP implementation
* Classic Ramachandran analysis (G. N. Ramachandran, 1963
