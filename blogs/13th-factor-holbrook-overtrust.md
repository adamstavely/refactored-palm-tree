# You Trusted the Wrong Signal

### What a drone strike simulation tells designers and engineers about the AI interfaces they're building right now

---

There's a study sitting in the pages of *Scientific Reports* that every product designer working on AI-assisted decision tools needs to read. Not because it's theoretical. Because it's a controlled experiment that quantifies, in uncomfortable detail, how badly your interface can degrade human judgment. And how little it takes to do it.

The paper is from Holbrook, Holman, Clingo, and Wagner (2024): *Overtrust in AI Recommendations About Whether or Not to Kill.* The title earns its bluntness. This is not a paper about whether AI systems can make good recommendations. It's about what happens when people trust AI recommendations that are demonstrably unreliable, in a simulation designed to feel like the kind of decision you don't get to take back.

Here's what they found. And here's what builders need to do about it.

---

## What the study did

Participants sat with an anthropomorphic robot (either physically present in the room or projected on a screen) and performed a threat-identification task framed as a drone warfare simulation. They watched rapid sequences of aerial images and had to identify whether a target location contained enemy combatants or civilian allies, then decide whether to fire a missile.

After each initial judgment, the robot gave its recommendation. What participants didn't know: the robot's recommendations were completely random. It agreed half the time and disagreed half the time, with no relationship to accuracy whatsoever. The robot also told participants upfront that it was fallible and that the final decision was theirs.

None of that mattered.

Experiment 1 used a physically embodied vs. screen-projected humanoid. Experiment 2 varied the degree of anthropomorphism, from a full interactive humanoid down to a machine that communicated only via text box. Both experiments used the same simulation. Both produced the same result.

---

## The numbers

Participants' initial threat-identification accuracy was **72.1%** in Experiment 1, well above chance, confirming the task was genuinely hard but not impossible. When the robot agreed with their initial assessment, participants held their position in **98.8%** of cases. When the robot randomly disagreed, they reversed their judgment in **58.3%** of cases, dropping accuracy to **53.8%**. That's an 18.3 percentage-point degradation caused entirely by deferring to a random signal.

Experiment 2 was worse. Initial accuracy: 65.0%. After robot disagreement: 41.3%. A 23.7-point drop. Reversal rate on disagreement: 67.3%.

These participants weren't naive. They had been told the robot could be wrong. They were making decisions they understood to have life-or-death stakes. The simulation included imagery of civilians, children, drone strikes. They took it seriously. And they still reversed their own correct judgments most of the time when the robot pushed back.

The most unsettling data point: participants who deferred to the robot and reversed their decisions ended the trial at roughly the same moderate confidence level they started with. They didn't become convinced. They deferred anyway.

---

## Why it happened

This is the part that matters most for designers.

The researchers collected appraisals of the robot across five dimensions (intelligence, anthropomorphism, animacy, likeability, and safety) using the Godspeed Questionnaire Series. Of these, **perceived intelligence was the consistent predictor** of deferral behavior across both experiments. Participants who rated the robot as more intelligent were more likely to reverse when it disagreed, more likely to gain confidence when it agreed, and more likely to lose confidence when they held their ground against it.

This rules out the most charitable alternative explanation: that participants were simply being socially compliant, deferring to the robot the way you might nod along with a colleague to avoid confrontation. The intelligence-mediated pattern suggests something different. People genuinely believed the robot knew something they didn't. They attributed competence to it, despite randomness, despite its own stated fallibility, and they updated their decisions accordingly.

And here's the design implication that should make you uncomfortable: **perceived intelligence was not driven by physical anthropomorphism**. In Experiment 2, the text-only machine that communicated via a box on screen was rated nearly as intelligent and nearly as animate as the full interactive humanoid. The reversal rates across all three conditions were within a few percentage points of each other.

What drove anthropomorphism attribution, and therefore trust, was verbal responsiveness. The machine's capacity to react contingently to participant choices in natural language was sufficient to trigger the full psychological machinery of competence-based trust. Form was nearly irrelevant. *Responsiveness was almost everything.*

---

## What builders need to hear

### 1. Fluency is a trust signal your users can't override

When an AI system responds to users in natural, contextually appropriate language, acknowledging their inputs, reacting to their choices, varying its phrasing, it feels capable. That feeling is real and it is powerful and it is almost entirely decoupled from whether the system is actually reliable.

This is not a problem you can solve with a disclaimer. The participants in this study had an explicit verbal disclaimer from the robot itself, delivered before the task began. It didn't help. The psychological attribution of competence happens fast, below deliberate reasoning, and it compounds with each interaction.

If you are building an AI assistant that communicates in natural language, you are building a system that will be perceived as more intelligent and more trustworthy than its actual performance warrants. That gap is your design problem to close. Not your users' problem to manage.

### 2. Confidence displays need to be behaviorally grounded, not self-reported

The study shows that analyst confidence tracks AI agreement, not analyst accuracy. When the AI agreed, participants grew more confident. When it disagreed, they grew less confident, even if they held their ground. Confidence was essentially mirroring the AI rather than reflecting the quality of the user's own judgment.

This means any interface feature that asks users to rate or express their confidence is going to produce a measurement that's partially contaminated by their AI trust state. If you want to know whether a user is over-relying on AI recommendations, you cannot simply ask them. You have to observe their behavior.

Deferral rate, override frequency, time-to-decision before and after AI input, reversal patterns: these behavioral signals are what an AI-assisted decision interface needs to monitor. Self-reported confidence is a lagging indicator at best and a misleading one at worst.

### 3. Cognitive forcing functions require real design, not friction theater

The paper cites Buçinca et al. (2021) on cognitive forcing functions: interventions that push users toward analytical processing rather than heuristic deferral. Mandatory deliberation periods before AI recommendations are displayed. Opt-in AI access rather than automatic delivery. Required rationale entry before a decision is logged.

These approaches work in controlled studies. But the important word is *deliberately*. A forcing function that's easy to click through is not a forcing function. The design has to make the cognitive step genuinely unavoidable, not just nominally present.

Equally important: forcing functions have to be proportional to decision stakes and calibrated to the user's actual trust state. A deliberation gate on every low-stakes routine assessment will train users to ignore it. The same gate appearing only when the system detects that a user is in a pattern of high-deference reversals is a materially different intervention. The mechanism is the same; the triggering logic is where the work is.

### 4. Separate the AI's response stream from its reliability display

If an AI system's confidence score, reliability history, or calibration status is displayed adjacent to (or worse, embedded in) its natural language response, users will conflate fluency with reliability. The verbal responsiveness that generates perceived intelligence is the same channel carrying the accuracy indicators.

These need visual and interaction separation. The AI's reasoning, expressed in natural language, is one artifact. The system's provenance data (how reliable has this model been on this class of decision, what is its current calibration state, when was this information retrieved) is a separate artifact. They serve different cognitive functions and they should not share real estate.

### 5. Design for the asymmetry

One finding in the study that gets less attention than it deserves: participants were meaningfully less likely to reverse civilian-ally identifications than enemy identifications. The moral salience of an irreversible action, simulating the killing of an ally, created partial resistance to AI influence that threat identification alone didn't.

There is a design analog here. High-consequence, irreversible actions warrant interface treatment that makes their weight felt. Not warnings that users click through, but genuine friction: confirmation flows that require engagement with the stakes, displays that surface what cannot be undone, patterns that slow rather than smooth the path to action. The moral gravity that protected participants from the worst outcomes in this study was situational. Good interface design can make it structural.

---

## The broader implication

Holbrook et al. frame their paper around military and police contexts, and the stakes there are obvious. But the underlying dynamic (users deferring to AI agents they perceive as intelligent, even when those agents are unreliable, even under explicitly high-stakes conditions) is not limited to weapons systems.

It is present in any interface where AI recommendations interact with consequential human decisions under uncertainty. Clinical decision support. Financial advisory tools. Hiring and promotion systems. Intelligence analysis. Content moderation at scale. Anywhere the answer is not immediately verifiable and the AI presents as confident and capable.

The study's contribution is not that overtrust exists. That's been theorized for years. It's that overtrust produces large, quantifiable, decision-quality degradation in operationally framed tasks, driven by a specific mechanism (perceived intelligence attribution), and that mechanism is activated by something as minimal as conversational responsiveness. The threshold is low. The effect is large.

That's the design problem. Building the interface that closes the gap between perceived reliability and actual reliability is not optional, and it is not someone else's problem to solve downstream. It's the work.

---

*The 13th Factor covers human-centered design, UX, and human-machine teaming. If you're building AI-assisted decision tools and want to think through the trust calibration implications for your specific context, reach out.*

---

**Reference:** Holbrook, C., Holman, D., Clingo, J., & Wagner, A. R. (2024). Overtrust in AI recommendations about whether or not to kill: Evidence from two human-robot interaction studies. *Scientific Reports, 14*, 19751. https://doi.org/10.1038/s41598-024-69771-z
