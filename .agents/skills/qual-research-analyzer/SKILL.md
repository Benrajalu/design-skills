---
name: qual-research-analyzer
description: |
  Analyzes qualitative user research transcripts and moderator observation notes
  into evidence-based findings with strict anonymization, verbatim rigor, and
  gated sanity checks. Use for BlaBlaCar qualitative research, interview
  transcripts, GTM asset feedback, landing page comprehension, or interface
  usability testing with FR or BR participants.
---

# Qual Research Analyzer

Use this skill to analyze qualitative research artifacts with disciplined
evidence handling, explicit validation checkpoints, and a clear split between
GTM asset feedback and product usability findings.

This skill is for analysis of existing qualitative material, not for writing a
research plan or designing a discussion guide.

## When to use this skill

- Analyze interview transcripts with moderator notes
- Review GTM assets such as emails or landing pages for comprehension and appeal
- Synthesize interface usability sessions such as a one-screen listing flow
- Map observations to explicit research questions and hypotheses
- Produce decision-ready qualitative reports for BlaBlaCar stakeholders

## Non-negotiable analysis rules

1. Do not use numeric percentages anywhere in qualitative analysis. Replace
   them with qualitative frequency labels such as a majority, several
   participants, a few participants, nearly half, isolated cases, or an
   outlier.
2. Ground every insight in direct evidence. Keep verbatims in their original
   language and add an immediate English translation in square brackets.
3. Use each verbatim quote no more than once across the entire output.
4. Integrate moderator observations with spoken evidence. Include hesitations,
   confusion, pauses, misclicks, body language, and non-verbal signals when
   they materially support interpretation.
5. Keep GTM asset feedback separate from interface usability findings.
6. Remove all personally identifiable information and replace identities with
   participant IDs such as `P01 - FR Member`.
7. Separate direct observation from interpretation. Do not jump from a single
   quote to a generalized recommendation without an evidenced pattern.
8. Cite every verbatim with its participant ID and the exact transcript
   timestamp (`HH:MM`). Keep the sentence immediately before and after each
   extracted verbatim in the evidence register so it can be surfaced on
   request for a sanity check, even though it is not shown by default.

### Verbatim citation format

Use this format for every quote in extractions and in the final report:

`"verbatim" [English translation] - P01, 14:32`

If asked for context on a specific verbatim, return the sentence before and
after it from the evidence register, labeled `Before:` and `After:`, still
attached to the same participant ID and timestamp.

## Required inputs

Before starting analysis, confirm or request these inputs:

- Research context and objectives
- Research questions
- Hypotheses to validate
- Assets being tested
- Target audience or participant profiles
- Raw transcript plus moderator notes

## Clarify first

If one or more required inputs are missing, ask for only the missing items.
Use this exact checklist shape:

```markdown
Before I start the analysis, please provide the missing study setup items:

1. Research Context & Objectives
2. Research Questions (RQs)
3. Hypotheses to Validate (Hs)
4. Assets Being Tested
5. Target Audience / Profiles
6. Raw Transcript + Moderator Notes
```

If all six are already present, proceed directly to Step 1.

## Workflow

### Step 1 - Anonymization, metadata, and data cleaning

1. Scrub all PII from the transcript and notes.
2. Create participant IDs and extract metadata:
   - participant ID
   - market
   - segment or role
   - any relevant study grouping
3. Clean the transcript while preserving meaning.
4. Align transcript excerpts with the corresponding moderator observations.
5. Summarize the cleaned participant setup and context.

Mandatory stop after Step 1:

```markdown
Here is the cleaned profile setup and metadata. Do you validate this setup before I proceed to quote extraction?
```

Wait for user validation before continuing.

### Step 2 - Observation extraction and mapping

1. Extract the strongest verbatims and the supporting moderator observations.
2. Map each extraction to one or more research questions or hypotheses.
3. Organize all extractions into exactly two categories:
   - GTM Assets: email, landing page, message comprehension, appeal, and value proposition resonance
   - Usability & Interface Flow: interaction flow, task completion, hesitations, UI clarity, and misclicks
4. Preserve native-language quotes with an English translation in square brackets.
5. Tag each verbatim with its participant ID and exact timestamp (`HH:MM`),
   and keep the sentence before and after it in the evidence register for
   on-request sanity checks.
6. Keep a quote register to ensure no quote is reused later.

Mandatory stop after Step 2:

```markdown
Here are the raw extractions and translations organized by GTM Assets vs Usability. Are these extractions accurate and relevant before I cluster them into patterns?
```

Wait for user validation before continuing.

### Step 3 - Pattern identification and proof mapping

1. Cluster similar observations using affinity-style grouping.
2. Identify recurring behaviors, mental models, value drivers, and friction points.
3. For every pattern, show the exact supporting evidence:
   - participant IDs
   - verbatim quotes
   - moderator observations
   - linked research questions or hypotheses
4. Keep contradictory evidence visible instead of smoothing it away.

Mandatory stop after Step 3:

```markdown
Here are the drafted key patterns and the verbatim evidence supporting each. Do you agree with these thematic groupings, or should any patterns be refined before generating the final report?
```

Wait for user validation before continuing.

### Step 4 - Final synthesis generation

Once Step 3 is approved, generate the final structured report using the format
below.

## Output requirements

- Do not use percentages.
- Do not repeat the same quote across sections.
- Keep participant IDs attached to evidence.
- Distinguish direct quotes from moderator observations.
- Explicitly separate GTM asset findings from interface usability findings.
- Flag contradictions, weak signals, and uncertainties instead of forcing closure.

## Final output format

```markdown
PARTICIPANT PROFILE
- Participant ID: P01 - FR
- Segment / Role: Frequent Driver
- Moderator Context: Key behavioral notes

GTM ASSET FEEDBACK (Email & Landing Page)
- Comprehension & Value Proposition: How clearly the concept was understood
- Appeal & Resonance: What resonated positively or created hesitation
- Supporting Evidence: Native quotes [English translation] + moderator observations

INTERFACE USABILITY (Listing Screen)
- Task Success & Mental Model: How easily the participant completed or navigated the flow
- Friction Points & Hesitations: UI elements that caused confusion, misclicks, or pauses
- Supporting Evidence: Native quotes [English translation] + moderator observations

KEY PATTERNS & THEMES
- Theme Name: Clear and specific
- Pattern Narrative: Detailed description of the behavior or attitude
- Evidenced Proof: At least 2 supporting verbatims, each cited as `"verbatim" [translation] - P01, 14:32`
- Qualitative Frequency: Use labels such as a majority, several participants, a few participants, isolated cases

CONTRADICTIONS & FRICTION
- Conflicting statements, resistance, or gaps between what users said and what they did

HIDDEN TRENDS & WEAK SIGNALS
- Unexpected behavioral nuances, non-verbal signals, or early indicators

UNCERTAINTIES & FOLLOW-UPS
- Ambiguities, unresolved intent, and recommended next research steps
```

## Suggested working method

1. First build a participant table with anonymized metadata.
2. Then create an extraction log with one row per quote or observation,
   including participant ID, timestamp, and the sentence before/after.
3. Only after validation, cluster the evidence into patterns.
4. Write the synthesis from the patterns, not from memory.
5. Preserve disconfirming evidence and minority reactions.

## Anti-patterns to avoid

- Turning qualitative findings into pseudo-quantitative findings with percentages
- Reusing the same quote in several sections
- Mixing GTM comprehension issues with product usability issues
- Dropping the native-language quote
- Ignoring moderator notes when the behavior contradicts the spoken statement
- Reporting themes without showing the supporting proof
- Leaving PII in transcripts or examples
- Citing a verbatim without its participant ID and timestamp

## Related skills

- `research-summarizer` for broader synthesis and stakeholder packaging