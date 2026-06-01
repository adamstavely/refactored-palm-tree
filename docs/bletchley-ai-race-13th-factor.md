# The Bombe Was Never the Point

### What Bletchley Park keeps trying to teach us about the AI race, and what we keep refusing to hear

In November 2023, many of the people who matter most in artificial intelligence flew to a Victorian manor in Milton Keynes and signed their names to a single page. They chose Bletchley Park on purpose. It is the place where, the story goes, a machine broke an unbreakable code and helped win a war. The symbolism was meant to be obvious. Humanity stands again at the threshold of a machine that will change everything, so gather where the last such machine was born and call it the AI Safety Summit.

It was a beautiful choice of venue and a near-perfect misreading of the history.

The Bombe did not break Enigma. People did. The machine was remarkable, but it was never the protagonist. And the distance between what actually happened at Bletchley and the story we tell about it is, almost exactly, the distance that is about to define the AI race.

## The machine was a filter, not an oracle

Here is the part that gets compressed out of the popular telling. Enigma's possible settings ran into the quintillions, far past anything a room of humans could test by hand. The Bombe attacked that space, but it could not start from nothing. It needed a *crib*: an educated human guess about a fragment of the plaintext hiding inside an intercepted message. Analysts knew the Germans were predictable. Weather reports arrived on schedule. Messages opened with stock phrases. A station that signed off the same way every night handed the codebreakers a foothold.

A cryptanalyst supplied that hypothesis. The Bombe then did the one thing it was magnificent at, which was to grind through candidate rotor settings and eliminate the ones that contradicted the crib. What came out was not an answer. It was a shortlist. Humans took that shortlist, tested it, threw out the false positives, and decided whether the day's break was real.

So the actual loop looked nothing like "machine solves problem." It looked like this. A human foraged through noise and formed a hypothesis. A machine collapsed an impossible search space into a tractable one. A human verified the result, placed it in context, and judged what it meant. If any of those human steps failed, the machine's horsepower was worth nothing. Pirolli and Card would later give this shape a name, the sensemaking loop, but Bletchley was running it under deadline and under fire decades before anyone diagrammed it. The Bombe accelerated exactly one expensive step in that loop. It did not replace the loop, and it could not have.

And none of this was the work of a lone genius at a blackboard. By the end of the war roughly ten thousand people worked at Bletchley, the majority of them women. The Wrens who operated the Bombes, the analysts in the huts, the indexers and translators and traffic analysts: this was a sociotechnical system, a human organization with a machine inside it, not a computer with some people attached. The intelligence was a product of the whole apparatus working in concert.

## The discipline of not trusting the answer

This is the piece that should interest anyone building AI for high-stakes work, and it is the piece the monument never mentions.

Breaking Enigma created a second problem that was arguably harder than the first. The decrypts, codenamed Ultra, were almost too good. If commanders acted on every piece of intelligence the moment it arrived, the pattern would have been unmistakable, and the Germans would have concluded their cipher was compromised and changed it. The most valuable source in the war could be destroyed by the simple act of believing it too readily.

So Bletchley and its consumers built an entire discipline of restraint. Need-to-know was enforced ruthlessly. Cover stories were manufactured so that acting on a decrypt could be attributed to a reconnaissance flight or a conveniently placed spy. Sometimes the calculus meant not acting at all, accepting a near-term loss to protect the source for a decision that mattered more.

Read that again with modern eyes. That is trust calibration. Not blind faith in the system's output, and not reflexive rejection of it, but a continuous, costly judgment about *how much* to rely on the machine in *this* instance given *these* stakes. The humans in the loop were not there to rubber-stamp the decrypt. They were there to decide what weight it could bear. The value of Ultra was never the raw decrypt. It was the disciplined human judgment wrapped around it.

The failure mode we now call overtrust would have lost the war. The failure mode we call undertrust would have wasted the single greatest intelligence advantage either side possessed. Winning lived in the narrow, deliberately maintained band between them. That band was held open by people, not by the Bombe.

## The story we told, and the story we are telling now

Watch what happened to the conversation after everyone went home from Bletchley Park in 2023.

The Bletchley Declaration, signed by twenty-eight countries and the European Union, committed in writing to AI that is human-centric, trustworthy and responsible. Whatever its limits, the language pointed at the right thing. It put the human and the question of trust at the center.

Eighteen months later the framing had inverted. The Seoul summit in 2024 kept the thread. Then Paris in February 2025 dropped the word "safety" from the title entirely and rebranded the whole enterprise the AI Action Summit. The mood had moved from cooperation to competition, accelerated by DeepSeek and by the growing conviction that the prize goes to whoever ships fastest. The United States and the United Kingdom declined to sign the closing declaration. The dominant noun was no longer trust. It was race.

In roughly fifteen months we went from a venue chosen to honor a machine, to a declaration that named the human as the point, to a summit that quietly buried the human under the language of speed and dominance. The place kept offering the lesson. We kept choosing the other story.

And it is the same misreading every time. The machine-centric story of Bletchley says the Bombe won. The machine-centric story of 2026 says the race is about compute, model scale, and who crosses the next capability line first. Both stories are seductive because the machine is the visible, quantifiable, demonstrable part. You can photograph a Bombe. You can chart a benchmark. You cannot photograph the judgment of the analyst who decided the decrypt was real, or the commander who decided not to act on it yet.

## The thirteenth factor

The thing that decided the outcome at Bletchley was not the most powerful machine. It was the quality of the human-machine team, and above all the human discipline of knowing how far to trust the output in front of them. That was the decisive variable then. There is no good reason to believe it is a different variable now.

The AI race, as currently framed, optimizes for the part of the system that was never the bottleneck. We are pouring extraordinary effort into building faster, more capable Bombes, which is genuinely worth doing, while treating the human in the loop as a legacy component to be minimized or designed around. But the lesson of the actual codebreaking war is that the human is not the slow part to be engineered out. The human is where the judgment lives. The machine collapses the search space. The person decides what the result is worth, and acts, or wisely does not.

If Bletchley Park deserves a monument, and it does, the monument should not be to the machine that filtered the candidates. It should be to the thousands of people who turned a filter into intelligence, and who understood, under pressure most of us will never face, that the most dangerous thing you can do with a powerful system is believe it without calibration.

That is the factor the race keeps leaving out. It was the thirteenth factor in 1943, and it is the thirteenth factor in the room every time we talk about who is winning. The summit came to Bletchley to honor the machine. It should have come to learn from the people.
