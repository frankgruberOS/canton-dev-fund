# Daml Reference Semantics: Machine-Checked Proofs for Daml Packages and the Canton Ledger Model

## Development Fund Proposal

**Organization:** Obsidian Systems  
**Author / Primary Contact:** Alexandre Oliveira, Obsidian Systems, alexandre.oliveira@obsidian.systems  
**Status:** Submitted  
**Created:** 2026-11-03  
**Proposal Type:** RFP-aligned  
**RFP / Roadmap Area:** Security, Assurance & Incident Readiness: RFP #22, Daml Security Standards and Secure Development  
**Champion:** Ali Abrar, Obsidian Systems (@ali-abrar)  
**Total Funding Request:** up to 4,000,000 Canton Coin (CC)  
**Project Duration:** 12 months  
**Label:** daml-tooling

---

## Abstract

Large Language Models make software defects cheaper to find and exploit, and write a growing share of code. Tests and fuzzers only check the inputs they try. A machine-checked proof covers every input the semantics accepts within its step bound, and Agda checks it no matter who, or what, wrote it. At this date, no public, executable, machine-checked semantics for Daml-LF 2.3 exists. Obsidian will build one in Agda with the Canton ledger model, under Apache 2.0. A comparator tests it against the Daml engine on 30 packages, and verification tools can test their models against it. An Agda theorem covers the code generator. Obsidian will also prove authorization soundness and 3 theorems on Splice code: the Canton Coin issuance bound, the app reward cap of Canton Improvement Proposal (CIP) 0098 and the CIP-0105 lock schedule. Obsidian earns 36.9% of the funding only when outside teams adopt the tool.

---

## Specification

### 1. Objective

Obsidian Systems will build one public machine-checked reference for Daml-LF 2.3 and the Canton ledger model. Daml verification tools can test their models of Daml against it. Auditors and other teams can prove theorems about their own Daml packages with it. A person, a tool or an AI can write a proof, and only the proof checker, here Agda, decides if the proof is valid [3].

### 2. Implementation Mechanics

**Semantics.** We will create Agda modules for Daml-LF 2.3 values, expressions, updates and interfaces, plus the authorization, consistency and privacy rules of the Canton ledger model. Each Daml update becomes a sequence of ledger commands: create, fetch and exercise. An interpreter answers each command, in the same way that the Daml engine asks its host for contracts and packages. The modules type-check under `--safe`. Evaluation takes a step bound because Daml-LF allows recursion. The comparator report shows when the bound runs out.

**Code generation.** The code generator is written in Agda and compiled to Haskell. It reads the decoded Daml-LF of a Daml package (DAR) and writes Agda modules for the proofs. A theorem in Agda shows that each generated term, with its types erased, equals the decoded Daml-LF in normalized form. The theorem covers the Agda source of the generator, for every package on which the generator returns a result. The compiled generator, the module printer and the definitions of erasure and normalization stay trusted, and a person reviews those definitions. Agda checks every generated module again under `--safe`.

**Runner.** Agda compiles the semantics to Haskell. One bridge module, outside `--safe`, loads DARs with Digital Asset's (DA) Haskell Daml-LF decoder (`daml-lf-proto-decode`) and passes the result to the semantics and to the generator. Obsidian writes no second decoder. Readers trust DA's decoder, and Obsidian pins its SDK version for each release.

**Comparator.** It runs each Daml Script test in the engine. It records the submitted commands and the full transaction, with fetch nodes. A test that produces a rollback node is reported as out of scope, because exceptions wait for Phase 2. The comparator replays the commands in the semantics and compares node by node.

**Theorem library.** The library holds machine-checked theorems about the ledger model and about real Splice packages, plus the generator theorem. Each statement names its source: a CIP, a docs page or a Daml file. Agda checks every proof. A person reviews every statement against its source, because a checker cannot tell if a statement says the right thing [1][3].

**Distribution.** Obsidian publishes the repository under Apache 2.0, with a trace-format spec, a tutorial and a plain-language report on the theorems. A command for the Daml Package Manager (`dpm`) lets a package team run the comparator in its own continuous integration (CI). One adapter replays the traces of another tool's model in the semantics.

**Trusted base**

A trusted base is the set of software and hardware that the project assumes to be correct, because no proof covers it. ARIA, the UK agency that funds the Safeguarded AI programme, states the limit: "A formal proof is only as strong as the assumptions behind it." [5] Fisher, Launchbury and Richards, in their paper on the DARPA HACMS program, give the same rule: "the results only apply if the assumptions are satisfied" [8]. The table lists the parts that a reader of a proof must trust, and the parts that the grant checks instead of trusting.

| Component | Role | Trusted? | How the grant limits the risk |
| :---- | :---- | :---- | :---- |
| Agda type checker | Checks every proof | Yes | Open source and independent of any AI vendor. Proof modules use `--safe`, so no postulate can hide a gap. Agda has no small separate kernel, so a defect in Agda stays a residual risk. |
| Agda standard library | Basic data types and lemmas | Yes | Pinned version. Proof modules import only `--safe` modules. |
| GHC (Haskell compiler) and MAlonzo (Agda to Haskell back end) | Build and run the generator, the semantics and the comparator | Yes | Pinned versions. Proofs run in the Agda checker, not through them. The comparator tests the compiled semantics, so its results trust both. Anthropic's proofs [1] trust the compiler of their checker in the same way. |
| Daml compiler (`damlc`) | Turns Daml source into Daml-LF | Yes | Proofs are about the Daml-LF in the published DAR, not about the Daml source. |
| DA Daml-LF decoder (`daml-lf-proto-decode`) | Reads DARs | Yes | DA's own decoder, not a second one. Pinned SDK version. |
| Bridge module | Connects DA's decoder to the semantics and the generator, outside `--safe` | Yes | One module. A person reviews it in each release. |
| Comparator and trace recorder | Compare the semantics with the engine | Yes | A defect can report false agreement. The 5 seeded defects test it. |
| Operating system and hardware | Run all tools | Yes | [1] trusts the hardware and the OS of its checker. [8] assumes that the hardware is correct. |
| Canton ledger guarantees | Unique contract IDs, one total order of transactions, atomic commit | Assumed, not proved | Each theorem names the guarantees that it uses. |
| Honest confirming participants | A separate Canton trust assumption | Assumed, not proved | Each theorem that needs it says so. |
| Theorem statements | Say what each proof means | Yes | A person reviews each one, with every definition and assumption it uses, against its named source: a CIP, a docs page or a Daml file. |
| Daml engine (Canton) | Runs Daml in production | Assumed to match the semantics | The comparator tests this on the Daml Script tests of the loaded packages. It does not prove it. |
| Code generator | Writes Agda modules from Daml-LF | Agda source: no. Compiled binary: yes | The generator theorem covers the Agda source. GHC and MAlonzo build the binary that runs. |
| AI agents (any vendor) | Can write code, definitions and proofs | No | Agda checks each proof. AI-written definitions and semantics code count only after a person reviews them. |

**Theorems Obsidian intends to prove**

| # | Property | Statement (informal) | Source | Why it needs induction |
| :---- | :---- | :---- | :---- | :---- |
| T1 | Authorization soundness (semantics) | Every transaction the semantics accepts is well-authorized under the ledger-model rule. | Ledger model | Quantifies over all Daml programs; induction on evaluation. |
| T2 | Issuance bound | The CC that coupons of one round can mint never exceeds the issuance configured for that round, if the round summary from the DSO (Decentralized Synchronizer Operations, the Super Validators that operate the Global Synchronizer) is accurate. | Global Synchronizer tokenomics | All rounds and all activity records; an unbounded sequence. |
| T3 | App reward cap | Each featured app reward stays within the configured CIP-0098 cap, at the CC price recorded for its round. | CIP-0098 | All states reachable from a stated initial state through the pinned packages. The statement fixes the Daml-LF Numeric rounding and any slack. |
| T4 | Lock release bound | Locked CC is released no faster than its vesting schedule allows, and only through the unlock paths that CIP-0127 defines. | CIP-0105, implemented in CIP-0127 | All sequences of lock, unlock and withdraw; induction on the transaction sequence. Ledger time is an input with a stated ordering assumption. |

T1 proves that the interpreter meets Obsidian's Agda statement of the ledger-model rule. A person reviews that statement against the docs page. T2 to T4 rest on that checked base. They are theorems about the Agda modules that the generator writes from published Splice DARs. Each holds for every input and transaction sequence that the semantics accepts, at any step bound. They say nothing about a run that exhausts the bound.

T4 is proved against the Daml lock code for CIP-0127, which Obsidian and DA are writing and which is not released yet. That code finds and merges locks through non-unique contract keys, so the semantics adds them.

### 3. Architectural Alignment

RFP #22 asks for "testing methodologies, automated analysis," "reference implementations, and CI/CD integration."

- Reference implementation. The semantics is a public, executable reference for the authorization, consistency and privacy rules of the Canton ledger model.
- Testing methodology. The comparator tests the semantics against the Daml engine, node by node. The semantics must reject 5 seeded authorization defects.
- Automated analysis. The comparator runs with no manual step. Every proof passes through the same Agda checker, whoever writes it. Formal proofs "can be easily and automatically audited by a third party" [7].
- CI/CD integration. A package team can run the comparator in its own CI pipeline and fail a build when the semantics and the engine disagree on a Daml Script test of the package.

### 4. Backward Compatibility

No backward compatibility impact. Smart Contract Upgrade does not apply because the grant produces no Daml contracts. A Phase 2 proposal will cover Smart Contract Upgrade in the semantics.

### Prior work

Obsidian used Agda on a production Daml system for a client. The proofs showed that an optimized code path computes the same results as the code it replaced. Machine-checked specifications covered the system's decision logic. The work is private. No public, executable, machine-checked Daml-LF semantics was available, so the proofs stopped at the Daml and Canton boundary and assumed how the ledger behaves. This grant builds that model.

---

## Milestones and Deliverables

| Milestone | Deliverable | Month | CC |
| :---- | :---- | :---- | :---- |
| M1 | Core semantics with the authorization and consistency rules + engine comparator for Daml-LF 2.3. Loads 10 real packages and agrees with the engine on their Daml Script tests. Rejects 5 seeded authorization defects. Public repository under Apache 2.0, CI green. | 3 | 885,000 |
| M2 | Code generator, written in Agda and compiled to Haskell, with the generator theorem. The generated modules for the 10 M1 packages type-check under `--safe`. | 5 | 310,000 |
| M3 | Interfaces, non-unique contract keys (lookupNByKey) and the privacy rules of the ledger model. Loads 30 real packages, with the CIP-0056 packages and the CIP-0127 lock package, and agrees with the engine on their Daml Script tests. `dpm` command, trace format, 1 adapter for another tool's model, tutorial. T1 type-checks under `--safe`. If the CIP-0127 lock code is not released by month 7, that package moves to M4. | 7 | 664,500 |
| M4 | Theorems on Splice code. T2 to T4 type-check under `--safe` against pinned Splice DARs. Each file names its source and assumptions. Plain-language report for reviewers who do not read Agda. Each theorem is 221,500 CC. If the CIP-0127 lock code is not released by month 9, T4's 221,500 CC pays at month 12. | 9 | 664,500 |
| M5 | Adoption. 246,000 CC per adopting outside team, up to 5, plus 246,000 CC at the fifth. | to 12 | up to 1,476,000 |
| **Total** |  |  | **up to 4,000,000** |

Every deliverable lands by month 9, unless the CIP-0127 lock code is late. Months 10 to 12 only count adoption, plus T4 in that case.

---

## Acceptance Criteria

- M1: The repository lists the package IDs of the 10 packages and publishes the 5 seeded defects. CI runs the comparator on every commit. For each Daml Script test, the comparator reports agreement node by node, and no run stops at the step bound. The semantics rejects each seeded defect.
- M2: The generator theorem type-checks under `--safe`. The generated modules of the 10 M1 packages type-check under `--safe`.
- M3: The M1 criteria hold for the 30 packages, or for 29 if the CIP-0127 lock package moves to M4. The semantics computes the informees of each node, and the comparator checks them against the engine. T1 type-checks under `--safe`. The tutorial, the trace-format spec, the `dpm` command and the adapter are public.
- M4: T2 to T4 type-check under `--safe` against the pinned package IDs. Each theorem file names its source and its assumptions. The plain-language report is public. If the lock package moved from M3, the M1 criteria hold for it. If T4 pays at month 12, T2 and T3 meet these criteria at month 9.

M5 (adopter): an outside team counts after 1 month using the tool on its own code or model in automated tests in a public repository, or after publishing a proof or audit report that uses it, confirmed to the Tech & Ops Committee. Obsidian's own use does not count. The adoption window runs to month 12. Unearned tranches lapse.

---

## Funding

**Total Funding Request:** up to 4,000,000 CC

| Milestone | Deadline | CC | Paid upon |
| :---- | :---- | :---- | :---- |
| M1 | Month 3 | 885,000 | Committee acceptance |
| M2 | Month 5 | 310,000 | Committee acceptance |
| M3 | Month 7 | 664,500 | Committee acceptance |
| M4 | Month 9 | 664,500 | 221,500 per theorem, on committee acceptance |
| M5 | To month 12 | up to 1,476,000 | 246,000 per adopting outside team, up to 5, plus 246,000 at the fifth |
| Total |  | up to 4,000,000 |  |

The grant gives every Daml verification tool one shared reference to test against. The network also gets machine-checked bounds on Canton Coin issuance, app rewards and locked CC. For comparison, the Canton Foundation approved Peaceful Studio's C#/.NET SDK (PR #46) at 2,705,000 to 2,855,000 CC and IntellectEU's Daml Code Assistant (PR #10) at 6,997,227 CC.

Post-grant: a separate annual retainer of 740,000 CC. Obsidian tracks each Daml-LF minor version in `LanguageVersion.scala` and each Splice release that changes a theorem target. Precedent: Obsidian's LMS Maintenance Grant (PR #459). The committee renews the retainer each year. If it declines, Obsidian transfers the repository to the Foundation's GitHub organization.

### Volatility Stipulation

The project duration is greater than 6 months. The grant is denominated in fixed Canton Coin. Obsidian may request a re-evaluation of the remaining milestones at month 9. The trigger is a CC price more than 25 percent above or below its price at approval. The Tech & Ops Committee runs the re-evaluation.

---

## Co-Marketing

At release, Obsidian will announce with the Foundation and publish a technical post.

---

## State of the art

Sources: proposal files in github.com/canton-foundation/canton-dev-fund, as of 2026-10-21.

| Project | Method | Guarantee | Funding |
| :---- | :---- | :---- | :---- |
| This proposal (Obsidian) | Machine-checked Daml-LF semantics in Agda, tested node by node against the engine. T1 on the semantics, T2 to T4 on Splice code. | On the semantics, not the engine: every input the semantics accepts, under stated assumptions. | Up to 4,000,000 CC, 12 months. 36.9% on adoption. |
| Certora Package Analyzer (#130, merged) | Static map of cross-package calls in a DAR. | None on properties. | 2,010,000 CC, 5 months. |
| Dedaub DamlSec (#194, closed) | Automatic symbolic execution of the compiled DAR, with an SMT (Satisfiability Modulo Theories) solver. | Bounded: "within the explored depth". | 250,000 CC, 14 weeks. |
| TKNFRA B-Method (#12, open) | Hand-written B models of CIP-0056 and CIP-0047. | Proof "where feasible" and bounded checks, on the model only. | 410,000 CC. |
| OpenZeppelin daml-verify (#262, merged) | Z3 over hand-written Python models of Daml logic. | On the model: "only as faithful as the model" [13]. | Part of the 28,378,378 CC OpenZeppelin grant. |
| Informal Systems Quint (#56, closed) | Quint model; traces replayed on Canton Sandbox. | Testing only. | USD 150,000. |
| Sigli.ai DAWE (#400, open) | Adversarial Daml Script tests. | "Not a proof of contract correctness". | 487,000 CC, 6 months. |
| ChainSafe corpus (#197, open) | AI retrieval corpus of Daml patterns. | Similarity score only. | 882,352 CC, 6 months. |
| PixelPlex cantonsim (#879, open) | Dry-run through the participant. | "A test standard, not a claim of correctness". | 2,400,000 CC, 6 months. |
| Quantstamp DamlFuzz (#52, open) | Stateful fuzzing on the engine. | Probabilistic. | 2,450,000 CC. |
| Quantstamp DamlCov (#323, open) | Test coverage by Daml-LF instrumentation. | None; a metric. | 1,300,000 CC. |
| Digital Asset research | Isabelle/HOL model of Canton Merkle trees [9]; ledger model started. | On the model. No Daml-LF semantics. | Not a grant. |

Tests, fuzzers, coverage and simulation (DAWE, DamlFuzz, DamlCov, cantonsim) run on the Daml engine and do not claim correctness for every input. DamlSec is automatic but bounded. TKNFRA, OpenZeppelin and Quint work on hand-written models, which can differ from the code. DA's proofs do not cover Daml-LF.

No other project in the table offers a machine-checked Daml-LF semantics tested against the engine. The budget covers the Daml-LF core with interfaces and contract keys, plus about 130 to 170 Splice and CIP-0127 definitions.

---

## Motivation

AI lowers the cost to find and exploit software defects [2][4], and AI writes a growing share of code: Google and Microsoft report that 25 to 30% of their new code is AI-generated [6]. In one month, with AI help, Mozilla found and fixed 423 Firefox security bugs, more than in the previous 15 months together [2]. AI can turn a new vulnerability into an attack in minutes, often for less than a dollar [4]. "[A]ttackers only need to find one good bug. Defenders need to find all of them." [2] Patching one bug at a time does not scale.

A proof rules out a whole class of defects [2], under its stated assumptions [7][8]. AI also makes proofs feasible at a larger scale. Anthropic's model patched the bugs it found, then wrote machine-checked proofs that 239 fuzz-test entry points in 123 open-source libraries never crash on any input up to 64 MB [1]. A team can prove theorems such as these about a financial product on Canton:
- A transfer never moves a holding without the authority of its owner.
- A split of a position never changes the total quantity.
- A lock never releases a holding earlier than its schedule allows.

One exploit can cost hundreds of millions of dollars and spread to other protocols. In April 2026, an attacker manipulated a cross-chain message and drained $292 million from the Kelp DAO bridge. The stolen tokens became collateral on Aave, so the loss spread into a second protocol's lending markets. Arkham Intelligence described one option: split the loss over all holders of the stolen token, rsETH, about 16% each. Preliminary findings link that attack to the Lazarus group, and the report mentions no use of AI [10].

The evidence on AI-written exploits comes from a simulator. There, AI agents wrote exploits worth $4.6 million against smart contracts that attackers broke after the models' training data ended. On the 2025 contracts, simulated exploit revenue doubled about every 1.3 months over one year [11]. In 2025, a group that Anthropic assesses as state-sponsored used Claude Code in an espionage campaign that tried to infiltrate about 30 targets, financial institutions among them. AI performed 80 to 90% of the campaign. It succeeded in a small number of cases [12].

On Canton, the parties are banks and market infrastructure. For them, one defect in a package can mean a direct loss and a loss of client trust. A proof covers the package logic only. It does not stop key theft or social engineering.

The Canton Foundation funded Certora's Daml Package Analyzer (PR #130). Dedaub's DamlSec (PR #194, closed) runs symbolic checks on Daml packages. OpenZeppelin published Z3 models of Daml logic [13]. Each of these tools works from its own interpretation of Daml, and no public reference exists to test those interpretations. A tool with a wrong model of Daml can report an unsafe package as safe. The comparator can help turn a wrong model into a failing test in CI.

OpenZeppelin checks each model "line-by-line against the Daml source" [13]. It builds the models by hand, and a formal semantics could be a first step toward automating the check. The grant targets adoption by up to 5 outside teams by month 12.

DA modeled the hierarchical transactions of Canton as authenticated data structures in Isabelle/HOL [9], and started to formalize the ledger model [14]. No public, executable, machine-checked Daml-LF 2.3 semantics exists (searched 2026-10-06).

---

## Rationale

An SMT solver such as Z3 decides one formula at a time within the encoded bounds. A property over all transaction sequences needs an inductive invariant that a person writes. The solver can return "unknown" for such properties. A proof assistant checks the induction argument itself. The grant's proofs are about the same semantics that the comparator tests against the engine.

A proof can come from an AI. The design keeps AI-written proofs outside the trusted base, and the checker comes from a party other than the AI vendor [6]. AI-written definitions and semantics code stay inside it until a person reviews them. Kleppmann states: "It doesn't matter if they hallucinate nonsense, because the proof checker will reject any invalid proof ..." [3]. A proof of the wrong statement can still pass. Anthropic reports a related risk for its own model: Claude could make a theorem provable by making the program trivially never crash [1]. In this grant, the DARs are pinned, and a person reviews each theorem statement, with every definition and assumption it uses, against its named source.

### Scope and Other Grants

A team writes each package proof with Obsidian's library, by hand or with AI help. The table shows how the grant relates to other work on Daml.

| Work | Owner | Relation |
| :---- | :---- | :---- |
| Cross-package authority in DARs | Certora (PR #130, funded) | Certora maps which package can call which. With Obsidian's semantics, an auditor can prove what such a call can do, for example that it never moves a holding without the owner's authority. |
| Symbolic bug detection on DARs | Dedaub DamlSec (PR #194, closed) | DamlSec can run the comparator to test its own model of Daml against Obsidian's semantics. |
| Lint, property tests, Z3 models | OpenZeppelin | Obsidian delivers 1 adapter in M3, chosen with the design partners (the outside teams that test early versions). Candidate: replay the Z3 models' counterexamples and witnesses in the semantics. |
| B-Method models of CIP-0056 and CIP-0047 | TKNFRA (PR #12, open) | With an adapter, the comparator can replay traces from those models in the semantics. Proving their invariants on the Daml code takes extra theorems, which TKNFRA can write with the library. |
| Transaction simulation | PixelPlex cantonsim (PR #879, open) | cantonsim can call the semantics to check authorization of each simulated transaction. |

## Constraints, Limitations and Out of Scope

The proofs are about the semantics, not about the Daml engine. The comparator tests the link between them on Daml Script tests and does not prove it. Each result holds within the step bound and under the assumptions in the trusted-base table. A proof covers the package logic. It does not cover key management, operations, the network or the Canton synchronization protocol, and it does not replace an independent security audit.

Out of scope, for a Phase 2 proposal: exceptions, Smart Contract Upgrade, multi-synchronizer reassignment, a throughput ceiling from the traffic rule, and off-ledger components with their ledger interactions. A test that needs one of these features is reported as out of scope and does not count toward agreement.

A Daml-LF feature outside 2.3, a Canton release that changes Daml-LF, or a package on which the generator returns no result triggers a re-scope with the Tech & Ops Committee. Each deadline shifts by the committee's review time for the previous milestone. If the CIP-0127 lock code is in no Splice release by month 12, the T4 tranche lapses. Unpaid M5 amounts lapse at month 12. Obsidian reports at milestone submissions and files no interim reports.

## References

[1] Anthropic Frontier Red Team (K. Lucas, A. Lynch, L. Tong, et al.), with Theorem. "LLMs are making formal software verification feasible at a much larger scale." Anthropic Research, 22 October 2026. https://www.anthropic.com/research/formal-verification. Claude wrote machine-checked Rocq proofs that 239 fuzz-test entry points in 123 OSS-Fuzz libraries never crash on any input up to 64 MB, after it patched the bugs it found. The authors state that "formally verified" does not mean "bug-free", and report the risk that the model games the theorem.

[2] S. Cuellar. "You Can't Patch Your Way Out of This." Galois, 13 May 2026. https://www.galois.com/articles/you-cant-patch-your-way-out-of-this. Argues that defenders cannot win by finding bugs faster, and that AI-enabled formal verification must rule out whole classes of attacks.

[3] M. Kleppmann. "Prediction: AI will make formal verification go mainstream." 8 December 2025. https://martin.kleppmann.com/2025/12/08/ai-formal-verification.html. Argues that proof scripts suit LLMs because the proof checker rejects every invalid proof, and that the challenge moves to the specification.

[4] E. Johnson and J. Cappos. "With $1 Cyberattacks on the Rise, Durable Defenses Pay Off." IEEE Spectrum, 30 April 2026. https://spectrum.ieee.org/ai-cyberattacks-memory-safe-code. Reports that AI can turn a new vulnerability into an attack in minutes, often for less than a dollar. Names memory-safe code and formal verification as defenses.

[5] ARIA (Advanced Research and Invention Agency). "Can AI help us prove software is secure?" 29 October 2026. https://aria.org.uk/insights/2026/can-ai-help-us-prove-software-is-secure. The Safeguarded AI programme funds eight projects (£22m) on AI-enabled formal methods. An independent Red Team will try to find weaknesses in their claims. Early results are preliminary.

[6] L. de Moura. "When AI Writes the World's Software, Who Verifies It?" 28 February 2026. https://leodemoura.github.io/blog/2026-2-28-when-ai-writes-the-worlds-software-who-verifies-it/. Cites Google and Microsoft reports that 25 to 30% of their new code is AI-generated. Argues for a small trusted proof checker, outside the AI, open source and independent of the AI vendor. The author is a co-founder of the Lean FRO.

[7] National Academies of Sciences, Engineering, and Medicine. Defense Software for a Contested Future: Agility, Assurance, and Incentives. Chapter 3, "Assurance". The National Academies Press, 2025. https://doi.org/10.17226/29129. Calls formal methods "the gold standard for creating high assurance software", with machine-checkable proofs that a third party can audit, and states their limits.

[8] K. Fisher, J. Launchbury and R. Richards. "The HACMS program: using formal methods to eliminate exploitable bugs." Philosophical Transactions of the Royal Society A 375(2104): 20150401, 2017. https://doi.org/10.1098/rsta.2015.0401. Peer-reviewed. In DARPA HACMS, a Red Team with root access to one Linux partition of a system on the verified seL4 kernel could not break out of it.

[9] A. Lochbihler and O. Marić (Digital Asset). "Authenticated Data Structures as Functors in Isabelle/HOL." 2nd Workshop on Formal Methods for Blockchains (FMBC 2020), OASIcs vol. 84, pp. 6:1–6:15, 2020. https://doi.org/10.4230/OASIcs.FMBC.2020.6. Models the hierarchical transactions of Canton as authenticated data structures in Isabelle/HOL, as a first step toward verifying the Canton protocol.

[10] K. Baird. "DeFi losses top $600 million in weeks as Kelp DAO exploit drags TVL to one-year low." The Block, 20 April 2026. https://www.theblock.co/news/defi/2026-04-20-defi-losses-1-billion-weeks-kelp-dao-exploit-drags-tvl-one-year-low-398154. An attacker manipulated a cross-chain message and drained $292 million in rsETH from the Kelp DAO bridge. The loss spread to Aave. Arkham Intelligence described one option: a loss of about 16% for every rsETH holder. Preliminary findings link the attack to the Lazarus group.

[11] W. Xiao, C. Killian, H. Sleight, A. Chan, N. Carlini and A. Peng (Anthropic Frontier Red Team, MATS and Anthropic Fellows). "AI agents find $4.6M in blockchain smart contract exploits." Anthropic Research, 1 December 2025. https://www.anthropic.com/research/smart-contracts. In blockchain simulators only, Claude Opus 4.5, Claude Sonnet 4.5 and GPT-5 wrote exploits worth $4.6 million against contracts exploited after the models' knowledge cutoffs. Exploit revenue on the 2025 problems doubled roughly every 1.3 months.

[12] Anthropic. "Disrupting the first reported AI-orchestrated cyber espionage campaign." 13 November 2025. https://www.anthropic.com/news/disrupting-AI-espionage. A group that Anthropic assesses as Chinese state-sponsored used Claude Code in a campaign that tried to infiltrate about 30 targets, financial institutions among them. AI performed 80 to 90% of the campaign. It succeeded in a small number of cases.

[13] A. Singh. "Smart Contract Security for Institutional Finance on Canton: An Entirely Different Problem." OpenZeppelin, 26 March 2026. https://www.openzeppelin.com/news/smart-contract-security-for-institutional-finance-on-canton-an-entirely-different-problem. OpenZeppelin builds Z3 models of Daml logic by hand. To limit model-code divergence, "every model is checked line-by-line against the Daml source".

[14] Digital Asset. "Research Publications." Canton Network documentation, accessed 25 November 2026. https://docs.canton.network/shared/research.md. The abstract of a CPP 2021 lightning talk states: "we have started to formalize the Daml ledger model and prove its properties in Isabelle/HOL."
