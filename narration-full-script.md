# GECCO 2026 Oral Presentation -- Narration Script (Trimmed)

**Paper:** "Composition Determines Diversity: Categorical Fingerprints of Genetic Algorithms"
**Authors:** Robin Langer, Claudius Turing, Lyra Vega
**Venue:** GECCO 2026, San Jose, Costa Rica, July 13-17
**Target length:** ~15:30 at ~100 wpm effective delivery (~1,550 words)
**Note:** Original file header claimed ~2,850 words / 19 min at 150 wpm; actual narration-only count was 1,919 words. At 100 wpm effective delivery (pauses, slide transitions) that gives ~19.2 min -- consistent. Trimmed to ~1,550 words / ~15:30 at 100 wpm.

---

## Slide 1: Title + Provocation
**Duration:** 30 seconds (~50 words)

**Visual:** Paper title centered. Authors and affiliations below. GECCO 2026 logo in the corner. Clean, minimal.

**Narration:**
Good morning. I'm Robin Langer, and this is joint work with Claudius Turing and Lyra Vega. Our paper is called "Composition Determines Diversity," and the core claim is: how you wire your islands matters more than what you evolve. That's a strong claim. Let me show you the evidence.

---

## Slide 2: The Surprising Claim [AHA MOMENT]
**Duration:** 45 seconds (~80 words)

**Visual:** The multi_domain_topology_ordering.pdf bar chart -- the money figure. Bars grouped by domain (OneMax, Maze, Graph Coloring, Knapsack, No Thanks!, Checkers), colored by topology. The ordering none > ring > star > random > FC is visually identical in every group. Annotate: W = 1.0, p = 0.00008, 23.9x.

**Narration:**
Here is the result. Six completely unrelated domains -- from trivial bit-counting to co-evolutionary card games. Five migration topologies. Nine hundred experimental runs. And in every single domain, the diversity ordering is identical: no migration, then ring, then star, then random, then fully connected. Kendall's W equals one-point-zero -- perfect concordance. The p-value is 0.00008. Topology explains 23.9 times more variance in diversity than which domain you're solving. Domain is noise. Topology is signal. Now let me explain why.

---

## Slide 3: Why Should You Care?
**Duration:** 30 seconds (~55 words)

**Visual:** Two-column contrast. Left: "What practitioners tune" -- operator selection, crossover type, mutation rate, population size. Right (highlighted): "What actually determines diversity" -- composition structure, migration topology. Three bullet-point previews of practical recommendations at the bottom.

**Narration:**
If you're a GA practitioner, you've spent weeks tuning operators -- crossover type, mutation rate, population size. Our result says that's optimizing the wrong thing. The composition structure -- how you wire your operators together, how your islands communicate -- dominates everything else. But first, we need to understand what "composition" means formally.

---

## Slide 4: The Problem -- Hidden Composition
**Duration:** 45 seconds (~75 words)

**Visual:** The Rust code snippet from the paper (the imperative loop, ~15 lines shown). Red arrows overlaid showing the implicit pipeline: tournament_select -> crossover -> mutate -> evaluate. Labels: "index, not individual," "manual sequencing," "explicit effect threading."

**Narration:**
Every genetic algorithm composes operators. Select, crossover, mutate, replace -- that's a pipeline. But look at how it appears in typical imperative code. This is a real Rust implementation from our maze evolution system. The pipeline is buried. Selection returns an index, not an individual. The sequence exists only as ordered statements. Randomness, configuration, and logging are threaded as separate arguments everywhere. The composition has no formal representation. It's invisible.

---

## Slide 5: Making Composition Explicit [AHA MOMENT]
**Duration:** 60 seconds (~100 words)

**Visual:** Side by side. Left: Rust loop (~15 lines, grayed slightly). Right: Haskell pipeline (6 lines, highlighted, large font). The Haskell code:
```
generationStep fitFunc mutFunc gen =
  evaluate fitFunc
    >>>: logGeneration gen
    >>>: elitistSelect
    >>>: onePointCrossover
    >>>: pointMutate mutFunc
```
Big annotation: "6 lines. Same algorithm."

**Narration:**
Same algorithm. Six lines. Each line is a morphism -- a typed, composable operation. The triple-arrow operator is Kleisli composition: it chains these operations while automatically threading randomness, configuration, and logging through a monad. The important thing is not that it's shorter. It's that the composition is now a first-class value. You can see it. You can type-check it. You can compose it further -- lift it into an island model, nest it inside an adaptive strategy. The structure was hidden. Now it's visible. And once it's visible, you can reason about it.

---

## Slide 6: Three Composition Levels
**Duration:** 45 seconds (~85 words)

**Visual:** Three-tier tower diagram. Bottom tier: "Operators -> Pipelines" (select, crossover, mutate composing into one generation step). Middle tier: "Pipelines -> Strategies" (generational, hourglass, island, adaptive). Top tier: "Strategies -> Multi-population systems." Arrows between tiers. Key annotation: "Same type at every level: Pop a -> M (Pop a)."

**Narration:**
The framework has three composition levels. Level 1: operators compose into a pipeline -- one generation step. Level 2: pipelines compose into strategies -- generational, island, hourglass, adaptive. Level 3: strategies compose into multi-population architectures. What makes this a design algebra rather than notation: the composite at every level has the same type as its parts. A pipeline and a strategy are both arrows from populations to populations. You can always compose further.

---

## Slide 7: Domain Invariance
**Duration:** 40 seconds (~75 words)

**Visual:** Table 2 from the paper (the invariance table). Left column: what changes (genome type, fitness function, mutation operator). Right column: what stays invariant (pipeline shape, monad stack, composition operator, strategy combinators). Below: the two pipelines side by side -- checkers ([Double], winRate, gaussianPerturb) vs. mazes ([Bool], mazeFitness, bitFlip) -- with identical structure highlighted.

**Narration:**
Category theory predicts structure lives in the arrows, not the objects. And that's exactly what we see. Checkers uses real-valued weight vectors and win-rate fitness. Mazes use binary vectors and path-length fitness. Incompatible types, incommensurable fitness. But the pipeline shape, the monad stack, the composition operator -- all identical. Which raises the natural question: if composition structure is domain-invariant, are its effects on diversity also domain-invariant?

---

## Slide 8: Six Domains
**Duration:** 35 seconds (~65 words)

**Visual:** 2x3 grid. Each cell: domain name, small icon or diagram, key properties. OneMax (trivial, unimodal), Maze (binary, multi-objective), Graph Coloring (constraint satisfaction), Knapsack (epistatic), No Thanks! (co-evolutionary, no fixed landscape), Checkers (intransitive). Color-coded by representation type.

**Narration:**
To test this, we chose six domains that share nothing in content. OneMax is trivially unimodal. Maze generation is multi-objective. Graph coloring is constraint satisfaction. Knapsack has epistatic interactions. No Thanks is co-evolutionary -- no fixed fitness landscape. Checkers has intransitive fitness: A beats B, B beats C, C beats A. If composition determines diversity across all six, it cannot be a landscape artifact.

---

## Slide 9: Experimental Setup
**Duration:** 30 seconds (~55 words)

**Visual:** Setup summary: 5 islands x 16 individuals, 100 generations, migration rate 0.1 every 5 generations, 30 seeds per (topology, domain) pair = 900 total runs. Diversity metric: normalized pairwise Hamming/Euclidean distance.

**Narration:**
Five islands, sixteen individuals each, one hundred generations. Migration rate 0.1 every five generations. Five topologies: no migration, ring, star, random (Erdos-Renyi resampled each migration), and fully connected. Thirty seeds per topology-domain combination: nine hundred runs total. Diversity is normalized pairwise Hamming distance for binary genomes, Euclidean for real-valued.

---

## Slide 10: Five Topologies Visualized
**Duration:** 30 seconds (~60 words)

**Visual:** Five small network diagrams in a row: none (5 disconnected nodes), ring (cycle), star (hub-and-spoke), random (irregular edges), fully connected (complete graph). Below each: lambda_2 value. Animation suggestion: edges appearing progressively from none to FC.

**Narration:**
Here are the five topologies. The key quantity is algebraic connectivity -- lambda-two, the second eigenvalue of the graph Laplacian. Smaller lambda-two means slower mixing, which means higher sustained diversity. Lambda-two increases as we add connections: none has zero, ring has 1.38, fully connected has five. More connected, faster mixing, lower diversity.

---

## Slide 11: The Universal Ordering [AHA MOMENT]
**Duration:** 90 seconds (~165 words)

**Visual:** The multi_domain_topology_ordering.pdf chart again, now with full statistical annotations. Animate bars appearing one domain at a time: OneMax first, then Maze, then Graph Coloring, Knapsack, No Thanks!, Checkers. Each time the same ordering appears. Final annotation: Kendall's W = 1.0, F_topo = 47.8, F_domain = 0.13 (p = 0.945).

**Narration:**
Let me walk you through this result domain by domain. OneMax -- the simplest possible landscape. The ordering is none, ring, star, random, fully connected. Mazes -- completely different genome, different fitness. Same ordering. Graph coloring. Same ordering. Knapsack. Same ordering. No Thanks -- and this is the one I want you to focus on. There is no fixed fitness landscape here. Fitness is entirely relative, determined by co-evolution within each island. If the ordering were an artifact of landscape geometry, it should break here. It doesn't. Same ordering. And checkers, with intransitive fitness. Same ordering.

[Pause.]

Six domains. Perfect concordance. Kendall's W is one-point-zero -- every domain producing the identical rank ordering. The two-way ANOVA: F for topology is 47.8. F for domain is 0.13 -- p equals 0.945, not significant. Domain explains essentially zero variance. Topology explains nearly all of it. Twenty-four times more variance from topology than from domain.

---

## Slide 12: The Spectral Explanation
**Duration:** 45 seconds (~85 words)

**Visual:** Plot of lambda_2 (x-axis) vs. final diversity (y-axis) showing the monotone relationship. Annotate the none-to-ring transition: 35% diversity drop. Subsequent steps labeled: at most 9% each.

**Narration:**
Why this ordering? The ordering follows algebraic connectivity -- lambda-two of the migration graph. Smaller lambda-two, slower mixing, higher diversity. The relationship is monotone. But notice: the none-to-ring transition is massive -- a 35 percent diversity drop. That's the symmetry-breaking first coupling. You go from isolated populations to the weakest possible connection and lose a third of your diversity. Every subsequent step -- ring to star, star to random, random to fully connected -- contributes at most 9 percent. The first coupling dominates.

---

## Slide 13: Falsifiable Prediction -- Ring/Star Inversion [AHA MOMENT]
**Duration:** 60 seconds (~125 words)

**Visual:** Two network diagrams side by side. Left: 5-node ring and 5-node star with lambda_2 values (ring: 1.382, star: 1.0). Right: 7-node ring and 7-node star with lambda_2 values (ring: 0.753, star: 1.0). Arrow showing the inversion. Below: confirmatory result -- ring 0.387 +/- 0.028 vs. star 0.336 +/- 0.053, p = 6.6e-5.

**Narration:**
Here's where the theory goes from descriptive to predictive. At five islands, lambda-two for the ring is 1.382 -- greater than the star's 1.0. So ring and star should be hard to distinguish. They are: Fisher's combined p-value across all six domains is 0.14, not significant. But at seven islands, the inequality reverses. Lambda-two for the seven-node ring drops to 0.753, below the star's 1.0. The theory predicts ring should now preserve more diversity than star. We ran the confirmatory experiment on mazes with thirty seeds: ring 0.387, star 0.336, p equals 6.6 times ten to the minus five. A novel prediction, confirmed. That's not post-hoc explanation. That's science.

---

## Slide 14: Sorting Networks -- The Scope Condition
**Duration:** 30 seconds (~60 words)

**Visual:** Brief slide. Icon of a sorting network. Text: "7th domain violates the ordering." Below: "Scope: holds where fitness landscape admits sufficient selective gradient." A checkmark for honesty/falsifiability.

**Narration:**
A seventh domain -- sorting networks -- violates the ordering. We include this deliberately. A theory that can't be falsified isn't useful. The scope condition is clear: the universal ordering holds where the fitness landscape admits a sufficient selective gradient. Where domain-specific constraints dominate migration dynamics, it breaks. Knowing the boundary is as valuable as knowing the result.

---

## Slide 15: Four Strategy Compositions
**Duration:** 35 seconds (~65 words)

**Visual:** Four small diagrams showing composition structure. Generational: simple loop arrow. Hourglass: three connected phases (explore -> converge -> diversify). Island: four parallel pipelines with migration arrows. Adaptive: two phases with a conditional switch (plateau detector). All using the same base operators, different wiring.

**Narration:**
We move from topology to strategy-level composition. Same base operators -- evaluate, select, crossover, mutate -- composed four different ways. Generational is pure iteration. Hourglass has three explicit phases: explore, converge, diversify. Island runs subpopulations with ring migration. Adaptive starts exploring, then switches to focused search when it detects a plateau. Same ingredients, different recipes. Does the recipe determine the dish?

---

## Slide 16: The Fingerprint Chart [AHA MOMENT]
**Duration:** 90 seconds (~190 words)

**Visual:** The three-panel fingerprint figure from the paper (Figure 2). Panels: (a) Maze, (b) Graph Coloring, (c) Knapsack. Four curves per panel: flat/blue (monotonic decline), hourglass/green (spike-crash-rebound), island/orange (stable maintenance), adaptive/red (spike-then-collapse). Animate one strategy at a time across all three panels.

**Narration:**
Yes. Emphatically yes. Look at these four trajectories.

Flat generational -- the blue curve -- declines monotonically. Diversity drops from 0.50 to 0.13 over fifty generations. No structural intervention, so selection pressure wins.

Hourglass -- the green curve -- watch the shape. High diversity in the explore phase, then a crash during convergence -- an 85 percent reduction to 0.06 -- then a rebound back to 0.37 in the diversify phase. The three-phase composition structure is directly visible in the trajectory. You can see the phase boundaries.

Island -- the orange curve -- maintains diversity above the flat strategy throughout. Migration every five generations injects just enough genetic material to counterbalance convergence. A diversity thermostat.

And adaptive -- the red curve -- starts like hourglass, then collapses to 0.02 when the plateau detector fires. Irreversible convergence. No recovery.

Now look across all three panels. Maze, graph coloring, knapsack -- completely different domains. Every fingerprint is the same shape. Same operators, 18x spread in final diversity. Adaptive ends at 0.02, hourglass at 0.37. The composition determines the trajectory. Not the operators. Not the landscape.

---

## Slide 17: Fingerprints Are Stable Across Domains
**Duration:** 30 seconds (~60 words)

**Visual:** Overlay or aligned comparison of the three domain panels, highlighting pattern consistency. Perhaps a correlation matrix showing cross-domain trajectory similarity for each strategy. Key annotation: "Shape determined by composition, not landscape."

**Narration:**
The hourglass crash happens at the same relative point in every domain. The island thermostat operates at the same level. The adaptive collapse is equally irreversible everywhere. These are structural consequences of composition, not coincidences. You can predict the diversity trajectory of a new system from its composition structure before running a single experiment.

---

## Slide 18: Three Recommendations for Practitioners
**Duration:** 45 seconds (~100 words)

**Visual:** Three numbered recommendations with supporting diagrams. 1: "Prefer ring/mesh over star" with before/after network diagram (star -> ring, one edge changes adding cycles, beta_1 >= 1). 2: "Monitor lambda_2, not diversity" with a cheap-to-compute indicator. 3: "Add cycle-closing edges, not migration rate" with diagram showing one edge addition.

**Narration:**
Three practical takeaways.

First: prefer ring or mesh over star. A star has beta-one equals zero -- no independent cycles. Replace the hub with a ring, you add n minus one cycles at zero additional edge cost. This avoids the premature convergence singularity.

Second: monitor lambda-two, not diversity directly. It's one eigenvalue of the Laplacian, cheap to compute, and predicts diversity collapse fifty to a hundred generations early.

Third: if lambda-two drops too low, add one cycle-closing edge. Don't just increase migration rate -- that raises coupling without changing topology. A cycle-closing edge changes both.

---

## Slide 19: Summary
**Duration:** 30 seconds (~60 words)

**Visual:** Key numbers displayed prominently: W = 1.0, 23.9x, 18x, 6 domains, p = 0.00008, p = 6.6e-5. Below: bullet summary. "Composition determines diversity." "Universal topology ordering explained by spectral graph theory." "Diversity fingerprints are properties of composition patterns." "Future: larger populations, longer runs, heterogeneous island strategies."

**Narration:**
To summarize. Composition determines diversity -- empirically, across six unrelated domains. The topology ordering is universal with perfect concordance, and spectral graph theory explains why and makes confirmed novel predictions. The same operators produce an 18x spread in diversity depending on composition. The categorical framework makes all of this formally visible. Future work scales to larger populations and heterogeneous island strategies.

---

## Slide 20: Thank You
**Duration:** 15 seconds (~35 words)

**Visual:** "Thank you." Paper title and DOI. Code repository URL (GitHub). Author contact emails. QR code to paper/repo if applicable. GECCO 2026 logo.

**Narration:**
Thank you. The paper and all experimental code are available at the links on screen. I'm happy to take questions -- you can reach me at the email shown, or find me at the poster session. Thank you.

---

## Script Statistics

- **Total word count:** ~1,555 words (narration only)
- **Estimated speaking time:** ~15:33 at 100 wpm effective delivery (original was ~19:12 at 100 wpm for 1,919 words)
- **Slide count:** 20
- **AHA moments:** 5 (Slides 2, 5, 11, 13, 16) -- all preserved at close to full strength

## Production Notes

### Note on word count vs. timing:
The original script header claimed ~2,850 words / ~19 min at 150 wpm, but the actual narration-only word count is 1,919 words. At 150 wpm that is ~12:48. The 19-minute figure is consistent with ~100 wpm effective delivery rate (pauses, slide transitions, natural speaking pace). This trimmed version targets ~1,550 words, which gives ~15:30 at 100 wpm.

### Sections that may need attention:

1. **Slide 10 (Five Topologies Visualized)** -- The lambda_2 values cited are: none = 0, ring = 1.382, FC = 5. The paper gives ring = 1.382 and star = 1.0 explicitly at n=5, but doesn't list all five lambda_2 values in a single table. The random topology lambda_2 varies per resampling. Confirm exact values or note that random is "expected lambda_2" for the video.

2. **Slide 12 (Spectral Explanation)** -- The paper states "none-to-ring transition produces a 35% diversity drop" and "each subsequent step contributes at most 9%." A lambda_2-vs-diversity plot may need to be generated from data.

3. **Slide 14 (Sorting Networks)** -- The paper mentions this briefly as a scope condition. Robin may want to add a specific data point.

4. **Slide 17 (Stability)** -- A cross-domain correlation matrix is described in the visual but may need to be computed from data.

5. **Timing** -- Slides 11 and 16 are the longest at 90 seconds each. These are the two central results. Slides 10 and 14 are the shortest.
