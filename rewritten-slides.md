# Slide 0: Diversity

Before a single result will mean anything, we need to pin down one word: *diversity*. In a genetic algorithm it isn't a vibe — it's a measurement. Take every pair of individuals in your population, compute the Hamming distance between them, average over all the pairs, and normalize to the unit interval. Zero means you're looking at a room full of clones. One means no two individuals agree on anything.

So why hang an entire talk on that one number? Because diversity is the tension that evolutionary search actually runs on. Spend it too fast and the whole population avalanches onto the first half-decent solution it stumbles into — premature convergence, a local optimum with no exit. Hoard too much of it and selection has nothing to bite down on; you've built an expensive random number generator. Anyone who has babysat a GA has felt both failure modes. The live question isn't whether the trade-off exists — it's *what turns the dial*. The reflex answers are mutation rate, selection pressure, population size. Our answer was none of them. It's the topology — how the islands are wired to each other — and the gap isn't close.

# Slide 1: Multi-Domain Topology Ordering

If you walk out remembering one image, let it be this one. Topology determines diversity. We ran six domains with nothing in common — counting bits, escaping mazes, NP-hard constraint satisfaction, a strategic card game, co-evolved checkers players — and the very same ordering came back every time. Isolated islands hold onto the most diversity; fully connected islands hold onto the least. Read the bars left to right and they march downward in one fixed order: None, Ring, Star, Random, Fully Connected. Kendall's W is exactly 1.0 — total agreement, not one domain stepping out of line. The single setting most people choose on the first afternoon and never look at again ends up mattering more than the problem they spent months picking.

# Slide 2: The Domains

1. **OneMax** — count the 1s in a bit string and push that count up. One smooth hill, no traps. It's the honest baseline: if the effect shows up even here, it isn't a side-effect of difficulty.

2. **Maze** — not *solving* a maze, *breeding* one. The genome is a perfect maze: a spanning tree over a grid graph, so every cell reaches every other by exactly one path, with no loops and no walled-off regions. Mutation is an edge-swap — drop one edge from the tree, splice in a non-tree edge that stitches the two halves back into a single tree. Crossover is union crossover — pool both parents' edges and carve a fresh spanning tree out of that pool with Kruskal's. Fitness rewards difficulty: long winding solution paths, high tortuosity, a healthy ratio of dead ends. Deceptive by design — two mazes can sit close in Hamming distance and still play nothing alike.

3. **Graph Coloring** — color the vertices so no edge connects two of the same color. NP-hard, with a rugged landscape full of plateaus where progress just stops.

4. **Knapsack** — pack items to maximize value under a weight budget. Also NP-hard, but the terrain is gentler than coloring's.

5. **No Thanks!** — an actual card game with actual strategy. Co-evolutionary: your fitness is scored against the other strategies in the population, so the ground shifts under you as everyone gets better.

6. **Checkers** — two players, evolved board-evaluation functions, real-valued genomes. The heaviest domain in the set, and co-evolutionary like No Thanks!

The variety here is the whole point. These six were chosen to stretch across three independent axes: **difficulty** (trivial to NP-hard), **landscape** (fixed to co-evolutionary), and **representation** (binary to real-valued). A result that holds across all three can't be about any one of them.

# Slide 3: Island Models & Topology

One definition, because everything downstream leans on it. In an island-model GA you don't run a single big population — you carve it into sub-populations called islands, let each one evolve on its own, and every so often migrate a few individuals between them. The *topology* is simply the graph that decides which islands are allowed to ship migrants to which. Far left: no edges at all — the islands are sealed, nothing ever crosses between them. Then a ring, where each island trades only with its two neighbors. Then a star, one hub wired to all the rest. Then random — an Erdős–Rényi graph carrying roughly half the possible edges. And finally fully connected, where everyone migrates to everyone.

The number under each graph is λ₂, the algebraic connectivity — the second-smallest eigenvalue of the graph Laplacian. It's the one quantity that captures how fast information diffuses across the network. λ₂ = 0 means the graph is in disconnected pieces and a good solution on one island never reaches the others. λ₂ = 5 means mixing is essentially instantaneous — every island sees everyone else's best individuals almost at once. And that — instant mixing — is precisely what flattens diversity. The arrow along the bottom is the entire mechanism in one line: more connected, faster mixing, less diversity.

# Slide 4: Multi-Domain Topology Ordering (reprise)

Same chart, second pass — but now you're armed. You know the six domains share no representation, no landscape, no difficulty class. You know what λ₂ measures and why connectivity fixes the mixing speed. So look again. Every domain, the identical staircase: None, Ring, Star, Random, Fully Connected, descending left to right with not one exception anywhere on the figure. That is what Kendall's W = 1.0 actually means — six independent witnesses agreeing on the ranking right down to the last rung. And this isn't a fragile effect you need statistics to tease out: topology accounts for **23.9 times** more of the variance in final diversity than the choice of domain does. Which problem you're solving barely shows up. How you wire the islands decides whether diversity lives through the run.

# Slide 5: Statistics

Let me take the numbers slowly, because they came back unusually clean and each one answers a specific question.

**Kendall's W** first. It's a concordance test, and its question is blunt: do the six domains agree on how to rank the five topologies? W runs from 0 to 1. Zero is total disagreement — six domains, six different orderings. One is unanimity — every domain hands back the same ranking. We got W = 1.0. Not 0.97, not 0.99 — 1.0. All six rank the topologies identically: None preserves the most diversity, then Ring, then Star, then Random, then Fully Connected dead last. The chi-squared is 24.0 at p = 0.00008 — meaning that if topology had no real effect, agreement this complete would surface by chance fewer than once in ten thousand runs.

As a cross-check we computed all **fifteen** pairwise Spearman correlations — every domain against every other, head to head. All fifteen came back ρ = 1.0. OneMax tracks Checkers. Maze tracks Knapsack. No Thanks! tracks Graph Coloring. Fifteen out of fifteen, with no soft agreement hiding inside the average.

Then the **two-way ANOVA**, which splits the variance in final diversity into its two sources. Topology: F = 47.8, p < 10⁻⁶ — overwhelming. Domain: F = 0.13, p = 0.945 — indistinguishable from noise. Domain explains essentially nothing. Set them side by side and topology carries 23.9 times the variance. In plain terms: if you want to predict how diverse a population will end up, the topology tells you almost everything, and knowing whether the task was OneMax or Checkers tells you almost nothing.

# Slide 6: The End

We've covered the main result — so if that's what you came for, this is a fine place to tune out. Everything from here on is optional: it's the story of how we actually found it.

# Slide 7: Kleisli Arrows

So how did this actually come to light? It began with a thoroughly unremarkable GA in Rust — a maze generator called `mazegen-rs` — about seventy-five lines of imperative code inside a `for` loop. Nothing wrong with it; it ran fine. The turn came when we ported it to Haskell. Structure that had been sitting there the whole time suddenly became *visible*.

Take selection. In Rust it hands you back an index — an integer — and the link to crossover is implicit: you use that index to pull an individual out of an array. In Haskell selection is a morphism. Population in, population out, same type on both ends. And every operator wears that same signature: `Pop a → M (Pop a)`. A population goes in, a population comes out, and every side effect — randomness, configuration, logging — lives inside that `M`.

That monad is where the leverage hides. In Rust you thread a mutable RNG reference through every call by hand, pass the config beside it, and bolt on logging separately. In Haskell `EvoM` folds all three into one stack: `ReaderT` for config, `WriterT` for logging, `State` for the generator. Want a new effect? You change the stack in one place, not in every signature across the program.

And because the operators all share a type, they compose — Kleisli composition, the fish operator `>=>`. Evaluate, then select, then crossover, then mutate. Six lines where Rust had seventy-five. But the line count isn't the real prize. The prize is that the pipeline is now a *value* — not statements trapped inside a loop, but a thing you can name, hand around, lift through an island functor, swap out for another strategy. The composition is out in the open, and the type checker is standing over it.

That's exactly what made the experiment cheap to run. Once the island model was a functor parameterized by its migration graph, changing topology was a one-line edit — swap the graph, hold everything else fixed. Changing domain was just as small — swap the genome type and the fitness function, leave the pipeline shape alone. In the Rust version either change meant rewriting the loop. In Haskell both are simply parameters. That's how 900 experiments across six domains and five topologies ran on a single body of code. The categorical structure was in the Rust all along. Haskell only made it impossible to miss.

# Slide 8: Why?

Everything up to here is empirical. We've shown *that* topology determines diversity; we have not shown *why*. Why should λ₂ predict the ordering so exactly? Why should the result refuse to care which domain you ran? A categorical account of this is open work — the goal is to derive the topology ordering from the structure of the island functor itself, not merely to watch it fall out of the data. If you have an idea for that proof, come find me afterward; I genuinely want to hear it.

And this is the finding I'd defend hardest. That topology *matters* surprises no one — most people would guess as much. What's surprising is the size of it. Topology doesn't nudge past domain; it dominates by a factor of twenty-four. The decision most practitioners make once, by reflex, and never revisit turns out to be the most consequential structural choice in the whole algorithm.

One last thing before you go: none of this is locked away. Every bit of code you'd need to repeat the experiments is public, and you can find it all at https://github.com/lyra-claude/categorical-evolution/blob/main/gecco2026/supplementary-materials-README.md — start from that README and it points you to everything else.
