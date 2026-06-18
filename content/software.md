+++
title = "Software, Specifications, Proofs, etc."
path = "software"
+++

Are specifications and proofs software?

## TLA+ specifications

Various TLA+ specifications are available in the [TLA+ examples repository](https://github.com/tlaplus/Examples).
If you like consensus algorithms, two interesting ones are [Streamlet](https://github.com/nano-o/streamlet) and [DAG-based consensus](https://github.com/tlaplus/Examples/tree/master/specifications/dag-consensus).

## CPP-DPOR

A model-checking library for distributed protocols, inspired by [Must](https://dl.acm.org/doi/10.1145/3689778).
Uses dynamic partial-order reduction; parallel search never explores the same execution graph class twice, yet workers don't have to synchronize!
Successfully used to debug new features in [stellar-core](https://github.com/stellar/stellar-core/).
Sources are on github at [https://github.com/nano-o/CPP-DPOR](https://github.com/nano-o/CPP-DPOR).

## python-fbas

A prototype tool to reason about Federated Byzantine Agreement Systems using SAT solvers.
Sources are on github at [https://github.com/nano-o/python-fbas](https://github.com/nano-o/python-fbas).

## Formalization and proof of a classical distributed algorithm

[https://github.com/nano-o/Distributed-termination-detection](https://github.com/nano-o/Distributed-termination-detection)

The algorithm is the "channel-counting" termination-detection algorithm of Kumar (1985).
The proof I propose is a great example of inductive reasoning about distributed algorithms.
The [exercise branch](https://github.com/nano-o/Distributed-termination-detection/tree/exercise) of the repository sets things up as an exercise in finding inductive invariants.

## Accountable safety of Tendermint

An IVy model and proof of accountable safety for Tendermint.
Developed with help from [Benoit Razet](https://www.linkedin.com/in/benoit-razet/) while I was at Galois.
Sources are in the CometBFT repository at [https://github.com/cometbft/cometbft/tree/main/spec/ivy-proofs](https://github.com/cometbft/cometbft/tree/main/spec/ivy-proofs).
