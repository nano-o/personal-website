+++
title = "Signature-Free BFT Consensus in Three Steps"
date = 2026-06-16
path = "blog/how-forget-it-commits-in-three-steps"

[extra]
katex = true
categories = ["bft", "consensus", "signature-free agreement"]
+++

**Update, June 16, 2026:** Revised the framing after first publication.

# Introduction

Recently, Abraham et al. presented [Forget-IT](https://eprint.iacr.org/2026/355), a new partially-synchronous, signature-free BFT consensus protocol for $n=3f+1$ parties among which $f$ may be Byzantine.
Forget-IT achieves two properties never achieved in combination before[^it-protocols]: (a) when the network is synchronous, it commits a correct leader's proposal in just three message delays, and (b) even in the worst case, it sends only $O(n^2)$ bits per view.
Having tried and failed to obtain such a protocol before, I set out to understand the core idea behind it.
[^it-protocols]: A solution that commits in three message delays in the good case but sends $O(n^3)$ bits per view in the worst case appears in Chapter 3 of [the PhD thesis of Miguel Castro](https://pmg.csail.mit.edu/~castro/thesis.pdf). Solutions with $O(n^2)$ bits per view but more than three message delays in the good case include [TetraBFT](https://dl.acm.org/doi/10.1145/3662158.3662783) and [IT-HS](https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.OPODIS.2020.11).

I initially framed this blog post as distilling Forget-IT down to the core mechanism that allows it to achieve points (a) and (b).
However, after first publishing this blog post, I realized that I must have had elements of [IT-Kuplex](https://decentralizedthoughts.github.io/2026-06-05-IT-Kuplex/) in mind too.
So, let us say that we are going to sketch how to obtain properties (a) and (b) using simple quorum patterns inspired by both works.

As Eli Gafni would say, the key to solving consensus is to first solve the adopt-commit problem[^gafni-rrfd].
In fact, solving our consensus problem reduces to solving adopt-commit with a good-case latency of two message delays while sending $O(n^2)$ bits in the worst case.
Essentially, each consensus view can be implemented using two consecutive instances of adopt-commit: the first to try to commit the leader's proposal, the second to lock the committed value, if any, before the next view (we leave the details as an exercise to the reader).
So we will now focus on solving adopt-commit, and this will expose clearly some interesting quorum patters found in Forget-IT and IT-Kuplex.

[^gafni-rrfd]: Adopt-commit was first presented at PODC 1998 by Eli Gafni in [*Round-by-round fault detectors: unifying synchrony and asynchrony*](https://dl.acm.org/doi/abs/10.1145/277697.277724).

# Adopt-Commit

Adopt-commit is a single-shot abstraction that, unlike consensus, is solvable asynchronously even with failures.

In the adopt-commit problem, each party receives an input value and must eventually *commit* or *adopt* a unique value, such that:

- Validity: if a correct party commits or adopts $v$, then $v$ is the input of a correct party.
- Agreement: if a correct party commits $v$, then no correct party commits or adopts a different value.
- Unanimity: if all correct parties have the same input value, then then all commit that value.

Here we consider a slightly relaxed variant of adopt-commit, which we call adopt-commit*, where parties do not terminate after adopting or committing a value and where we allow a party to adopt and then commit the same value.
This simplifies the solution and works fine if the goal is to use it to implement consensus.

# Adopt-Commit* in two message delays and $O(n^2)$ communication

Now let us consider a message-passing system consisting of $n$ parties among which at most $f=\lfloor\frac{n-1}{3}\rfloor$ may be Byzantine while the others are "correct" (meaning they follow the protocol and keep taking steps).
We assume that the system is asynchronous: parties have no local clocks and message delay is unbounded.
However, the network is reliable: every message sent from a correct party to a correct party is eventually received.

Our task is to solve the adopt-commit* problem such that:
1. If all correct parties receive the same input (the *good case*), then they all commit in two message delays.
2. In the worst case, correct parties together send at most $O(n^2)$ bits in total.

Here is an algorithm. It evolves in two phases.
First, each party broadcasts a vote for its input.
Second, parties exchange three types of message to let each other know what votes they received:

- *Commit messages.* A party sends a commit message for a value $v$ when it has received votes for $v$ from $n-f$ parties and it has not previously sent a no-core message or a commit or candidate message for a different value.
- *Candidate messages.* A party sends a candidate message for a value $v$ when it has received votes for $v$ from $f+1$ parties (excluding parties that sent multiple votes) and it has not previously sent a commit message for a different value.
- *No-core messages.* A party sends a no-core message when it has received votes from $n-f$ parties (excluding parties that sent multiple votes) and, among those votes, no value got votes from $n-2f$ or more parties, and the party has not previously sent a commit message.

Finally, parties output according to the following rules:

- If a party receives commit messages for $v$ from $n-f$ parties, it commits $v$.
- If a party has not output yet and it receives commit or candidate messages for $v$ from $n-f$ parties, it adopts $v$.
- If a party has not output yet and it receives no-core messages from $n-f$ parties, it adopts its own input.

# Correctness proof

We must show the following:
- The algorithm satisfies the validity and agreement properties.
- Every correct party eventually adopts or commits a value.
- If all correct parties have the same input, then they all commit in two message delays.
- Correct parties together send $O(n^2)$ bits in the worst case.

First, let us define some vocabulary.
Call sets of at least $n-f$ parties *quorums*, sets of at least $n-2f$ parties *cores*, and sets of at least $f+1$ parties *validity witness sets*.
Say a set is correct when all its members are correct.
The algorithm depends on the following properties of quorums and cores.
- *Quorum intersection:* Every two quorums have a correct party in common (because $2(n-f)-n>f$).
- *Quorum availability:* Correct parties form a quorum.
- *Quorum validity:* Every quorum contains a core of correct parties (because $(n-f)-f=n-2f$).
- *Witness-set validity:* Every witness set contains a correct party.
- *Witness subsumption:* Every core set is a witness set (because $n-2f\geq f+1$).
- *Core/quorum intersection:* Every correct core set and every quorum have a correct party in common (because $(n-f)+(n-2f)-n>0$).

## Validity and Agreement

Validity is trivial.
Agreement is not too hard:
Note that no correct party broadcasts both a commit message for a value $v$ and either a commit/candidate message for a different value or a no-core message (this follows purely from the local rules a party follows).
Thus, by quorum intersection, it is impossible for a correct party to commit a value $v$ and another correct party to commit or adopt a different value.

## Unanimity and good-case latency

Next, let us show that, if all correct parties have the same input, then they all commit in two message delays.
This covers both the unanimity property and the good-case latency.
Suppose all correct parties have the same input $v$.
It follows that:
- By witness-set validity, no correct party ever broadcasts a candidate message for a value other than $v$.
- By quorum validity, no correct party ever broadcasts a commit message for a value other than $v$.
- By quorum validity, no correct party ever broadcasts a no-core message.

Thus, by quorum availability, every correct party will eventually broadcast a commit message for $v$ and will eventually commit $v$ after two message delays.

## Liveness

It remains to show that every correct party eventually commits or adopts a value.
Consider the following two exhaustive cases.

First, assume there is a value $v$ such that at least $n-2f$ correct parties (a core set) have input $v$.
Then, by core/quorum intersection, no correct party broadcasts a commit message for a value other than $v$.
Thus, by witness subsumption, every correct party eventually broadcasts a candidate message for $v$ and, by quorum availability, every correct party adopts $v$ unless it has already adopted a value.

Second, assume there is no value $v$ such that at least $n-2f$ correct parties have input $v$.
Then, by quorum validity, no correct party ever broadcasts a commit message.
Moreover, every correct party eventually receives the votes from all correct parties, which form a quorum in which no subset of $n-2f$ parties voted for the same value.
Hence, every correct party broadcasts a no-core message and then adopts its own input, unless it has already adopted a value.

In both cases, we concluded that every correct party adopts a value, and so we are done.

## Communication complexity

Finally, we must show that correct parties only send $O(n^2)$ bits in total.
Assuming values are of constant size, it suffices to observe that each party broadcasts at most six messages: one vote message, one no-core message, one commit message, and three candidate messages (each distinct candidate needs a disjoint set of $f+1$ non-equivocating voters; since $n\leq 3(f+1)$, there can be at most three).

Note that, if we used $f<\lfloor\frac{n-1}{3}\rfloor$, we would increase the number of possible candidate messages.
In the general case of $f<\frac{n}{3}$ we may get a linear number of candidate messages and an overall bit complexity of $O(n^3)$.

## Mechanically-checked proofs in Ivy

A formalization in Ivy is included below, and mechanically-checked proofs of safety and liveness are available at <https://github.com/nano-o/2-step-sf-bft-adopt-commit>.

# Epilogue

So what is the key insight?
I think it is the 3-way case split and the three corresponding message types:
- If all correct parties have the same input $v$, then every correct party gets a quorum of "commit" messages for $v$ in two message delays.
- If some value $v$ has a correct core ($n-2f$ correct processes with the same input), then that core prevents commits for conflicting values, and we eventually get a quorum of candidate messages for $v$.
- If no correct core has the same input, then we eventually get a quorum of "no-core" messages.

On top of that, local exclusion rules prevent a party from sending both a commit message and either a commit/candidate message for a different value or a no-core message; thus, by quorum intersection, we have agreement.

# Appendix: Ivy protocol model

For reference, here is the core Ivy model of the protocol.

```ivy
#lang ivy1.7

################################################################################
# Types
################################################################################

type party
type val

# The set types below abstract the thresholds.  Cardinalities are not encoded
# directly; the axioms capture the threshold facts used by the protocol for n >=
# 3f+1 and at most f Byzantine parties.

# A quorum represents at least n-f distinct senders.
type quorum

# A core represents at least n-2f distinct senders.
type core

# A witness set represents at least f+1 distinct senders.
type witness

################################################################################
# Immutable state and quorum theory
################################################################################

relation faulty(P:party)

relation quorum_member(Q:quorum, P:party)
relation core_member(S:core, P:party)
relation witness_member(S:witness, P:party)

# Quorum intersection
axiom [quorum_intersection]
    exists P. ~faulty(P) & quorum_member(Q1, P) & quorum_member(Q2, P)

# Quorum availability
axiom [non_faulty_quorum]
    exists Q. forall P. quorum_member(Q, P) -> ~faulty(P)

# Every quorum contains a core of nonfaulty members.
axiom [quorum_contains_non_faulty_core]
    exists S. forall P.
        core_member(S, P) -> ~faulty(P) & quorum_member(Q, P)

# Every witness has a nonfaulty member.
axiom [witness_has_non_faulty_member]
    exists P. ~faulty(P) & witness_member(S, P)

# Every core contains a witness.
axiom [core_contains_witness]
    exists W. forall P. witness_member(W, P) -> core_member(S, P)

# A nonfaulty core intersects a quorum in a nonfaulty party.
axiom [core_quorum_intersection]
    (forall P. core_member(S, P) -> ~faulty(P))
    ->
    exists P. ~faulty(P) & core_member(S, P) & quorum_member(Q, P)

################################################################################
# Mutable protocol state
################################################################################

# First-round value votes.
relation vote(P:party, V:val)

# Second-round support.
relation candidate(P:party, V:val)
relation commit(P:party, V:val)
relation no_core(P:party)

# Output state.
relation commit_out(P:party, V:val)
relation adopt_support_out(P:party, V:val)
relation adopt_no_core_out(P:party, V:val)

################################################################################
# Derived predicates
################################################################################

relation adopt_out(P:party, V:val)
definition adopt_out(P, V) = adopt_support_out(P, V) | adopt_no_core_out(P, V)

relation output(P:party, V:val)
definition output(P, V) = commit_out(P, V) | adopt_out(P, V)

relation started(P:party)
definition started(P) = exists V. vote(P, V)

relation correct_input(V:val)
definition correct_input(V) = exists P. ~faulty(P) & vote(P, V)

################################################################################
# Initialization
################################################################################

after init {
    vote(P, V) := false;
    candidate(P, V) := false;
    commit(P, V) := false;
    no_core(P) := false;
    commit_out(P, V) := false;
    adopt_support_out(P, V) := false;
    adopt_no_core_out(P, V) := false;
}

################################################################################
# Protocol actions
################################################################################

action start_step(p:party, v:val) = {
    require ~vote(p, V);
    vote(p, v) := true;
}

action commit_step(p:party, v:val, q:quorum) = {
    require quorum_member(q, P) -> vote(P, v);
    require ~commit(p, V);
    require ~no_core(p);
    require candidate(p, V) -> V = v;
    commit(p, v) := true;
}

action candidate_step(p:party, v:val, w:witness) = {
    require witness_member(w, P) -> vote(P, v);
    require commit(p, V2) -> V2 = v;
    candidate(p, v) := true;
}

action no_core_step(p:party, q:quorum) = {
    require ~commit(p, V);
    require (forall P. quorum_member(q, P) -> started(P));
    require (forall B.
        (forall P . core_member(B, P) -> quorum_member(q, P))
        -> (forall V . exists P . core_member(B, P) & ~vote(P,V)));
    no_core(p) := true;
}

action output_commit_step(p:party, v:val, q:quorum) = {
    require quorum_member(q, P) -> commit(P, v);
    commit_out(p, v) := true;
}

action output_adopt_step(p:party, v:val, q:quorum) = {
    require ~(adopt_out(p, V) | commit_out(p, V));
    require quorum_member(q, P) -> candidate(P, v) | commit(P, v);
    adopt_support_out(p, v) := true;
}

action output_no_core_step(p:party, v:val, q:quorum) = {
    require ~(adopt_out(p, V) | commit_out(p, V));
    require vote(p, v);
    require quorum_member(q, P) -> no_core(P);
    adopt_no_core_out(p, v) := true;
}

# Byzantine parties may equivocate and may change their sent-message relations
# arbitrarily.
action byz_party(p:party) = {
    require faulty(p);
    vote(p, V) := *;
    candidate(p, V) := *;
    commit(p, V) := *;
    no_core(p) := *;
}
```
