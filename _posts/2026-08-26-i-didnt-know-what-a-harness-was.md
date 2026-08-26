---
layout: post
title: "I Didn't Know What a Harness Was"
date: 2026-08-26
description: >-
  What I learned building an autonomous AI agent from scratch, including the
  uncomfortable question of using AI to build the thing that harnesses AI.
---

A year ago I sat in a room at Boston Fintech Week and listened to a keynote
speaker get asked what the focal point would be when everyone reconvened the
following year. He said: AI agents.

I wrote it down. That was the entire extent of my response. I carelessly noted
it and didn't think much of it, and now agents are what everyone is talking
about and I'm late to the party — because I heard something and mindlessly
wrote it down with no action plan.

I'd been around AI for a while by then. Computer vision projects, machine
learning coursework, using the tools like everyone else. For some reason I
never went into agents the way I should have.

So this summer I built one. A local, permissioned autonomous agent, from
nothing, over about two months.

## I didn't know what a harness was

That is the honest starting point. Not "I had a rough idea." I didn't know
the word.

I had heard of the LangGraph family without really knowing what it was. I knew
containers existed as a concept, via Docker, but had never used one. I
understood the Model Context Protocol well enough to explain it and had never
applied it to anything real.

A harness, I now understand, is everything around the model. The model is one
component and arguably the least interesting one. The harness is what decides
which tool gets called, what happens when that tool fails, what the agent is
allowed to do without asking, what it must stop and ask about, where generated
code is permitted to run, and what happens when the model produces something
confident and wrong.

That last one turned out to be the whole game.

## The principle I'd defend

The single idea that shaped everything else in my agent:

**The LLM writes text. Deterministic code makes the consequential decisions.**

The model drafts the plan. It does not pick the tool. Tool selection runs
through code I can read, test, and reproduce.

The reasoning is uncomfortable but simple. A model is probabilistic. When it
picks the wrong tool, every node downstream trusts that pick and builds on it,
and what comes out the other end is a confident hallucination. Not an error
message — an answer, delivered with the same tone as a correct one.

The worst version of this in my system: someone says "send the email" and the
router quietly selects "search emails." It looks like it worked. Nothing
throws. The email was never sent.

So I built a deterministic guard that classifies write intent separately from
the model, and vetoes a read-tool selection when the user clearly asked to
write. Not because it's elegant. Because that failure is silent, and silent
failures are the ones that hurt.

## "Regex is a bandaid"

My tool routing started as a large regex layer — roughly 1,500 lines turning
phrasings into tool calls. It worked. I was reasonably proud of it.

Then my manager told me regex was a bandaid.

He wasn't wrong, and my first instinct was to argue. What I did instead was
go find out what frontier labs actually do — trained function-calling,
structured outputs, and evaluation harnesses — and then build that version on
my local model, keep the regex behind a flag so the whole thing was
reversible, and *measure which one was better.*

That reframing is the most useful thing I took from the internship. The
question stopped being "is regex bad" and became "bad compared to what, on
what data, by how much."

## The number that embarrassed me

I wrote 488 prompts and split them: 300 to tune on, 188 sealed in a drawer.
Thresholds were swept on the training split only. The hold-out got opened
once, at the very end.

My regex router had been scoring near 100%.

On the sealed set it scored **51.1%**.

That gap is the entire lesson. The old number wasn't a lie, it was train
accuracy on its own examples — I had tuned the regex against the same
phrasings I was testing it with, and then believed the result. It's the
oldest mistake in machine learning and I walked directly into it while
thinking I was being careful.

The cascade router I replaced it with — a local model first, a second model
behind it, regex only as a tiebreak — scored **89.9%** on the same sealed set.
More importantly it got read-versus-write right **100%** of the time, against
19 dangerous misroutes from the regex.

Worth being precise about what that does and doesn't mean: 89.9% is against
*my* set of roughly 45 tools, using a 7B model running locally on my machine.
A frontier model would score higher. That's rather the point — the interesting
result isn't that it's state of the art, it's that a small local model plus a
carefully built harness was good enough to beat the thing I'd hand-tuned for
weeks.

The harness did more work than the model did.

## The uncomfortable part

Here is the thing I keep circling.

The same way AI is a loop of processes, the use of AI is a circle for a
consumer like me as well. I write code with AI. I ask questions about my code
to AI. I brainstorm with AI. And then I sat down to build an agent — a harness
whose entire purpose is deciding when to trust a model's output — using a
model to help me build it.

So: am I validating my work with AI, or walking into a loop of bias I should
be avoiding?

My honest answer is that it depends, and that the distinction is whether
something outside the loop can tell you that you're wrong.

When I asked AI whether my routing approach was sound, I got agreement. Useful
agreement, even. But agreement is what that interaction produces, and I can't
tell the difference between "this is correct" and "this is plausible" from
inside the conversation.

The sealed hold-out told me I was wrong. 51.1% is not an opinion and it did not
care how confident I was. My manager saying "regex is a bandaid" — that was
outside the loop too.

That's the rule I've landed on. Use AI to build, absolutely. But every claim
that matters needs a check that can independently return "no." A test that
fails. A hold-out you only open once. A person who'll tell you the thing you
don't want to hear. Without at least one of those, you're not validating
anything, you're just generating more confidence.

## Where I actually am

I'm in no way an expert in any of this. If anything, building the thing showed
me how far I am from going from a novice to a master. I now know what a harness
is, which mostly means I know how much of it I haven't built yet.

But then again, isn't that the joy of it? When I'm tired but I still work.

---

The agent is open source: **[langgraph-code-agent](https://github.com/mohammadeissa/langgraph-code-agent)**
— the cascade router, the evaluation harness, and the methodology write-up with
the full per-service breakdown are all in there.

<!-- TODO(Mohammad): a few places to make this more yours —
     1. The Boston Fintech Week keynote — name the speaker/session if you want it concrete.
     2. "My manager told me regex was a bandaid" — reword however you'd prefer to characterise it.
     3. Consider adding one specific moment where the agent failed in a way that surprised you.
     4. Delete this comment block before publishing. -->
