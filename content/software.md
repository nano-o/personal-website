+++
title = "Software, Specifications, Proofs, etc."
path = "software"
+++

Are specifications and proofs software?

## TLA+ specifications

Various TLA+ specifications are available in the [TLA+ examples repository](https://github.com/tlaplus/Examples).
If you like consensus algorithms, two interesting ones are [Streamlet](https://github.com/nano-o/streamlet) and [DAG-based consensus](https://github.com/tlaplus/Examples/tree/master/specifications/dag-consensus).

## python-fbas

A prototype tool to reason about Federated Byzantine Agreement Systems using SAT solvers.
Sources are on github at [https://github.com/nano-o/python-fbas](https://github.com/nano-o/python-fbas).

## Formalization and proof of a classical distributed algorithm

[https://github.com/nano-o/Distributed-termination-detection](https://github.com/nano-o/Distributed-termination-detection)

The algorithm is the "channel-counting" termination-detection algorithm of Kumar (1985).
The proof I propose is a great example of inductive reasoning about distributed algorithms.
The [exercise branch](https://github.com/nano-o/Distributed-termination-detection/tree/exercise) of the repository sets things up as an exercise in finding inductive invariants.
