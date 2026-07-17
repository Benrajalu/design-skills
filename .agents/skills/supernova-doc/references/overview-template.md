# Overview Page Template

The Overview page introduces the component and gives designers a quick understanding of what it is and when to reach for it.

## Structure

```markdown
## {Component name} basics

- {First bullet: primary use case — what problem it solves}
- {Second bullet: key characteristic or constraint}
- {Third bullet: relationship to similar components, if relevant}
- {Fourth bullet: interactive capability, if any}
- {Fifth bullet: variant summary — list the variants available}

{For each variant, include an image and one-line description:}

![{Variant name}](placeholder-{variant-slug}.png)

**{Variant name}**

{One sentence describing when to use this variant.}

## Quick links

- [Link to the component on Figma]({figma-url})
```

## Content Guidelines

### Basics section

The bulleted list should answer these questions in order:
1. **What is it for?** — The primary use case in one sentence
2. **What makes it distinct?** — A key characteristic (size, weight, interactivity)
3. **What is it NOT?** — How it differs from similar components (optional, include if there's a common confusion)
4. **What can it do?** — Interactive capabilities (clickable, expandable, etc.)
5. **What are the options?** — List of available variants

**Example (Nudge):**
- Use a Nudge when you need to highlight content with a lower visual weight than a PushInfo.
- Nudge is not meant for long content: think of it as a bigger brother to Tags.
- Nudge can accept data or a word, but that too is meant to be very short.
- Nudge can be clickable, and used to invite users to discover something new.
- Nudge has three variants: Default, Warning and Strong.

### Variant showcase

For each variant:
1. **Image**: Screenshot of the variant in its default state
2. **Name**: Bold, matching the Figma variant name
3. **Description**: One sentence explaining the use case

Keep descriptions focused on *when* to use, not *how* it looks. Visual differences are obvious from the image.

**Example (Nudge/Warning):**
> **Warning**
>
> Indicates caution: something negative might happen, but urgency is moderate.

### Quick links

Always include:
- Link to the Figma component (extract from the component-doc frame)

May also include (if available):
- Link to Storybook
- Link to related components
