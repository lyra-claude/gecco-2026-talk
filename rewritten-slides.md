# Slide 0: Diversity

Before any of the results make sense, we have to agree on one word: *diversity*. In a genetic algorithm it has a precise meaning — it's how spread out your population is in genotype space. Concretely, take every pair of individuals, measure the Hamming distance between them, average those, and normalize to the unit interval. Zero is a population of clones. One is a population that agrees on nothing.

Here's why that single number is worth a whole talk. Diversity is the load-bearing tension in evolutionary search. Burn through it too quickly and the whole population piles onto the first decent solution it finds — premature convergence, a local optimum you can't climb out of. Keep too much of it and selection has nothing to grip; you're paying for evolution and getting random search. Everyone who has run a GA has felt this trade-off. The interesting question isn't *whether* it exists — it's **what holds the dial**. The usual suspects are mutation rate, selection pressure, population size. Our answer turned out to be none of those. It's the topology — the wiring between your islands — and it isn't close.

# Slide 1: Multi-Domain Topology Ordering

If you remember one picture from this talk, make it this one. Topology determines diversity. We ran six domains that have nothing in common — counting bits, solving mazes, NP-hard constraint satisfaction, a card game, evolved checkers players — and the *same* ordering came back every single time. Most diversity preserved by isolated islands; least by fully connected ones. Read the bars left to right and they always descend in the same order: None, Ring, Star, Random, Fully Connected. Kendall's W is exactly 1.0 — flawless agreement, not one domain out of step. The one knob most people set on day one and never touch again turns out to matter more than the problem they spent months choosing.

# Slide 2: The Domains

1. **OneMax** — count the 1s in a bit string and maximize them. A single smooth hill, no traps. It's here as the honest baseline: if the effect shows up even on OneMax, it isn't an artifact of difficulty.

2. **Maze** — find a path through a grid from start to goal, with the genome encoding a sequence of moves. Deceptive on purpose: being close to the answer in Hamming distance doesn't mean you're close to a good route.

3. **Graph Coloring** — color the vertices so no edge joins two of the same color. NP-hard, with a rugged landscape full of plateaus where progress stalls.

4. **Knapsack** — choose items to maximize value under a weight budget. Also NP-hard, but the landscape is gentler than coloring's.

5. **No Thanks!** — a real card game with genuine strategy. Co-evolutionary: your fitness is judged against the other strategies in the population, so the landscape moves under your feet as everyone improves.

6. **Checkers** — two players, evolved board-evaluation functions, real-valued genomes. The heaviest domain in the set, and co-evolutionary like No Thanks!

The point of this list isn't variety for its own sake. The six were chosen to span three independent axes: **difficulty** (trivial to NP-hard), **landscape** (fixed to co-evolutionary), and **representation** (binary to real-valued). If a result survives all three, it isn't about any one of them.

# Slide 3: Island Models & Topology

A quick definition, because everything hinges on it. In an island-model GA you don't run one big population — you split it into sub-populations called islands, let each evolve on its own, and every so often migrate a few individuals between them. The *topology* is just the graph that says which islands are allowed to send migrants to which. Far left: no edges at all — the islands are sealed off and never exchange anything. Then a ring, where each island only trades with its two neighbors. Then a star, one hub wired to everyone else. Then random — an Erdős–Rényi graph with roughly half the possible edges present. And finally fully connected, where everyone migrates to everyone.

The number printed under each graph is λ₂, the algebraic connectivity — the second-smallest eigenvalue of the graph Laplacian. It's the single quantity that says how fast information diffuses across the network. λ₂ = 0 means the graph is in disconnected pieces and good solutions never spread. λ₂ = 5 means mixing is essentially instantaneous: every island sees everyone else's best individuals almost immediately. And *that* — instant mixing — is what flattens diversity. The arrow along the bottom is the whole mechanism in one line: more connected, faster mixing, less diversity.

# Slide 4: Multi-Domain Topology Ordering (reprise)

Same chart, second look — but now you're armed. You know the six domains share no representation, no landscape, no difficulty class. You know what λ₂ measures and why connectivity sets the mixing speed. So look again. Every domain, the identical staircase: None, Ring, Star, Random, Fully Connected, descending left to right with no exception anywhere on the figure. That's the meaning of Kendall's W = 1.0 — six independent witnesses agreeing on the ranking down to the last position. And this is not a delicate effect you need statistics to coax out: topology accounts for **23.9 times** more of the variance in final diversity than the choice of domain does. Which problem you're solving barely registers. How you wire the islands decides whether diversity survives the run.

# Slide 5: Statistics

I want to walk the numbers slowly, because they came back unusually clean and each one answers a specific question.

**Kendall's W** first. It's a concordance test, and the question it asks is plain: do the six domains agree on how to rank the five topologies? W runs from 0 to 1. Zero is total disagreement — six domains, six different orderings. One is unanimity — every domain produces the same ranking. We got W = 1.0. Not 0.97, not 0.99 — 1.0. All six rank the topologies identically: None preserves the most diversity, then Ring, Star, Random, and Fully Connected dead last. The chi-squared is 24.0 with p = 0.00008 — meaning that if topology had no real effect, agreement this complete would turn up by chance less than once in ten thousand runs.

As a cross-check we computed all **fifteen** pairwise Spearman correlations — every domain against every other, head to head. All fifteen came back ρ = 1.0. OneMax tracks Checkers. Maze tracks Knapsack. No Thanks! tracks Graph Coloring. Fifteen for fifteen, no partial agreements hiding in the average.

Then the **two-way ANOVA**, which splits the variance in final diversity into its two sources. Topology: F = 47.8, p < 10⁻⁶ — overwhelming. Domain: F = 0.13, p = 0.945 — indistinguishable from noise. Domain explains essentially nothing. Put the two side by side and topology carries 23.9× the variance. In plain terms: if you want to predict how diverse a population will end up, the topology tells you almost everything, and knowing whether the task was OneMax or Checkers tells you almost nothing.

# Slide 6: The End

*[Transition slide — no notes. A good place to lose anyone who only came for the result. What follows is how we found it.]*

# Slide 7: Kleisli Arrows

So how did this actually surface? It started with a perfectly ordinary GA in Rust — a maze generator, `mazegen-rs` — about seventy-five lines of imperative code inside a `for` loop. Nothing wrong with it; it ran. The shift came when we ported it to Haskell. The structure that had been there all along suddenly became *visible*.

Look at selection. In Rust it hands you back an index — an integer — and the link to crossover is implicit: you use that index to fish an individual out of an array. In Haskell selection is a morphism. Population in, population out, same type on both ends. And every operator shares that signature: `Pop a → M (Pop a)`. A population goes in, a population comes out, and every side effect — randomness, configuration, logging — lives inside that `M`.

That monad is where the leverage is. In Rust you thread a mutable RNG reference through every call by hand, pass the config alongside it, and bolt logging on separately. In Haskell `EvoM` folds all three into one stack: `ReaderT` for config, `WriterT` for logging, `State` for the generator. Want a new effect? You change the stack in one place, not in every signature in the program.

And because the operators all share a type, they compose — Kleisli composition, the fish operator `>=>`. Evaluate, then select, then crossover, then mutate. Six lines where Rust had seventy-five. But the line count isn't the real prize. The prize is that the pipeline is now a *value* — not statements trapped inside a loop, but a thing you can name, pass around, lift through an island functor, swap for another strategy. The composition is out in the open and the type checker is watching it.

That is exactly what made the experiment cheap to run. Once the island model was a functor parameterized by its migration graph, changing topology was a one-line change — swap the graph, hold everything else fixed. Changing domain was just as small — swap the genome type and the fitness function, leave the pipeline shape untouched. In the Rust version either change meant rewriting the loop. In Haskell both are just parameters. That's how 900 experiments across six domains and five topologies ran on one body of code. The categorical structure was in the Rust the whole time. Haskell only made it impossible to miss.

# Slide 8: Why?

Everything so far is empirical. We've shown *that* topology determines diversity; we have not shown *why*. Why should λ₂ predict the ordering so exactly? Why should the result not care which domain you ran? Finding a categorical account of this is open work — the goal is to derive the topology ordering from the structure of the island functor itself, not merely to observe it in the data. If you have an idea for that proof, come find me afterward; I genuinely want to hear it.

And this is the finding I'd defend hardest. That topology *matters* is not surprising — most people would guess as much. What's surprising is the magnitude. Topology doesn't edge out domain; it dominates it by a factor of twenty-four. The decision most practitioners make once, by reflex, and never revisit is the most consequential structural choice in the whole algorithm.
