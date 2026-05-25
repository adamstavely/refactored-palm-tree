# Your Search Bar Is Not a Sensemaking Tool

### What Pirolli and Card's foundational model still gets right about how analysis actually works, and what most AI tools still get wrong

---

In 2005, Peter Pirolli and Stuart Card published a paper that should be required reading for anyone building tools for knowledge workers. It appeared in the proceedings of the International Conference on Intelligence Analysis, it drew on cognitive task analysis and verbal protocol studies of real analysts, and it produced a model of the analysis process that remains more accurate than most of the software built to support it.

The paper is "The Sensemaking Process and Leverage Points for Analyst Technology." It is twenty years old. The problems it identifies are not.

---

## The model

Pirolli and Card describe intelligence analysis as a sensemaking process organized around two interconnected loops. The first is the foraging loop. The second is the sensemaking loop. They are distinct in their cognitive demands, their outputs, and the kinds of tool support they require. Most AI-assisted analysis platforms conflate them or serve only one.

The foraging loop is about information. Analysts search for it, filter it, organize it, and narrow it down to a working set of material relevant to the problem. The sensemaking loop is about meaning. Analysts take the material they've gathered and build schemas, generate hypotheses, marshal evidence, and produce assessments.

These loops are not sequential. Analysts move between them continuously, in what Pirolli and Card call an opportunistic mix of bottom-up processing (from data toward theory) and top-down processing (from theory back toward data). A hypothesis generated in the sensemaking loop sends the analyst back into the foraging loop looking for confirming or disconfirming evidence. A pattern spotted while foraging pulls the analyst into the sensemaking loop to update their schema. The model describes sixteen distinct steps connecting these loops, and the movement between them is iterative, recursive, and rarely clean.

That's the model. It is not complicated. It is, however, routinely ignored in product design.

---

## What the foraging loop actually requires

Foraging is not just search. That distinction matters enormously for tool design.

Search is a point query: you know roughly what you're looking for, you construct a query, you retrieve results. Foraging is a process of discovery under uncertainty: you don't fully know what's relevant until you've seen enough of the information space to recognize what's missing, what's anomalous, what clusters with what.

Pirolli draws on information foraging theory, which he developed with Card across the 1990s, to describe how analysts navigate this uncertainty. They follow information scent, adjusting their path based on signals of relevance that they're evaluating continuously and often implicitly. They explore broadly first, then narrow. They exploit promising clusters while remaining open to pivots. The process looks inefficient from the outside. From the inside, it's how you avoid anchoring on the first plausible hypothesis and missing the more important one hiding three lateral moves away.

The design implication is that foraging tools need to support exploration as much as retrieval. A search bar optimized for precision, returning exactly what you asked for, is often the wrong tool for an analyst who doesn't yet know what to ask. Faceted browsing, lateral recommendation, anomaly surfacing, visual clustering: these are foraging affordances. They are underbuilt in most enterprise analysis platforms because retrieval is easier to measure and precision is easier to demo.

---

## What the sensemaking loop actually requires

The sensemaking loop is where analysis happens. It is also where most AI tools either stop at the door or actively interfere.

The sensemaking loop involves building a representation of the problem: a schema, a mental model, a hypothesis structure that organizes what the evidence means and what it implies. This representation is not a database entry. It is a working cognitive artifact that the analyst holds, tests, revises, and refines across the course of the analysis. It is the thing that gets turned into an assessment, a report, a recommendation.

Pirolli identifies the leverage points in the sensemaking loop as: problem structuring (generating, exploring, and managing hypotheses), evidentiary reasoning (marshalling evidence to support or disconfirm hypotheses), and argumentation (building and testing chains of inference). These are the cognitive operations that determine whether the final assessment is good. They are also the operations that AI tools are least likely to support directly.

What AI tools typically offer instead is summarization. They compress the information space, surface the most salient content, and return a digest. Summarization helps with foraging. It does not help with sensemaking. A summary of the evidence is not the same as a structured representation of the hypothesis space. Giving an analyst a summary and calling it analysis support is like giving a surgeon a medical textbook and calling it a surgical tool.

This is not a criticism of summarization as a capability. It's a criticism of its position in the product. Summarization belongs in the foraging loop, as a tool for narrowing the information space to a workable set. When it is positioned as the primary AI contribution to the analysis, it short-circuits the sensemaking loop rather than supporting it.

---

## What the opportunistic mix means for workflow design

The most consequential finding in the Pirolli and Card model, for tool designers, is the characterization of analysis as an opportunistic mix of bottom-up and top-down processing. Analysts do not proceed linearly from data collection through hypothesis generation to assessment. They move back and forth continuously. The direction of movement is determined by what they find, not by a workflow template.

This creates a serious problem for tool design. Most enterprise software imposes linear workflows: gather, analyze, report. The gathering phase hands off to the analysis phase, which hands off to the reporting phase. These phases are often implemented as separate views, separate modules, or even separate applications. The data is in one tool. The hypothesis management is in another. The reporting is in a third.

Every transition between tools is a context switch that imposes cognitive overhead and creates friction in the natural movement between the foraging and sensemaking loops. When an analyst needs to pivot from their current hypothesis back into the information space to test a new angle, they shouldn't have to change applications. The loops need to be co-present, accessible from the same cognitive workspace, with movement between them requiring minimal friction.

The design target is a unified workspace that makes both loops visible and supports transitions between them without forcing the analyst to reorganize their mental state each time they switch directions.

---

## What builders need to take from this

### The foraging loop is necessary but not sufficient

Search, retrieval, filtering, summarization: these are table stakes. They are also the primary value proposition of most AI-assisted analysis tools on the market. That's not enough. A tool that helps analysts find information faster but provides no structured support for what they do with that information has automated the cheaper half of the job.

The more consequential design question is not "how do we surface the right information?" It is "how do we support the analyst in building and testing the right representation of what that information means?"

### Hypothesis management is a first-class feature, not a nice-to-have

The sensemaking loop runs on hypotheses. Analysts generate them, test them against evidence, discard some, refine others, and eventually converge on the ones that the evidence best supports. This process needs explicit tool support: a place to hold competing hypotheses, a way to tag evidence to them, a mechanism for tracking which hypotheses have been tested and which haven't, a view that makes the current state of the evidential argument visible.

Most tools treat notes and annotations as the solution to this problem. They aren't. A note is unstructured. A hypothesis is a structured claim with an associated body of evidence and a current confidence state. These are different artifacts with different design requirements.

### The schema is the unit of work, not the document

Analysts think in schemas: mental representations of how the pieces fit together, what the story is, who the actors are, what the causal structure looks like. The document is a downstream product of the schema. Most analysis tools are organized around the document: creating it, populating it, formatting it, approving it.

A tool organized around the schema would look different. It would surface the analyst's current representation of the problem as a first-class object in the interface. It would allow that representation to be revised as new evidence arrives. It would make explicit what is known, what is inferred, and what is assumed. It would show the analyst and their collaborators where the representation is fragile.

None of this is impossible. It is rarely built because documents are more legible to stakeholders than schemas, and legibility to stakeholders tends to drive product decisions.

### AI assistance belongs in both loops, with different designs for each

An AI capability positioned in the foraging loop should help analysts navigate the information space: surfacing relevant material they haven't seen, identifying anomalies that don't fit the current schema, flagging lateral connections across source types. The design goal is to extend the analyst's reach without narrowing their aperture.

An AI capability positioned in the sensemaking loop should help analysts stress-test their reasoning: identifying alternative hypotheses that the evidence could support, surfacing evidence that cuts against the current schema, flagging logical gaps in the argument structure. The design goal is to increase the quality of the reasoning, not to accelerate it.

These are different goals and they require different AI behaviors and different interface treatments. Conflating them produces a tool that does neither well.

---

## Why this still matters twenty years later

The Pirolli and Card paper was published before large language models existed. The analysis tools of 2005 were structured databases, search engines, and document management systems. The leverage points they identified were theoretical targets, capabilities that the technology of the time couldn't easily reach.

The technology has changed. The model is still right.

Large language models are powerful foraging tools. They retrieve, they summarize, they surface connections across large information spaces faster than any prior technology. They are also being deployed as sensemaking tools, generating assessments, producing conclusions, drafting finished intelligence, without the underlying hypothesis structure, the evidentiary scaffolding, or the explicit reasoning chain that the sensemaking loop requires.

The risk is not that the models are wrong, though they sometimes are. The risk is that they produce outputs that look like the products of the sensemaking loop while bypassing the process that gives those products their epistemic grounding. A fluent, confident summary of a complex situation is not an analysis. It is a very good-looking foraging output that has been formatted to resemble one.

Pirolli and Card give us the vocabulary to name this problem and the model to address it. The leverage points they identified in 2005 are still largely unbuilt. That's the work.

---

*The 13th Factor covers human-centered design, UX, and human-machine teaming.*

---

**Reference:** Pirolli, P., & Card, S. K. (2005). The sensemaking process and leverage points for analyst technology as identified through cognitive task analysis. *Proceedings of the International Conference on Intelligence Analysis,* 2-4. https://analysis.mitre.org/proceedings/Final_Papers_Files/206_Camera_Ready_Paper.pdf
