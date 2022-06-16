# Not-All-Equal 3-Satisfiability (NAE3SAT)

NAE3SAT is an [NP-complete](https://en.wikipedia.org/wiki/NP-completeness)
Boolean satisfiability (SAT) problem that has clauses of three literals each and
requires that for every clause the literals are not all equal to each other; i.e.,
all configurations are valid except `(-1, -1, -1)` and `(+1, +1, +1)` (for
spin-valued variables, or `(0,0,0)` and `(1, 1, 1)` for binary-valued variables).

---
**SAT Terminology:**

The [SAT problem](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem)
problem is to decide whether the literals in its clauses can be assigned values
that satisfy all the clauses. In CNF, the SAT is satisfied only if all its
clauses are satisfied.

 * *Literal* is a Boolean variable such as `x`  and its negation.
 * *Clause* is a disjunction of literals such as `x1 OR x0`.
 * *conjunctive normal form (CNF)* conjoins clauses by the AND operator; i.e.,
   (clause 1) AND (clause 2) AND (clause 3).

---

NAE3SAT problems are known to transition from the satisfiable to the unsatisfiable
regime at a clause-to-variable ratio `ρ=2.1`. This code example solves problems
with `ρ=2.1` (critical point) as well as with `ρ=3.0` (max-SAT regime).

This example explores solving this satisfiability problem with two different
generations of D-Wave quantum computers:

* A Pegasus-[topology](https://docs.ocean.dwavesys.com/en/stable/concepts/topology.html)
  Advantage system.
* An experimental prototype of a Zephyr-topology quantum processing unit (QPU)
  for the Advantage2 next-generation system currently under development.

## Installation

You can run this example
[in the Leap IDE](https://ide.dwavesys.io/#https://github.com/dwave-examples/NAE3SAT).

Alternatively, install requirements locally (ideally, in a virtual environment):

    pip install -r requirements.txt

## Usage

To run the example:
```bash
python nae3sat_example.py
```

The code saves the resulting graphics under a `plots` folder.

## Code Overview

The code reformulates NAE3SAT instances as Ising problems, embeds these onto the
QPU graphs, and analyzes the solution quality obtained from both QPUs.

### Ising Formulation

NAE3SAT problems can be mapped to the
[Ising model](https://docs.ocean.dwavesys.com/en/stable/concepts/bqm.html) by
anti-ferromagnetically coupling the variables of a clause. For example, a clause
for variables `(s0, s1, s2)` is represented by the Ising Hamiltonian,

`H(s0, s1, s2) = s0*s1 + s1*s2 + s0*s2`.

The table below shows the energy for all configurations of the variables. The
valid not-all-equal configurations have a lower energy than the all-equal ones. Therefore, the NAE3SAT clause is solved when minimizing the Hamiltonian objective.

|s0| s1|s2|E|
|---:|---:|---:|---:|
|-1| -1| -1|3|
| 1| -1| -1|-1|
| 1|  1| -1|-1|
|-1|  1| -1|-1|
|-1|  1|  1|-1|
| 1|  1|  1|3|
| 1| -1|  1|-1|
|-1| -1|  1|-1|

Negated variables are represented by a
[spin-reverse transform](https://docs.dwavesys.com/docs/latest/handbook_qpu.html)
of such variables. For example, a clause with `-s1`, the negation of `s1`,
is represented by the Ising Hamiltonian,

`H(s0, -s1, s2) = -s0*s1 - s1*s2 + s0*s2`.

A NAE3SAT problem is generated by adding all its clauses together; for example:

`H(s0,..., sN) = H(s0, s1, s2) + H(s6, -s1, s4) + H(-s3, s5, -s9) + ...`

The minimization of this objective solves the NAE3SAT problem.

For more information on reformulating SAT problems in Ising format, see the
[system documentation](https://docs.dwavesys.com/docs/latest/handbook_reformulating.html).

### Embedding

Both the Pegasus and Zephyr graphs natively support triangles, enabling the
[embedding](https://docs.ocean.dwavesys.com/en/stable/concepts/embedding.html)
of three-variable clauses without chains. Nevertheless, large NAE3SAT problems
typically include interactions between variables that are not natively connected
and do require chains.

The graphic below shows the distribution of chain lengths for a `ρ=3.0` NAE3SAT
problem of 75 variables minor-embedded into the two topologies using Ocean software's
[minorminer](https://docs.ocean.dwavesys.com/en/stable/docs_minorminer/source/sdk_index.html)
heuristic. Smaller chains typically improve the success of quantum-annealing computations.

![](/readme_images/rho_300_chain_length.png)

The Zephyr topology of the Advantage2 prototype has a greater connectivity than
the Pegasus topology of Advantage. This enables more-compact embedding (minor
embeddings with shorter chains) of problems in the Advantage2 prototype, which in turn improves the performance of the QPU on embedded problems.

### Solution Analysis

The graphic below shows the solution quality of 100 samples from a `ρ=3.0` problem.

![](/readme_images/rho_300_energies.png)

The distribution of samples obtained by the Advantage2 prototype tends to have
lower energy than that of the Advantage QPU. Lower energies mean better solutions.

---
**Note:** The data displayed in the graphics above was generated for a particular execution of this code example; results will slightly vary from run to run. If you wish to experiment further to get more consistent results, you can increase the number of samples or the number of calls (keep in mind these runs will consume your solver access time at the standard rate).


---

## License

Released under the Apache License 2.0. See [LICENSE](LICENSE) file.
