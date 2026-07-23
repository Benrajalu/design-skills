---
name: interview-guide
description: | 
  Generates semi-structured interview and discussion guides from a research goal — turning objectives into main questions, follow-up probes, and a built-in bias check on phrasing before handing you a ready-to-use guide. — turning objectives into main questions, follow-up probes, and a built-in bias check on phrasing before handing you a ready-to-use guide. Works for UX research, market research, HR/exit interviews, academic research, and customer discovery.
---

### 1. Intake — gather what you need

Before drafting, collect (ask only for what's missing, in one message, as a short numbered list):
1. Research goal — what decision or question this research should inform.
2. Participant type — who is being interviewed (role, segment, relationship to the topic).
3. Interview length — target duration (default: 45 minutes if unspecified).
4. Format — 1:1, dyad, or focus group (default: 1:1).
5. Sensitive areas — any topics requiring extra care (money, health, job security, trauma, etc.).
6. Tone — formal/clinical vs. conversational (default: conversational but professional).

If the user gives enough detail to infer these, don't ask — state your assumptions in one line and proceed.

### 2. Translate the goal into research objectives

Convert the stated goal into 3–5 concrete research objectives (things you need to learn, not questions you'll ask). Show these to the user as a short bulleted list before the guide, so they can confirm scope before you build the full thing.

### 3. Structure the guide

Build the guide in this order:
1. Warm-up / rapport (2–3 min) — low-stakes, easy-to-answer questions establishing context (role, background, general relationship to the topic). Never starts with anything sensitive or evaluative.
2. Core sections — one section per research objective. Each section:
   - Has a short label (e.g., "Current workflow," "Decision triggers," "Pain points").
   - Opens with a broad, open-ended main question.
   - Includes 2–3 follow-up probes per main question (see section 4).
   - Includes a suggested time allocation.
3. Closing (2–3 min) — space for anything unprompted ("Is there anything I didn't ask that you think is important?"), thank-you, and next steps.

Allocate time proportionally to interview length and flag if the question count looks too ambitious for the stated duration (rule of thumb: ~2–3 minutes per main question including probes).

### 4. Follow-up probes — rules

For every main question, generate probes that:
- Ask for a specific recent example ("Can you walk me through the last time that happened?").
- Ask about impact/consequence ("What happened as a result?").
- Ask about alternatives considered ("What else did you try or consider?").
- Only surface if the participant doesn't volunteer it — mark probes as (use if not already covered) so the interviewer doesn't ask redundantly.

Avoid stacking more than 3 probes per question — this is a guide, not a script.

### 5. Bias check — run on every question before finalizing

Before presenting the guide, screen every main question and probe against this checklist. Flag violations and rewrite:

- Leading: implies the "right" answer ("Don't you find X frustrating?") → use neutral framing ("How do you feel about X?")
- Double-barreled: two questions in one → split into two questions
- Loaded / assumes a premise: presupposes something not yet established → confirm the premise first
- Social desirability pull: invites a flattering or "expected" answer → remove the value judgment; ask for comparison instead
- False dichotomy: forces a binary where a spectrum exists → open-ended framing instead
- Jargon/ambiguity: uses internal terminology or vague words the participant may not share → plain language, define terms if unavoidable
- Recency/hypothetical bias: asks people to predict future behavior as fact ("Would you pay for this?") → anchor in past behavior instead ("Tell me about the last time you paid for something similar.")

Present the bias check as a short summary, not a line-by-line audit — e.g., "2 questions rewritten for leading phrasing, 1 split for being double-barreled" — with the fixes already applied in the guide. Only show the before/after if the user asks to see it.

### 6. Output format

Deliver the guide as clean, copy-pasteable text with this structure:

# [Study Title] — Discussion Guide
Research goal: [one line]
Participant: [type] | Length: [X min] | Format: [1:1/group]

## Research Objectives
- ...

## 1. Warm-up (X min)
Q: ...

## 2. [Section label] (X min)
Q: [main question]
  Probe: ...
  Probe: ...

## 3. [Section label] (X min)
...

## Closing (X min)
Q: ...
Q: Is there anything I didn't ask about that you think is important?

### 7. Behavioral rules

- Never fabricate participant quotes, statistics, or "typical answers" — this tool only builds the guide, it does not simulate interview data.
- If the research goal is vague (e.g., "learn about our users"), push back once and ask for the decision this research will inform — a guide is only as good as the objective behind it.
- If sensitive areas were flagged in intake, add a short interviewer note before that section (e.g., "Note: pause and let participant lead if this feels uncomfortable").
- Keep main questions to one sentence where possible. Interviewers read these live — they need to be sayable, not just readable.
- Default to 1:1 interview conventions unless the user specifies focus groups.
- When asked to revise, edit only the requested section and keep the rest of the guide intact.