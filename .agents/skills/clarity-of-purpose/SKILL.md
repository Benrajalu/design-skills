---
name: clarity-of-purpose
description: |
   Helps define, sharpen, and audit the purpose behind a product, feature, or design decision. Use this skill whenever the request involves defining user goals, figuring out who a feature is for, turning vague requirements into a clear goal statement, deciding between design/feature options with data instead of guesses, or sanity-checking whether something being built actually serves a real user need.
---
# Clarity of Purpose

A framework for building products people love by grounding every design and product
decision in a clear, shared understanding of who you're solving for, what they're
trying to achieve, and why — backed by data rather than assumptions.

Core idea: products succeed one interaction at a time when each interaction serves a
purpose the user actually has. Clarity of purpose is achieved through two
complementary practices:

1. User goals — a clear, shared, solution-agnostic statement of who the user is,
   what they want, and why.
2. Shape of data — using real usage data (or direct user input) to inform
   decisions instead of guessing.

Use this skill to help draft user goals, ladder them into sub-goals, generate the
right shape-of-data questions, or audit a feature/product idea against this
framework.

## When to use which part

- Starting something new, or the purpose feels fuzzy → Draft a user goal.
- Have a goal but it's too broad, too narrow, or secretly a solution → Laddering.
- Choosing between options and about to decide on a hunch → Shape of data.
- Want a gut-check on something already built or planned → Clarity audit.

Don't dump the whole framework on every request — identify which part is actually
needed and lead with that, then offer the rest if useful.

## 1. Drafting a user goal statement

A user goal has three ingredients. Get all three before writing the statement — if
one is missing, ask for it rather than inventing it:

- User — who specifically are you solving a problem for? Not "users" — a real
  segment ("commuters," "finance approvers," "first-time sellers").
- Goal — what are they trying to achieve? Must be solution-agnostic — a goal,
  not a feature. "A hassle-free way to get to work on time" is a goal. "A commute
  notification widget" is a solution wearing a goal costume — push back and ask "what
  would that actually let them do or feel?" until you reach the real goal underneath.
- Motivation — why do they need this? What's the payoff or pain avoided?

Format the statement exactly like this:

[User] need(s) [goal] so that [motivation].

Example:
Commuters need a hassle-free way to get to work on time so that they arrive relaxed
and ready to start their day.

Red flag that the "goal" slot secretly holds a solution: it names a UI element, a
feature, or a technology. If so, ask "why do they want that?" and use the answer as
the real goal.

## 2. Laddering goals into sub-goals

Goals exist at different altitudes. Use two questions to move between levels:

- "Why?" — moves up to a bigger, more fundamental goal (the "north star").
- "How?" — moves down to a sub-goal: what's needed in order to meet the bigger
  goal.

Example ladder:

User goal:  Commuters need a hassle-free way to get to work on time
            so that they arrive relaxed and ready to start their day.
                 ▲ Why?                    ▼ How?
Sub-goal:   Commuters need to know the best commuting option
            so that they can plan their trip to work.

If a goal feels too solution-y or too narrow, ladder it up with "Why?" until it's a
real human motivation. If it feels too abstract to act on, ladder it down with "How?"
until it's concrete enough to design against. Keep the same Who/Goal/Motivation
template at every rung.

Seek alignment: for a team, the deliverable is a goal (and sub-goals) the whole
product/design/eng group agrees is the north star — not just a solo guess. Flag when
a goal statement needs validation from the team or from real users rather than being
finalized alone.

## 3. Shape of data — data-informed decisions instead of assumptions

Before locking in a design or product decision, replace assumptions with questions
that can be answered with real usage data. Use this for decisions of any size, from a
single UI choice to a feature investment.

If building on top of, or replacing, an existing product, start with this baseline
question set (adapt to context — skip questions irrelevant to the decision at hand):

- How many users currently use it?
- How often do they use it?
- How much time do they spend using it?
- Where do they spend the most time?
- What tasks do they do, and how long does each take to complete?
- How satisfied are they? How do they feel about it?

For a specific decision (e.g. "which of these two layouts"), tailor the question set
to what would actually discriminate between the options — ask "what data, if you had
it, would settle this?" and generate questions from that.

If there's no access to usage data, say so plainly and suggest the fallback: talk
directly to users. The best way to find out what users are trying to achieve is to
ask them directly — treat this as a legitimate substitute, not a lesser one, when
telemetry isn't available.

## 4. Clarity-of-purpose audit

When asked to sanity-check an existing feature, product, or decision, walk it
against these questions and report back honestly, flagging gaps rather than smoothing
over them:

1. Can it be stated, in one Who/Goal/Motivation sentence, who this is for and why
   they need it? If not, that's the core gap — start at step 1.
2. Is the stated goal actually a goal, or is it a solution in disguise?
3. Is there team alignment on this goal, or is it one person's assumption?
4. Is the decision backed by usage data or direct user input, or by assumption? If
   assumption, list the shape-of-data questions that would de-risk it.
5. Does each component/interaction in the flow trace back to the stated goal? Flag
   anything that seems to exist for its own sake rather than serving the user's goal.

Keep the audit concise and actionable — a short list of gaps and concrete next steps,
not a lecture on the framework.

## Tone

This is a practical facilitation tool. Don't recite the philosophy at length — apply
it. Ask the sharp clarifying question ("who specifically?", "why do they want that,
really?", "what data would settle this?") rather than restating the framework's
definitions back at the person.