# Building and Evaluating a Personal Engineering Knowledge Base for AI Agents

_A well-organised belief, measured._

![Article cover - The title beside code, notes and checklist pages feeding into an open book](./assets/ai-engineering-kb.avif)

Second brains are usually discussed in two extremes. Some treat them as a lifestyle, with elaborate systems for capturing every thought. Others dismiss the whole idea as note-taking with extra steps. Mine started as neither. It started as an annoyance.

Over the last couple of years, most of my work shifted to AI-assisted engineering, with an agent in the loop for almost every task. And I caught myself pasting the same prompt fragments into session after session: my conventions, my preferences, the same lessons I had already learned once. Every new project meant re-explaining the same things to the same tools, and every explanation drifted a little from the last one.

At some point the explanations became files. The files grew into a private git repository, just under 200 markdown documents by now, and every repo I work in carries a short manifest at its root that points whichever agent I am using that day at it.

For the first few months, I could not answer the obvious question. Does any of this actually help? I believed it did. In July I measured it, and some of what came back was uncomfortable.

---

## Standards, Not Notes

The first thing that separates this from a notes vault is the kind of writing that goes in. The core idea fits in one line:

> A note records what you learned. A standard decides what happens next time.

Where a note might say "connection pooling matters", a standard names the exact value and the reasoning behind it. The difference matters because the files have two readers: me, and the agent that receives them as context. An agent given "be careful with timeouts" produces careful-sounding prose. An agent given an exact value, with the reasoning attached, can produce the right code.

That constraint shaped how everything is written:

- plain prose, no decorative formatting that wastes tokens
- one concern per file, so a task can load only what it needs
- exact values instead of adjectives
- a mark on every claim: did this come from a real incident, or is it a reasoned default?

Ephemeral things stay out. Sprint work, in-progress bugs, and temporary decisions live in tickets; the corpus only holds what is meant to stay true for years.

It is not a wiki or a notes app. Every change passes through review, and the files are written for agents as much as for people. It is not a product either. The repository is private. What transfers is the pattern, not the implementation.

One file matters more than the rest: the personal layer. The standards are things most senior engineers would agree with. The personal layer holds my own defaults, and the lessons from my own production incidents. Team standards tell an agent how to engineer. The personal layer tells it how I engineer.

---

## Where the Files Come From

The most useful files were paid for.

One example. Debezium, the change-data-capture connector, holds a replication slot open against PostgreSQL. While the connector runs, the database recycles its write-ahead log as usual. When the connector dies, the slot stops advancing, and PostgreSQL keeps every log segment the dead consumer might still need. Nothing errors. Disk usage climbs quietly until something unrelated falls over. On a write-heavy system, a connector that has been down for half a day can pin tens of gigabytes of log. The lesson that went into the personal layer is simple: monitor slot lag, and alert before it threatens the disk.

Entries like that are tagged as incident-derived. Untagged entries are reasoned defaults. That distinction sounds small, but it is what makes a pile of claims trustworthy: a reader, human or agent, can tell which ones are load-bearing. And because everything lives in git, every lesson lands as a diff with a date and a reason, never from memory alone.

---

## One Source of Truth, Outside Every Tool

I use Claude Code most days and Antigravity often. Cursor comes and goes. Every one of these tools has its own way to hold context: skills, rules files, memory features. They are all good, and they are all vendor formats. Knowledge written into one does not travel to the next.

So the knowledge lives outside all of them, as plain markdown, and each tool just gets pointed at it. A browser chat with no filesystem can receive the same content as a pasted bundle. The tools stay replaceable; the source of truth does not move.

This is also what makes new projects cheap. Each project carries a thin manifest: one page that says which parts of the brain apply here, plus the project's own conventions. Setting up a new repo means dropping in that manifest and picking the presets that fit. A preset is a saved list of brain files for a common kind of project; a Go service gets one list, a static site gets another. Nothing gets re-taught, and the drift that started this whole effort loses its mechanism: there is nothing to copy. A migration gets the same treatment: the discipline for a schema change or a framework move is already written down, and the agent follows the playbook rather than my memory of it.

Not everything loads, and that is deliberate. The corpus outgrew what I can usefully load into a context window a while ago, and loading everything would be pointless anyway: cost and attention dilution arrive before the hard limit does. The manifest points, and the agent opens a file only when the task touches it. What it opens is layered: universal constraints first, then a role, then the technical depth the task needs. Each layer narrows the focus without dragging in the rest.

Review is where the brain does most of its daily work. Before work merges, an agent with the right standards loaded reads the diff against them; the same move confirms an analysis or pressure-tests a finding before I act on one. The role files set the perspective the agent holds while doing it: what a backend engineer treats as non-negotiable, where a security reviewer pushes back. Review panels stage several roles at once. It is the closest I have found to a team looking over the work.

This is also the least tested layer: whether a role changes the substance of a review, or only its tone, I have not measured. My guess is that the standards carry the substance and the role carries the emphasis, but a guess is exactly what this article argues against trusting. The test would be the same A/B as below, with the role swapped instead of the files.

If your corpus is ten files, none of this machinery is worth it. A single instructions file at the repo root, a `CLAUDE.md` or its equivalent, is enough. The machinery earns its keep only after the knowledge outgrows what one file, or one memory, can hold.

---

## It Maintains Itself, Mostly

Documentation rots because nothing checks it. So the corpus has a small linter that runs on every change: broken links and unresolvable references fail the commit, and stale review dates come back as warnings. The linter only catches the mechanical kind of rot, though. The kind that matters more is guidance that quietly stops being true, because a platform changed or an incident superseded a rule, and no gate detects that. So once a quarter I walk the tree and prune. The priority is accuracy over completeness. A small set of current, trusted rules beats a large one where you cannot say what still applies.

The newer part is that capture is increasingly delegated. At the end of a build, I point an agent at the codebase and its git history, and ask it to draft the lesson entries: what broke, which defaults proved wrong. The drafts are only drafts. Nothing lands until I have fact-checked each claim against what actually happened. That step stays manual on purpose. Deciding what is true is the one job I do not want to delegate. Then the gate runs: the same linter that checks every change.

That loop is why each project starts a little further ahead than the last one.

---

## Does It Actually Help?

The claim behind the entire system is simple: an agent that reads the corpus should make better engineering decisions than one that does not. The conventions, the constraints, the lessons from old mistakes, all arrive with it.

But a beautifully organised system can still do nothing. A strong model might already know everything in those files. Until July, my only evidence was that answers felt better.

> A knowledge base whose impact is unmeasured is a well-organised belief.

So I measured it.

- Give the same task to two fresh agents: one handed the files I judged relevant when writing the task, one working from its own knowledge only.
- A blind judge, a separate model instance, scores each answer by how much of a checklist it covers. The checklist is written in advance, the points a good answer must contain: covering none of it scores 0, covering all of it scores 1. Each pair is judged three times and averaged.
- Repeat across fifteen tasks, including a control group of textbook questions where the corpus should not help at all. If the corpus had "won" there, that would have meant a broken eval, not a good corpus.

One bias worth knowing about: the generators and the judge share a base model, so self-preference is possible; the blind checklist narrows it without removing it.

There is a fair question to ask of any eval you run on your own system: what would it look like if it were just flattering me? Lift everywhere, including on the control questions; numbers that grow as the eval matures; and no results that ever hurt the corpus. What came back was the opposite on all three counts.

| Knowledge type                      | Expectation | Result (15 tasks)            |
| ----------------------------------- | ----------- | ---------------------------- |
| Incident-derived (scar tissue)      | should lift | large: +0.25, won all 5 of 5 |
| Reasoned defaults (my own opinions) | modest lift | small: +0.07                 |
| Textbook material (control)         | no lift     | none: roughly zero           |

The overall lift was about +0.1. In plain terms: with the files, an answer covered about a tenth more of the checklist than without them. And it shrank as the eval improved. An earlier, narrower run had reported nearly twice that; adding broader tasks produced the smaller, more honest number. Almost all of the lift came from the incident-derived entries. The pattern is easy to explain: a strong model already knows the textbook. It has never seen my incidents. The highest-value context is the part no training run could contain: what failed here, which constraint mattered, and what the failure taught.

Two tasks actively regressed. The sharper case: on a question the bare model answered perfectly, the agent that had read the relevant file did slightly worse, because the file had a gap and the agent anchored on what the file did cover. Reading the file made the answer worse. I fixed the gap, confirmed it on a re-run, and took away a rule I now apply everywhere: an incomplete authority in the context window can displace knowledge the model already had. The rule reaches past second brains: any file an agent is told to trust, a rules file and a retrieved document alike, does damage when it has gaps.

Fifteen tasks is a small sample, the anchors were written by the same person who wrote the corpus, and a task set you keep fixing content against will eventually flatter you. The repo is private, so you cannot check my numbers either; I hold the result loosely. But it settled the question I actually had. The part of the brain that pays is the part written from incidents. Most of the rest measured somewhere between modest and nothing, and if I were starting again, the ratio would be different.

---

## A Note on Trade-offs

This setup has real costs, and they are worth naming plainly.

- Nothing auto-invokes. Skills and rules files fire themselves; the brain waits to be pointed at. Against modern tooling, this is its weakest point.
- It only informs. The moment a piece of knowledge needs to do something, it has to graduate out of the corpus and into a tool.
- The platform is catching up. Imports and task-scoped loading are going native, and context windows grow every year. I assume the edge narrows every quarter.
- Maintenance is a standing tax. The gate slows rot without stopping it. I measured the benefit this month, and I have not measured this cost yet.

Some of this is deliberate, though. The season we are in is one-click automation, and much of it is genuinely useful. But for the layer that decides how I work, I want plain text I can read and a decision that stays mine. The brain points at what to check; acting on it stays my job, and that ownership is much of why the system fits the way I work.

These costs argue for layering rather than choosing. Knowledge stays in the corpus; the most-used slices become thin auto-loading pointers back into it; anything that must execute becomes a tool. The dependency runs from tool to brain, never the other way.

---

## Final Thoughts

Little of this depends on the file format, or on the tools of this particular season. One concern per file, provenance on claims, a gate in front of changes, a measurement you can re-run: that is the part worth copying.

If you build one, start small. A thin manifest in one project, pointing at a dozen files that state exact values. At that size a single `CLAUDE.md` works too; split into files when it stops fitting in one. Write them from your incidents, not your opinions; incident-derived knowledge was the only kind that reliably paid. The linter can wait until links start breaking. The eval can wait until the day you catch yourself telling someone the corpus works, because that claim deserves a number you can re-run when the corpus changes.

Mine turned out to be about +0.1. Smaller than I had guessed, and concentrated almost entirely where the scars are. That result quietly redefines what I built. I set out to write down what I know; what turned out to be worth keeping is what reality taught me when I was wrong. I trust it more than the feeling it replaced.
