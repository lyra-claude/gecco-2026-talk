# GECCO 2026 -- Narration Per Slide (TTS-Ready)

One section per slide. No visual notes, no metadata. Spoken text only.

---

## Slide 1

Good morning. I'm Robin Langer, and this is joint work with Claudius Turing and Lyra Vega. Our paper is called "Composition Determines Diversity," and the core claim is: how you wire your islands matters more than what you evolve. That's a strong claim. Let me show you the evidence.

---

## Slide 2

Here is the result. Six completely unrelated domains -- from trivial bit-counting to co-evolutionary card games. Five migration topologies. Nine hundred experimental runs. And in every single domain, the diversity ordering is identical: no migration, then ring, then star, then random, then fully connected. Kendall's W equals one-point-zero -- perfect concordance. The p-value is 0.00008. Topology explains 23.9 times more variance in diversity than which domain you're solving. Domain is noise. Topology is signal. Now let me explain why.

---

## Slide 3

If you're a GA practitioner, you've spent weeks tuning operators -- crossover type, mutation rate, population size. Our result says that's optimizing the wrong thing. The composition structure -- how you wire your operators together, how your islands communicate -- dominates everything else. But first, we need to understand what "composition" means formally.

---

## Slide 4

Every genetic algorithm composes operators. Select, crossover, mutate, replace -- that's a pipeline. But look at how it appears in typical imperative code. This is a real Rust implementation from our maze evolution system. The pipeline is buried. Selection returns an index, not an individual. The sequence exists only as ordered statements. Randomness, configuration, and logging are threaded as separate arguments everywhere. The composition has no formal representation. It's invisible.

---

## Slide 5

Same algorithm. Six lines. Each line is a morphism -- a typed, composable operation. The triple-arrow operator is Kleisli composition: it chains these operations while automatically threading randomness, configuration, and logging through a monad. The important thing is not that it's shorter. It's that the composition is now a first-class value. You can see it. You can type-check it. You can compose it further -- lift it into an island model, nest it inside an adaptive strategy. The structure was hidden. Now it's visible. And once it's visible, you can reason about it.

---

## Slide 6

The framework has three composition levels. Level 1: operators compose into a pipeline -- one generation step. Level 2: pipelines compose into strategies -- generational, island, hourglass, adaptive. Level 3: strategies compose into multi-population architectures. What makes this a design algebra rather than notation: the composite at every level has the same type as its parts. A pipeline and a strategy are both arrows from populations to populations. You can always compose further.

---

## Slide 7

Category theory predicts structure lives in the arrows, not the objects. And that's exactly what we see. Checkers uses real-valued weight vectors and win-rate fitness. Mazes use binary vectors and path-length fitness. Incompatible types, incommensurable fitness. But the pipeline shape, the monad stack, the composition operator -- all identical. Which raises the natural question: if composition structure is domain-invariant, are its effects on diversity also domain-invariant?

---

## Slide 8

To test this, we chose six domains that share nothing in content. OneMax is trivially unimodal. Maze generation is multi-objective. Graph coloring is constraint satisfaction. Knapsack has epistatic interactions. No Thanks is co-evolutionary -- no fixed fitness landscape. Checkers has intransitive fitness: A beats B, B beats C, C beats A. If composition determines diversity across all six, it cannot be a landscape artifact.

---

## Slide 9

Five islands, sixteen individuals each, one hundred generations. Migration rate 0.1 every five generations. Five topologies: no migration, ring, star, random (Erdos-Renyi resampled each migration), and fully connected. Thirty seeds per topology-domain combination: nine hundred runs total. Diversity is normalized pairwise Hamming distance for binary genomes, Euclidean for real-valued.

---

## Slide 10

Here are the five topologies. The key quantity is algebraic connectivity -- lambda-two, the second eigenvalue of the graph Laplacian. Smaller lambda-two means slower mixing, which means higher sustained diversity. Lambda-two increases as we add connections: none has zero, ring has 1.38, fully connected has five. More connected, faster mixing, lower diversity.

---

## Slide 11

Let me walk you through this result domain by domain. OneMax -- the simplest possible landscape. The ordering is none, ring, star, random, fully connected. Mazes -- completely different genome, different fitness. Same ordering. Graph coloring. Same ordering. Knapsack. Same ordering. No Thanks -- and this is the one I want you to focus on. There is no fixed fitness landscape here. Fitness is entirely relative, determined by co-evolution within each island. If the ordering were an artifact of landscape geometry, it should break here. It doesn't. Same ordering. And checkers, with intransitive fitness. Same ordering.

Six domains. Perfect concordance. Kendall's W is one-point-zero -- every domain producing the identical rank ordering. The two-way ANOVA: F for topology is 47.8. F for domain is 0.13 -- p equals 0.945, not significant. Domain explains essentially zero variance. Topology explains nearly all of it. Twenty-four times more variance from topology than from domain.

---

## Slide 12

Why this ordering? The ordering follows algebraic connectivity -- lambda-two of the migration graph. Smaller lambda-two, slower mixing, higher diversity. The relationship is monotone. But notice: the none-to-ring transition is massive -- a 35 percent diversity drop. That's the symmetry-breaking first coupling. You go from isolated populations to the weakest possible connection and lose a third of your diversity. Every subsequent step -- ring to star, star to random, random to fully connected -- contributes at most 9 percent. The first coupling dominates.

---

## Slide 13

Here's where the theory goes from descriptive to predictive. At five islands, lambda-two for the ring is 1.382 -- greater than the star's 1.0. So ring and star should be hard to distinguish. They are: Fisher's combined p-value across all six domains is 0.14, not significant. But at seven islands, the inequality reverses. Lambda-two for the seven-node ring drops to 0.753, below the star's 1.0. The theory predicts ring should now preserve more diversity than star. We ran the confirmatory experiment on mazes with thirty seeds: ring 0.387, star 0.336, p equals 6.6 times ten to the minus five. A novel prediction, confirmed. That's not post-hoc explanation. That's science.

---

## Slide 14

A seventh domain -- sorting networks -- violates the ordering. We include this deliberately. A theory that can't be falsified isn't useful. The scope condition is clear: the universal ordering holds where the fitness landscape admits a sufficient selective gradient. Where domain-specific constraints dominate migration dynamics, it breaks. Knowing the boundary is as valuable as knowing the result.

---

## Slide 15

We move from topology to strategy-level composition. Same base operators -- evaluate, select, crossover, mutate -- composed four different ways. Generational is pure iteration. Hourglass has three explicit phases: explore, converge, diversify. Island runs subpopulations with ring migration. Adaptive starts exploring, then switches to focused search when it detects a plateau. Same ingredients, different recipes. Does the recipe determine the dish?

---

## Slide 16

Yes. Emphatically yes. Look at these four trajectories.

Flat generational -- the blue curve -- declines monotonically. Diversity drops from 0.50 to 0.13 over fifty generations. No structural intervention, so selection pressure wins.

Hourglass -- the green curve -- watch the shape. High diversity in the explore phase, then a crash during convergence -- an 85 percent reduction to 0.06 -- then a rebound back to 0.37 in the diversify phase. The three-phase composition structure is directly visible in the trajectory. You can see the phase boundaries.

Island -- the orange curve -- maintains diversity above the flat strategy throughout. Migration every five generations injects just enough genetic material to counterbalance convergence. A diversity thermostat.

And adaptive -- the red curve -- starts like hourglass, then collapses to 0.02 when the plateau detector fires. Irreversible convergence. No recovery.

Now look across all three panels. Maze, graph coloring, knapsack -- completely different domains. Every fingerprint is the same shape. Same operators, 18x spread in final diversity. Adaptive ends at 0.02, hourglass at 0.37. The composition determines the trajectory. Not the operators. Not the landscape.

---

## Slide 17

The hourglass crash happens at the same relative point in every domain. The island thermostat operates at the same level. The adaptive collapse is equally irreversible everywhere. These are structural consequences of composition, not coincidences. You can predict the diversity trajectory of a new system from its composition structure before running a single experiment.

---

## Slide 18

Three practical takeaways.

First: prefer ring or mesh over star. A star has beta-one equals zero -- no independent cycles. Replace the hub with a ring, you add n minus one cycles at zero additional edge cost. This avoids the premature convergence singularity.

Second: monitor lambda-two, not diversity directly. It's one eigenvalue of the Laplacian, cheap to compute, and predicts diversity collapse fifty to a hundred generations early.

Third: if lambda-two drops too low, add one cycle-closing edge. Don't just increase migration rate -- that raises coupling without changing topology. A cycle-closing edge changes both.

---

## Slide 19

To summarize. Composition determines diversity -- empirically, across six unrelated domains. The topology ordering is universal with perfect concordance, and spectral graph theory explains why and makes confirmed novel predictions. The same operators produce an 18x spread in diversity depending on composition. The categorical framework makes all of this formally visible. Future work scales to larger populations and heterogeneous island strategies.

---

## Slide 20

Thank you. The paper and all experimental code are available at the links on screen. I'm happy to take questions -- you can reach me at the email shown, or find me at the poster session. Thank you.
