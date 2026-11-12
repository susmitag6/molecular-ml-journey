# Medicinal Chemistry Learning Journal — Lesson 23 (Continuation and Completion)

**Date:** 12 November 2026  
**Topic:** Water Networks, Water Displacement, Hydrophobic Effect, and Grand Canonical Monte Carlo (GCMC)  
**Learning Focus:** Understanding the role of solvent in protein–ligand binding and how computational methods can investigate hydration in binding pockets.

---

## 1. What I Learned Today

Today, I continued studying the role of water in protein–ligand recognition. The main focus was understanding why water molecules inside binding pockets can influence ligand binding affinity and how their displacement can affect the thermodynamics of binding.

I also explored three computational approaches used to study hydration: Grid Inhomogeneous Solvation Theory (GIST), WaterMap, and Grand Canonical Monte Carlo (GCMC).

### Water Networks and Water-Mediated Interactions

Water molecules can form hydrogen-bond bridges between protein residues and ligand functional groups. These interactions can contribute to the stability of a protein–ligand complex.

However, the presence of a water-mediated hydrogen bond does not automatically imply stronger binding affinity. The overall contribution depends on the free-energy balance, including solvation, desolvation, and entropy.

### Water Displacement

When a ligand binds to a protein, it may displace water molecules from the binding pocket into bulk solvent.

I learned that the thermodynamic effect of water displacement depends on the environment of the water molecule.

- A water molecule that is strongly stabilized by protein interactions may be costly to displace.
- A water molecule in a hydrophobic cavity with unfavorable hydration thermodynamics may be a promising candidate for displacement.
- The ligand must form sufficiently favorable interactions, or gain other free-energy advantages, to compensate for the changes associated with displacement.

An important point is that water displacement cannot be evaluated only by counting hydrogen bonds.

### Hydrophobic Effect

The hydrophobic effect arises from the thermodynamics of nonpolar surfaces interacting with water.

When hydrophobic parts of a ligand become buried inside a protein pocket, the solvent-exposed nonpolar surface area decreases. This can favor ligand binding, although the magnitude and enthalpy–entropy contributions depend on the molecular environment.

---

## 2. Computational Methods for Studying Water

### GIST: Grid Inhomogeneous Solvation Theory

GIST uses explicit-solvent molecular dynamics trajectories to estimate spatially resolved hydration thermodynamics.

The simulation region is divided into a three-dimensional grid, and water occupancy, interaction energies, and local entropy contributions are analyzed.

This method is particularly interesting to me because it connects directly to my experience with molecular dynamics, statistical mechanics, and trajectory analysis.

### WaterMap

WaterMap is a computational approach that identifies hydration sites in protein binding pockets and estimates their thermodynamic properties.

It can help medicinal chemists investigate which water molecules may be favorable candidates for displacement during ligand optimization.

### Grand Canonical Monte Carlo (GCMC)

GCMC allows the number of particles in a simulation system to fluctuate through insertion and deletion moves.

Unlike conventional molecular dynamics in a fixed-particle-number ensemble, GCMC samples configurations at fixed temperature, volume, and chemical potential.

This can be useful for buried protein binding pockets, where water exchange with bulk solvent may be slow.

---

## 3. Statistical Mechanics of GCMC

The grand canonical ensemble is characterized by fixed:

$$
\mu,\quad V,\quad T
$$

while the particle number \(N\) fluctuates.

The statistical weight of a configuration is related to

$$
\exp[-\beta(U-\mu N)]
$$

where

$$
\beta=\frac{1}{k_BT}.
$$

The activity is defined as

$$
z=\frac{e^{\beta\mu}}{\Lambda^3}
$$

where \(\Lambda\) is the thermal de Broglie wavelength.

### Insertion Acceptance Probability

$$
P_{\mathrm{ins}}=
\min\left(1,\frac{zV}{N+1}e^{-\beta\Delta U_{\mathrm{ins}}}\right)
$$

### Deletion Acceptance Probability

$$
P_{\mathrm{del}}=
\min\left(1,\frac{N}{zV}e^{-\beta\Delta U_{\mathrm{del}}}\right)
$$

These expressions follow from the detailed-balance condition and the Metropolis–Hastings acceptance rule, assuming equal probabilities of proposing insertion and deletion.

The Boltzmann factor accounts for the potential-energy change, while the activity and particle-number factors account for exchange with the reservoir.

---

## 4. Python Coding Exercises

### Exercise 1: Ideal-Gas GCMC

I studied a simple Python implementation in which particles were inserted and deleted without intermolecular interactions.

For an ideal gas,

$$
\langle N\rangle=zV
$$

and the equilibrium particle-number distribution is Poisson:

$$
P(N)=\frac{(zV)^Ne^{-zV}}{N!}.
$$

Therefore,

$$
\operatorname{Var}(N)=\langle N\rangle.
$$

This provides a useful theoretical benchmark for validating a GCMC implementation.

### Exercise 2: Three-Dimensional Lennard-Jones Fluid

I then studied an extension of the code to an interacting Lennard-Jones fluid.

The pair potential is

$$
U(r)=4\epsilon\left[
\left(\frac{\sigma}{r}\right)^{12}
-\left(\frac{\sigma}{r}\right)^6
\right].
$$

The simulation includes three types of Monte Carlo moves:

1. Particle insertion
2. Particle deletion
3. Particle displacement

Periodic boundary conditions and the minimum-image convention are used to calculate pair interactions.

For an interacting fluid, the average particle number is not simply \(zV\), because the potential-energy changes influence the acceptance probabilities.

A useful quantity to investigate is

$$
R=\frac{\operatorname{Var}(N)}{\langle N\rangle}.
$$

For an ideal gas, \(R=1\). For interacting systems, this ratio can differ from unity.

**Coding status:** The example implementations and their statistical-mechanical basis were studied. Numerical validation and systematic parameter exploration remain follow-up exercises.

---

## 5. Understanding the Role of Chemical Potential

During the discussion, I initially predicted that increasing chemical potential would decrease the average number of particles.

After reviewing the grand canonical statistical weight and insertion probability, I understood why the opposite is true.

Increasing chemical potential favors configurations containing more particles.

For an ideal gas,

$$
\langle N\rangle=
\frac{V}{\Lambda^3}e^{\beta\mu}.
$$

More generally,

$$
\left(\frac{\partial\langle N\rangle}{\partial\mu}\right)_{T,V}
=
\beta\operatorname{Var}(N)\geq0.
$$

This fluctuation–response relationship shows that increasing chemical potential cannot decrease the equilibrium average particle number.

I then correctly predicted that increasing the water chemical potential would increase the average water occupancy of a protein binding pocket, or cause it to approach saturation.

---

## 6. Conceptual Questions and My Understanding

### Question 1: What happens to pocket occupancy when water chemical potential increases?

**My understanding:** The average water occupancy increases.

This follows because a higher chemical potential makes water insertion more favorable relative to deletion.

### Question 2: Would displacing a strongly hydrogen-bonded water molecule with a hydrophobic ligand group be favorable?

**My answer:** It might be energetically unfavorable because the water-mediated interactions could stabilize the protein–ligand complex. Additional favorable interactions would be needed to compensate for their loss.

**What I learned:** This reasoning is a useful starting point, but the complete thermodynamic picture also includes the interactions formed by displaced water in bulk solvent, new ligand–protein contacts, and entropy changes.

An enthalpically unfavorable displacement can still be favorable in terms of Gibbs free energy.

### Question 3: Which water should be prioritized for displacement?

Two hypothetical water molecules were considered:

- **Water A:** Forms several strong hydrogen bonds with protein residues.
- **Water B:** Occupies a hydrophobic cavity and has relatively unfavorable interactions compared with bulk water.

**My answer:** I would prioritize Water B because its interactions in the hydrophobic cavity are less favorable.

**What I learned:** Water B is the more promising initial candidate for displacement, but this does not guarantee improved ligand affinity. The complete binding free-energy difference must still be considered.

---

## 7. Connection to My Research Background

This lesson connects closely to my previous work in computational physics and molecular simulations.

My experience with molecular dynamics, statistical mechanics, and trajectory analysis provides a foundation for understanding GCMC and hydration thermodynamics.

Some particularly relevant quantities and concepts are:

- Water occupancy and density distributions
- Hydrogen-bond networks
- Water residence times and exchange kinetics
- Potential-energy changes associated with insertion and deletion
- Particle-number fluctuations and chemical potential
- Free-energy differences between ligand-bound states

I also learned that water occupancy and residence time alone cannot establish whether a hydration site is thermodynamically favorable to displace.

A possible future computational exercise is to introduce an attractive cavity into the Lennard-Jones GCMC model and investigate how particle occupancy depends on cavity interaction strength and chemical potential.

---

## 8. Key Takeaways

**Protein–ligand binding must be understood as a thermodynamic process involving the protein, ligand, and solvent together.**

Strong direct interactions do not necessarily guarantee high binding affinity.

Water molecules may stabilize a complex through hydrogen-bond networks, or their displacement may favor binding when they occupy thermodynamically unfavorable hydration environments.

GIST and WaterMap provide approaches for analyzing hydration thermodynamics, while GCMC enables sampling of particle insertion, deletion, and equilibrium occupancy.

The central statistical-mechanical insight is that chemical potential controls particle exchange with a reservoir, and increasing chemical potential increases equilibrium average occupancy.

Most importantly, the effect of a ligand modification must ultimately be evaluated through the overall binding free-energy change rather than any single interaction.

---

## 9. Next Steps

**Lesson 24: Binding Thermodynamics**

The next lesson will focus on:

- Gibbs free energy of protein–ligand binding
- The relationship between binding free energy and dissociation constant \(K_d\)
- Enthalpic and entropic contributions to binding
- Enthalpy–entropy compensation
- Why stronger hydrogen bonding does not always lead to stronger affinity
- Connections to molecular dynamics and free-energy calculations

**Lesson 23 Status: Completed**

