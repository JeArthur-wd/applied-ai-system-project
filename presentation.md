# TuneJudge: 6-minute project presentation

**Portfolio code:** [JeArthur-wd/applied-ai-system-project](https://github.com/JeArthur-wd/applied-ai-system-project)  
**Presenter:** JeArthur  
**Length:** About 6 minutes, including a short live CLI demo

Use each section as one slide. The speaker notes are a ready-to-read script. The demo command runs the deterministic offline planner and needs no API key.

---

## Slide 1 — TuneJudge: explainable music recommendations

- Applied AI System built from the Module 1–3 Music Recommender Simulation
- Turns a natural-language listening request into ranked, explained recommendations
- Focus: retrieval grounding, bounded agent behavior, and reliability

**Speaker notes (about 40 seconds):**

Hi, I’m presenting TuneJudge, an applied AI system built from my earlier Music Recommender Simulation. The original project ranked songs from a small catalog using listener preferences and a transparent scoring formula. TuneJudge adds a natural-language interface: someone can describe a situation or mood, and the system retrieves relevant music notes, builds a profile, ranks songs, and explains its choices. I focused on making each step inspectable and testable, rather than treating a fluent answer as proof that the system understood the request.

---

## Slide 2 — From a scoring prototype to an integrated system

- Original: content-based ranking over a 20-song catalog
- Extension: BM25 retrieval over 36 local music notes
- Planner: rule-based offline by default; optional constrained Claude planner
- Output: recommendations, scores, explanations, sources, and warnings

**Speaker notes (about 50 seconds):**

The starting point is deliberately small. Each song has metadata such as genre, mood, energy, and acousticness. A scoring engine ranks songs against a listener profile, so the result has a reason that can be checked. I extended this with a 36-document local knowledge base covering activities, moods, contexts, and related genres. A BM25 retriever finds notes relevant to the request. The planner combines the request and retrieved hints into a structured profile. It uses deterministic rules offline by default, with an optional Claude planner that is restricted to catalog vocabulary and validated again. The result includes both its recommendations and the notes used to interpret the request.

---

## Slide 3 — Architecture and human review

```text
Request → input guardrails → BM25 retrieval → profile planning
        → profile validation → song ranking → self-check / revision
        → output validation → recommendations and sources
```

- Human reviews the interpretation and recommendations
- Bounded revision loop; no open-ended tool execution
- Full diagram: [`diagrams/architecture.mmd`](diagrams/architecture.mmd)

**Speaker notes (about 50 seconds):**

This is the end-to-end flow. Input guardrails reject empty, oversized, or suspicious requests. Retrieval looks up local notes rather than relying only on a model’s general knowledge. The planner produces a profile, then a profile guardrail normalizes and checks it before ranking. The agent checks its first ranking for fixable issues, such as a genre missing from the catalog or repeated artists. It can revise a limited number of times, then an output guardrail checks the final recommendations. The listener stays in the loop: they can see how the request was interpreted, which sources were retrieved, and what the system recommends, then adjust the request.

---

## Slide 4 — Live demo: a rainy study request

Run from the repository root after installing requirements:

```powershell
$env:TUNEJUDGE_USE_LLM = "0"
python -m src.main --ask "something calm for studying on a rainy day" --trace -k 3
```

Observed behavior:

- Retrieves *Rainy days*, *Studying and focus*, *Lo-fi hip hop*, and *Chill and relaxed*
- Plans a lofi / chill profile at energy 0.25
- Detects a repeated artist and revises with diversity enabled
- Returns *Library Rain*, *Midnight Coding*, and *Spacewalk Thoughts* with reasons

**Speaker notes (about 1 minute 20 seconds, including running the command):**

I’ll run one request through the actual command-line application. I’m forcing the offline planner so anyone can reproduce this without an API key. The trace first shows the input passed its guardrail. Retrieval finds four relevant notes about rainy days, studying, lo-fi, and chill music. The planner turns that into a lofi and chill profile with a low energy target. After the first ranking, the self-check notices the same artist appears more than once and turns on the diversity penalty. The revised results are Library Rain, Midnight Coding, and Spacewalk Thoughts. The output also gives reasons and source notes. The README includes this interaction and two other full request-and-output examples, including a “punk please” request where retrieval maps a genre missing from the catalog to related rock.

---

## Slide 5 — Reliability and guardrails

| Evidence | Result |
|---|---:|
| Automated tests | 71 / 71 passed |
| Offline reliability checks | 170 / 170 passed |
| Golden-request average confidence | 0.93 |

- Checks include expected outputs, repeatability, wording perturbations, fuzzed profiles, and output validation
- Run: `python -m src.evaluate` and `pytest`

**Speaker notes (about 55 seconds):**

I evaluated more than a few happy-path examples. The test suite passed 71 tests. The separate offline evaluator passed 170 checks: golden requests, consistency across repeated runs, robustness to case and whitespace changes, 200 corrupted profiles, and output validation across the preset profiles and ranking modes. During development, the evaluator exposed issues with an infinite decade value, sleep-related retrieval, and whitespace normalization. Those cases were fixed and covered. The confidence value measures how well the catalog matches the parsed profile; it is not a probability that the system understood a person correctly. The optional Claude path is tested with mocks, but these offline scores do not evaluate live model behavior.

---

## Slide 6 — Responsible design and limits

- Small, hand-curated catalog; song attributes and retrieval hints encode assumptions
- Keyword planner can miss sarcasm, comparisons, or unfamiliar wording
- Prompt filtering is not a complete defense
- Keep the user in control; expose interpretation, sources, and reasons

**Speaker notes (about 55 seconds):**

TuneJudge has important limits. It has only 20 songs, and many features are hand-assigned or synthetic rather than measured from audio. The knowledge base and genre labels reflect assumptions about what music fits an activity. The rule planner can miss nuance, and prompt-injection filters can be bypassed. The optional model is restricted to structured profile fields and its output is validated, but that does not make the interpretation infallible. I designed the interface to show its interpretation, evidence, and recommendation reasons so a listener can notice a mismatch and try again. I would not use this small prototype to make high-stakes decisions or claim objective musical quality.

---

## Slide 7 — What I learned

- Retrieval changes how ambiguous requests are interpreted
- A bounded agent is easier to inspect and test than an open-ended one
- Reliability failures often appear in edge cases, not polished examples
- Portfolio reflection: build AI that people can understand, evaluate, and correct

**Speaker notes (about 45 seconds):**

This project taught me that system behavior depends on the data representation and the boundaries around the model, not only on the model itself. Retrieval made ambiguous requests more grounded. The bounded workflow made behavior easier to trace. Reliability testing found bugs that looked plausible in a demo. As an AI engineer, I want to build systems that expose what they used, test what can go wrong, and make it possible for people to correct them. The code, README, model card, and this presentation are all available in the public GitHub repository. Thank you.
