---
layout: post
title: "I Didn't Know What a Harness Was"
date: 2026-08-26
description: >-
  What I learned building an autonomous AI agent from scratch, including the
  uncomfortable idea of using AI to build an AI agent.
---

A year ago I sat in a room at Boston Fintech Week and listened to a keynote
speaker get asked what the focal point would be when everyone reconvened the
following year. He said: AI agents.

I remember writing it down in my notes. That was the entire extent of my response. I carelessly noted
it and didn't think much of it, and now agents are what everyone is talking
about and I'm late to the party; all because I heard something and mindlessly
wrote it down with no action plan.

I'd been around AI for a while by then. Computer vision projects, machine
learning coursework, using the tools like everyone else. For some reason I
never went into agents the way I should have.

So this summer, as an intern at one of Deloitte's technology arms, I built one. A local, permissioned autonomous agent, from
nothing, over about two months.

## I didn't know what a harness was

That is the honest starting point. I had just heard of the word but what it actually meant?
I had no idea.

I had heard of the LangGraph family without really knowing what it was. I knew
containers existed as a concept, via Docker, but had never used one. I
understood the Model Context Protocol well enough to explain it and had never
applied it to anything real.

A harness, as I now finally understand, is everything around the model. The model is one
component and arguably the least interesting one. The harness is what decides
which tool gets called and what happens when that tool fails. It also determined what the agent is
allowed to do without asking, where generated code is permitted to run, and what 
happens when the model produces something confident and wrong.

All of this was news to me because before this internship, I'd just look at benchmarks. This year alone,
people went crazy when Gemini 3 came out and within a few days, OpenAI had clapped back with GPT-5.2. Then we
saw Anthropic take the lead for months with reports of them not giving the average Joe the best models but they 
quickly changed their mind when the Chinese open-source models like Kimi K3 and Qwen 3.8 (which can run locally emerged).
All of this does matter and I won't stop keeping up with these updates. But for someone like me, I'm
not planning on creating my own LLM so I'll just use the best frontier model that's cost-efficient at the same
time. Long story short, I was focused on the wrong part of the AI stack.

## The principle I'd defend

The single idea that shaped everything else in my agent:

**The LLM writes text. Deterministic code makes the consequential decisions.**

The model drafts the plan. It does not pick the tool. Tool selection runs
through code I can read, test, and reproduce.

The thing is, a model is probabilistic. When it picks the wrong tool, every node
downstream trusts that pick and builds on it, and what comes out the other end
is a confident hallucination. Unlike runnable code where I can get an error message,
I get absolute confidence and very unsafe behavior from the AI.


The worst version of this in my system: someone says "send the email" and the
router quietly selects "search emails." It looks like it worked. Nothing
throws. The email was never sent.

So I built a deterministic guard that classifies write intent separately from
the model, and vetoes a read-tool selection when the user clearly asked to
write. Not because it's elegant. Because that failure is silent, and silent
failures are the ones that hurt.

## "Regex is a bandaid"

My tool routing started as a large regex layer which consisted of roughly 1,500 lines turning
phrasings into tool calls. It worked... kind of. I thought the project would end there
and confidently told my boss I was basically done, but little did I know that this was just the beginning.

My manager told me regex was a bandaid and that it wasn't scalable. So it was back to the drawing board.

He wasn't wrong, and my first instinct was to argue. What I did instead was
go find out what frontier labs actually do: trained function-calling,
structured outputs, and evaluation harnesses and then build that version on
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

The cascade router I replaced it with a local model first, a second model
behind it, regex only as a tiebreak. This cascade router scored **89.9%** on the same sealed set.
More importantly it got read-versus-write right **100%** of the time, against
19 dangerous misroutes from the regex.

Worth being precise about what that does and doesn't mean: 89.9% is against
*my* set of roughly 45 tools, using a 7B model running locally on my machine.
A frontier model would score higher. That's rather the point my manager was trying to put forward 
the whole time. The interesting result isn't that it's state of the art, it's that a small local model plus a
carefully built harness was good enough to beat the thing I'd hand-tuned for
weeks.

The harness did more work than the model did.

## The uncomfortable part

Here is the thing I keep circling.

The same way AI is a loop of processes, the use of AI is a circle for a
consumer like me as well. I write code with AI. I ask questions about my code
to AI. I brainstorm with AI. And then I sat down to build an agent and a harness
whose entire purpose is deciding when to trust a model's output... using a
model to help me build it. 

Am I validating my work with AI, or walking into a loop of bias I should
be avoiding? Is this inherently wrong? I don't think so. But it does
dial in on an important matter and it's that my judgement is more important than ever.
I can now build much more impressive things and much faster due to AI, but with that comes
responsibility to make the right decision to scale a product/solution safely and efficently. 
And yes, AI can help with that too but if you're blindly following it, you're the one at blame
in the end. And I think I've seen enough stories about companies in the consulting industry
have multi-million dollar cases against them because of just that.


When I asked AI whether my routing approach was sound, I got agreement. Useful
agreement, even. But agreement is what that interaction produces, and I can't
tell the difference between "this is correct" and "this is plausible" from
inside the conversation.

In my project, I saw this when the sealed hold-out told me I was wrong. 51.1% is not 
an opinion and it did not care how confident I was. My manager saying "regex is a bandaid"  was
outside the loop too and must be considered.

That's the rule I've landed on. Use AI to build, absolutely. But every claim
that matters needs a check that can independently return "no." A test that
fails. A hold-out you only open once. A person who'll tell you the thing you
don't want to hear. Without at least one of those, you're not validating
anything, you're just generating more confidence.

## Where I actually am

I'm in no way an expert in any of this. If anything, building the thing showed
me how far I am from going from a novice to a master. I now know what a harness
is, which mostly means I know how much of it I haven't built yet. 

And as a final note, the biggest lesson: AI is only as good as you are. Coding agents
are insanely good not but after trying to build my own harness and seeing how stupid the LLM
can be was a wake up call that if you just depend on the model's intelligece and whatever frontier
model is going to be released next Tuesday, you're just falling behind.

---

The agent is open source: **[langgraph-code-agent](https://github.com/mohammadeissa/langgraph-code-agent)**
— the cascade router, the evaluation harness, and the methodology write-up with
the full per-service breakdown are all in there.


