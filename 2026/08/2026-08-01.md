# MathLab Lab Notes --- 2026-08-01

## Focus

Today's work shifted MathLab from evaluating a single tutoring
conversation toward building the foundation for a reusable evaluation
framework.

The central idea is that **pedagogical decision-making remains the
research target**, while the evaluation engine becomes the laboratory
used to study it.

## Completed Today

### Evaluation infrastructure

-   Added evaluation types (`src/evals/types.ts`)
-   Added first scripted trajectory: **Persistent Stuckness**
-   Implemented `evaluator.ts`
-   Added Markdown evaluation report generation
-   Introduced the `TutorProvider` abstraction
-   Implemented a provider-independent runner
-   Successfully evaluated Gemma end-to-end

Observed tutoring sequence:

    productive → affirm
    productive → probe
    stuck → recall
    stuck → nudge

Scores:

-   State: 100%
-   Intervention: 100%
-   Content: 100%
-   Overall: 100%

## Key Architectural Decision

Research target: - Adaptive pedagogical decision-making.

Infrastructure: - Generic evaluation engine.

The evaluation engine exists to make pedagogical experiments repeatable,
measurable, and reproducible.

## Next Milestone

Generalize the runner into an Experiment Engine:

    Experiment
    ├── Provider
    ├── Runs
    └── Trajectories
          └── Turns

Initial benchmark suite:

-   Persistent Stuckness
-   Misdirection
-   Recovery
-   Near Solution

Later additions:

-   Induction
-   Pigeonhole Principle
-   Invariants
-   Contradiction
-   Construction Problems

## Milestone

MathLab now has:

-   tutoring policy
-   reusable evaluation framework
-   provider abstraction
-   automated Markdown reports
-   end-to-end benchmark pipeline

Next session: implement the experiment engine with multiple
trajectories, repeated stochastic runs, and aggregate statistics.
