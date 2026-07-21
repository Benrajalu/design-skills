---
name: mobbin-bench
description: |
  Builds a competitive UX pattern-review doc in Figma for any product experience — a screen, flow, or feature. Pulls examples from Mobbin and generates a fixed format: a "Key Patterns & Current Trends" legend with short P-codes, a row of established high-traffic examples, and a row of unconventional/unexpected examples — each screenshot tagged with pills and bullets tied to the P-codes. Use for pattern reviews, competitive audits, or Mobbin research docs at BlaBlaCar. 
---

Make sure the user gave you the two required parameters: [WHAT] (what the bench is about), and [LINK] (the link to the figma file to output to). If either is missing, ask the user to provide them before proceeding.

I'm designing [WHAT — e.g. a home page, an onboarding flow, a checkout experience, a settings panel]. Research this on Mobbin and produce a competitive pattern-review doc using this exact structure. The structure is fixed; the content, pattern count, and example count adapt to what the experience actually warrants.
Use the Mobbin and Figma MCPs to interact with either.

1. Title + summary line
[What] — Mobbin Pattern Review
One line stating how many established vs. unconventional examples are included.

2. "N Key Patterns & Current Trends" box
Derive as many patterns as the experience genuinely supports (typically 4–8) — don't force a fixed count, and don't reuse patterns from other reviews without re-verifying they apply here
Assign each a short code: P1, P2, P3…
Add a "Screenshot Markup Legend": each code + one-line description, in order discovered

3. "N Strong Established Examples"
Pick as many well-known, high-traffic products as meaningfully represent the dominant patterns for this experience (typically 4–6). One card per example, same layout every time:

Product name + "Established" label
2–3 colored pill tags = the P-codes this screen demonstrates
Representative screenshot of the screen/flow in question
2–3 short bullets (<12 words each), each explicitly referencing a P-code
Direct Mobbin reference link

4. "N Unconventional / Unexpected Examples"
Include as many examples as genuinely deviate from the dominant pattern (typically 2–3) — don't pad this out if fewer exist. Same card format, labeled "Unexpected" + a "UX shift" tag. Bullets must name the standard pattern being broken and why it's a deliberate deviation, not a weaker execution.

5. Footer
One line: Mobbin path/collection this was sourced from.
Hard rules for every run:

Every bullet must map to a P-code defined in the legend — no orphan observations
Pill colors are consistent per P-code across all cards in a single doc
Card layout (name → label → pills → screenshot → bullets → link) never changes order
Proceed automatically — don't pause to confirm the derived P-codes before building the Figma doc; just build it and I'll give feedback after

Output destination: Build into Figma file [LINK], mirroring the structure as frames (trends box frame → established-examples row → unconventional-examples row → footer). Consistent card sizing across all cards in the doc.
Before moving on to the report on Figma, makes sure you have access to the screenshot images from Mobbin. Images are essential to this task, do not settle for URLs.