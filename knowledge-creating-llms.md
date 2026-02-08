# Knowledge-Creating LLMs: The Shift from Democratization to Discovery

**Source:** [Tom Cunningham @ METR](https://tecunningham.github.io/posts/2026-01-29-knowledge-creating-llms.html)  
**Published:** January 29, 2026

---

## The Core Thesis

Cunningham argues we're witnessing a fundamental phase transition in what LLMs *are*. The first wave—GPT-3 through early Claude—was about **knowledge-sharing**: compressing human expertise into accessible APIs, democratizing access to what we already knew. The emerging wave is **knowledge-creating**: LLMs that can extend beyond the frontier of documented human knowledge.

This isn't incremental improvement. It's a categorical shift with profound implications for business models, intellectual property, and how value flows through the economy.

---

## Two Types of LLMs

### Knowledge-Sharing LLMs

- **Training signal:** Human judgment as ground truth (RLHF, preference learning)
- **Capability ceiling:** Rarely exceed human expertise—they're bounded by what humans can evaluate
- **Economic value:** Democratizing access to existing knowledge
- **Cost structure:** High fixed costs, near-zero marginal costs
- **Business model:** Broad API access, subscription pricing

**Observable patterns:**
- Higher adoption among junior professionals facing unfamiliar problems
- Greater use *outside* one's expertise area (lawyers asking medical questions)
- Disproportionate value in well-documented domains
- Decreased "knowledge rents"—premium compensation for rare expertise erodes
- Increased home production substituting for market-provided services

### Knowledge-Creating LLMs

- **Training signal:** Real-world verification systems (Reinforcement Learning from Verifiable Rewards)
- **Capability ceiling:** Can discover solutions beyond documented knowledge
- **Economic value:** Novel discoveries with exclusive value
- **Cost structure:** High returns to exclusivity
- **Business model:** Licensing, IP-based agreements, outcome-based pricing

---

## Evidence: The Discoveries Are Already Happening

### AlphaEvolve (Google)

An "autonomous pipeline of LLMs whose task is to improve an algorithm by making direct changes to code."

Achievements:
- More efficient data center scheduling algorithms
- Novel mathematical solutions
- **427× speedup** on optimization tasks using experimental scaffold (190× with standard scaffold)

### TTT-Discover

State-of-the-art results across:
- Mathematics
- GPU kernel engineering
- Algorithm design
- Biology

**2× faster GPU kernel performance** vs. prior art.

### Claude Opus 4.6

Per Anthropic's system card: achieved "427× best speedup using experimental scaffold" on kernel optimization.

---

## The "Dozen Deep Problems" Framework

Cunningham proposes that complex problem domains reduce to approximately **twelve canonical problem types**:

1. **Constraint-satisfaction problems** — optimization challenges equivalent to 3SAT or similar
2. **Factual problems** — retrieving and inferring from documented facts
3. **Statistical inference problems** — applying best-practice ML techniques

If true, this creates a benchmarking crisis: once LLMs learn to map idiosyncratic problems onto canonical forms and apply textbook solutions, it becomes difficult to assess genuine intelligence advances.

The remaining good benchmarks involve **"emulsions" of data and logic**—problems requiring both deductive reasoning and statistical inference that can't be solved by pattern-matching to known forms.

---

## Economic Model: Why Exclusivity Matters

Cunningham sketches a simplified model of a "disembodied knowledge economy":

- **Aggregate output** is determined by the best available knowledge
- **Aggregate profit** is determined by the gap between best and second-best knowledge
- **Price equilibrium** reached when the lowest-cost producer serves the market

Two implications:

1. **Knowledge-sharing spreads output but doesn't increase it.** Distributing the best recipe eliminates profit concentration but doesn't grow the pie.

2. **Knowledge-creation increases total output.** Improving the best recipe grows the pie. Profit effects depend on whether competitive leadership changes hands.

This explains why novel discoveries command premium pricing: exclusivity captures significant value, while shared knowledge faces competition that eliminates rents.

---

## The Coming IP Land-Grab

If LLMs create high-value discoveries, firms will race to patent novel technologies. Cunningham notes this exclusivity may be economically *inefficient*—discoveries might occur regardless of patent incentives—but predicts competitive dynamics will drive aggressive IP acquisition anyway.

OpenAI CFO Sarah Friar: "As intelligence moves into scientific research, drug discovery, energy systems, and financial modeling, new economic models will emerge. Licensing, IP-based agreements, and outcome-based pricing will share in the value created."

Target domains:
- AI R&D stack optimization (the most recursive application)
- Drug discovery
- Novel algorithm development
- Trading algorithm improvement
- High-quality cultural product generation

---

## The Data Bottleneck Question

Will knowledge-creation be constrained by experimental capacity?

The answer depends on **optimization landscape dimensionality**:

- **If low-dimensional structure exists:** Computational thinking yields high returns. LLMs can explore the space efficiently with limited real-world data.
- **If genuinely high-dimensional:** Real-world experimentation becomes necessary. Discovery bottlenecks at the speed of physical verification.

This is the key uncertainty for predicting how fast knowledge-creating LLMs can compound.

---

## Visual Framework

Cunningham includes two illustrative graphs:

1. **Knowledge-sharing LLM:** A lower bound across three human specialists. The LLM matches or slightly exceeds human performance across all domains—democratizing access but not extending the frontier.

2. **Knowledge-creating LLM:** Pushing *below* the human frontier on select tasks. The LLM discovers solutions humans haven't found, creating novel value.

---

## What This Means

The shift from knowledge-sharing to knowledge-creating represents a change in what LLMs *are*:

**Before:** Compressed libraries of human expertise. Value from access democratization. Business model: subscriptions and API calls.

**After:** Discovery engines that extend human knowledge. Value from novel insights. Business model: licensing, IP agreements, outcome-based pricing.

The economic implications flow directly from this distinction. Knowledge-sharing is a cost-reduction story. Knowledge-creation is a value-generation story—with winner-take-most dynamics for those who own the discoveries.

---

## References

- Chatterji et al. (2025) — ChatGPT usage patterns study
- Abreu et al. (2025) — Taxonomy of Transcendence
- Amodei (2024) — "Machines of Loving Grace"
- Romera-Paredes et al. (2023) — Prior automated discovery methods
- EPOCH — FrontierMath Open Problems benchmark

---

*Author notes the economic model remains preliminary and is seeking theorist collaborators for formalization.*
