# Content Page Template

The Content page provides writing guidelines for the text that goes inside the component. It's the UX writing reference.

## Structure

```markdown
## Writing guidelines

{Introductory sentence about the component's content purpose.}

### Tone

- {Bullet: Primary tone characteristic}
- {Bullet: Secondary tone characteristic}

### Length

- {Character or word limits, if any}
- {Guidance on brevity vs. detail}

### Structure

{If the component has multiple text slots (title, description, label), explain each:}

**{Slot name}**
- {Purpose of this text slot}
- {Do's for this slot}
- {Don'ts for this slot}

## Examples

### ✅ Good examples

| Context | Text |
|---------|------|
| {Scenario} | {Example copy} |
| {Scenario} | {Example copy} |

### ❌ Avoid

| Instead of... | Try... | Why? |
|---------------|--------|------|
| {Bad example} | {Better alternative} | {Brief explanation} |
```

## Content Guidelines

### Writing guidelines intro

One sentence framing the content challenge. What makes writing for this component different?

**Example (Nudge):**
> Nudge content should be brief and attention-grabbing: think headline, not paragraph.

### Tone

What emotional register should the text have? This varies by component:

- **Informational components** (Nudge, Tag): Neutral, factual, concise
- **Warning components** (Alert, Banner): Clear, calm, actionable
- **Success components** (Toast, Confirmation): Positive, brief, reassuring
- **Error components** (Validation, Error state): Helpful, specific, non-blaming

**Example (Nudge):**
- Keep it punchy: Nudge competes for attention, so every word must earn its place.
- Avoid urgency unless using the Warning variant. Default and Strong should inform, not alarm.

### Length

Provide concrete guidance:
- Character limits (if enforced by the component)
- Word count recommendations (if not enforced but important for design)
- Truncation behavior (what happens if content is too long?)

**Example (Nudge):**
- Title: 2-4 words ideal, 6 words maximum before it feels like a sentence.
- Data slot: Numbers or single words only (e.g., "3 left", "New", "12%").

### Structure (for multi-slot components)

If the component has multiple text areas, explain each:

1. **Slot name** — Match the Figma property name
2. **Purpose** — What information goes here?
3. **Do's** — Specific guidance
4. **Don'ts** — Common mistakes

### Examples

**Good examples** should show real, usable copy — not lorem ipsum. Provide context so designers understand when each example applies.

**Avoid examples** must include:
1. The problematic text
2. A better alternative
3. A brief explanation of why the first version is worse

Keep explanations to one sentence. If it takes a paragraph to explain why something is bad, it's probably not a clear-cut anti-pattern.

## Localization Note (Optional)

If relevant, add guidance for international content:

```markdown
## Localization

- {Guidance for translated content}
- {Languages that may need more/less space}
- {Cultural considerations, if any}
```
