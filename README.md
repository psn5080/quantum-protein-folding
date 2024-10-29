# Quantum Protein Folding

## Introduction
The structure and function of many natural and human-engineered proteins is still only poorly understood. As a result, our understanding of processes connected with protein folding, such as those encountered in Alzheimer’s disease, vaccine development, and crop improvement research, has remained limited.

Unfolded polypeptides have a very large number of degrees of freedom and thus an enormous number of potential conformations. For example, a chain with 100 amino acids has on the order of 10^47 conformations. In reality, however, many proteins fold to their native structure within seconds. This is known as Levinthal’s paradox [1].

The exponential growth of potential conformations with chain length makes the problem intractable for classical computers. In the quantum framework, our resource-efficient algorithm scales linearly with the number of amino acids N.

The goal of this work is to determine the minimum energy conformation of a protein. Starting from a random configuration, the protein's structure is optimized to lower the energy. This can be achieved by encoding the protein folding problem into a qubit operator and ensuring that all physical constraints are satisfied.

## Problem Encoding
For the problem encoding we use:
- **Configuration qubits**: qubits that are used to describe the configurations and the relative position of the different beads.
- **Interaction qubits**: qubits that encode interactions between the different amino acids.

For our case, we use a tetrahedral lattice (diamond shape lattice) where we encode the movement through the configuration qubits.

<img src="aux_files/lattice_protein.png" width="300">

The Hamiltonian of the system for a set of qubits $\mathbf{q}=\{\mathbf{q}_{cf}, \mathbf{q}_{in}\}$ is:

$$H(\mathbf{q}) = H_{gc}(\mathbf{q}_{cf}) + H_{ch}(\mathbf{q}_{cf}) + H_{in}(\mathbf{q}_{cf}, \mathbf{q}_{in}) $$

where:
- $H_{gc}$ is the geometrical constraint term (governing the growth of the primary sequence of amino acids without bifurcations).
- $H_{ch}$ is the chirality constraint (enforcing the right stereochemistry for the system).
- $H_{in}$ is the interaction energy terms of the system. In our case, we consider only nearest neighbor interactions.

Further details about the used model and the encoding of the problem can be found in [2].

## Protein Main Chain
The protein consists of a main chain that is a linear chain of amino acids. For the naming of different residues, we use the one-letter code as defined in Ref. [3]. Further details about the naming and the type of amino acids can also be found in [4].

For this particular case, we demonstrate the generation of the qubit operator in a neuropeptide with the main chain consisting of 7 amino acids with letter codes APRLRFY (see also [2]).

## Side Chains
Beyond the main chain of the protein, there may be amino acids attached to the residues of the main chain. Our model allows for side chains of the maximum length of one. Elongated side chains would require the introduction of additional penalty terms which are still under development. In this example, we do not consider any side chains to keep the real structure of the neuropeptide.

## Interaction between Amino Acids
For the description of inter-residue contacts for proteins, we use knowledge-based (statistical) potentials derived using quasi-chemical approximation. The potentials used here are introduced by Miyazawa, S. and Jernigan, R. L. in [5].

Beyond this model, we also allow for random contact maps (interactions) that provide a random interaction map. One can also introduce a custom interaction map that enhances certain configurations of the protein (e.g., alpha helix, beta sheet, etc).

## Physical Conditions
To ensure that all physical constraints are respected, we introduce penalty functions. The different penalty terms used are:
- **penalty_chiral**: A penalty parameter used to impose the right chirality.
- **penalty_back**: A penalty parameter used to penalize turns along the same axis. This term is used to eliminate sequences where the same axis is chosen twice in a row. In this way, we do not allow for a chain to fold back into itself.
- **penalty_1**: A penalty parameter used to penalize local overlap between beads within a nearest neighbor contact.

## Peptide Definition
Based on the main chain and possible side chains, we define the peptide object that includes all the structural information of the modeled system.

## Protein Folding Problem
Based on the defined peptide, the interaction (contact map), and the penalty terms we defined for our model, we define the protein folding problem that returns qubit operators.

## Using VQE with CVaR Expectation Value for the Solution of the Problem
The problem that we are tackling has now implemented all the physical constraints and has a diagonal Hamiltonian. For the particular case, we are targeting the single bitstring that gives us the minimum energy (corresponding to the folded structure of the protein). Thus, we can use the Variational Quantum Eigensolver with Conditional Value at Risk (CVaR) expectation values for the solution of the problem and for finding the minimum configuration energy [6]. We follow the same approach as in Ref. [2] but here we use COBYLA for the classical optimization part. One can also use the standard VQE or QAOA algorithm for the solution of the problem, though as discussed in Ref. [2], CVaR is more suitable.

## Visualizing the Answer
In order to reduce computational costs, we have reduced the problem's qubit operator to the minimum amount of qubits needed to represent the shape of the protein. In order to decode the answer, we need to understand how this has been done.
- The shape of the protein has been encoded by a sequence of turns, $\{0,1,2,3\}$. Each turn represents a different direction in the lattice.
- For a main bead of $N_{aminoacids}$ in a lattice, we need $N_{aminoacids}-1$ turns in order to represent its shape. However, the orientation of the protein is not relevant to its energy. Therefore, the first two turns of the shape can be set to $[1,0]$ without loss of generality.
- If the second bead does not have any side chain, we can also set the $6^{th}$ qubit to $[1]$ without breaking symmetry.
- Since the length of the secondary chains is always limited to $1$, we only need one turn to describe the shape of the chain.

The total amount of qubits we need to represent the shape of the protein will be $2(N_{aminoacids}-3)$ if there is a secondary chain coming out of the second bead or $2(N_{aminoacids}-3) - 1$, otherwise. All the other qubits will remain unused during the optimization process.

## References
1. [Levinthal’s paradox](https://en.wikipedia.org/wiki/Levinthal%27s_paradox)
2. A. Robert, P. Barkoutsos, S. Woerner, and I. Tavernelli, "Resource-efficient quantum algorithm for protein folding," NPJ Quantum Information, 2021, [https://doi.org/10.1038/s41534-021-00368-4](https://doi.org/10.1038/s41534-021-00368-4)
3. IUPAC–IUB Commission on Biochemical Nomenclature (1972). "A one-letter notation for amino acid sequences". Pure and Applied Chemistry. 31 (4): 641–645. doi:10.1351/pac197231040639. PMID 5080161.
4. [Amino acid](https://en.wikipedia.org/wiki/Amino_acid)
5. S. Miyazawa and R. L. Jernigan, "Residue – Residue Potentials with a Favorable Contact Pair Term and an Unfavorable High Packing Density Term for Simulation and Threading," J. Mol. Biol. 256, 623–644, 1996, Table 3, [https://doi.org/10.1006/jmbi.1996.0114](https://doi.org/10.1006/jmbi.1996.0114)
6. P. Barkoutsos, G. Nannichini, A. Robert, I. Tavernelli, S. Woerner, "Improving Variational Quantum Optimization using CVaR," Quantum 4, 256, 2020, [https://doi.org/10.22331/q-2020-04-20-256](https://doi.org/10.22331/q-2020-04-20-256)