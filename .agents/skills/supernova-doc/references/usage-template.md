# Usage Page Template

The Usage page tells designers *when* to use the component and *how* to choose between variants. It's the decision-making guide.

## Structure

```markdown
## When to use?

- {Bullet: Primary trigger — what user need or context calls for this component}
- {Bullet: Secondary consideration — another valid use case}

### {Variant 1 name}

![{Variant screenshot}](placeholder-{variant-slug}-usage.png)

- {When to choose this variant}
- {Real product example, if available}

### {Variant 2 name}

![{Variant screenshot}](placeholder-{variant-slug}-usage.png)

- {When to choose this variant}
- {Real product example, if available}

{Repeat for each variant...}

## Minimal content rules

![{Screenshot showing minimum required content}](placeholder-minimal.png)

{One sentence explaining what content is required vs. optional.}
```

## Content Guidelines

### When to use?

Start with the highest-level guidance — when should a designer even consider this component?

**Structure:**
1. **Primary trigger**: The main user need this component serves
2. **Distinguishing factor**: What makes this the right choice over alternatives

**Example (Nudge):**
- When you need to draw attention to short text content, in a block. If your need is more "inline", use a Tag.
- When your eye-catching content can be clickable.

### Variant guidance

For each variant, explain:
1. **The decision criteria** — What condition or context calls for this variant?
2. **A real example** — Where is this variant used in the product? (optional but highly valuable)

**Avoid:**
- Describing visual differences (that's what the image is for)
- Repeating information from the Overview page
- Generic guidance like "use for important content" without specifics

**Example (Nudge/Warning):**
- Nudge/Warning signals caution: something negative might happen, but urgency is moderate.
- Used for scarcity indication on the Ride Details page.

### Minimal content rules

This section answers: "What's the bare minimum I need to provide?"

**Include:**
- A screenshot showing the component with only required content
- One sentence stating what is required vs. optional

**Example (Nudge):**
> Nudge should at least have a title. The data slot and icon are optional.

## Do/Don't Section (Optional)

If there are common misuses, add a Do/Don't section:

```markdown
## Do's and Don'ts

### ✅ Do

- {Good practice with brief explanation}
- {Another good practice}

### ❌ Don't

- {Anti-pattern with brief explanation why it's problematic}
- {Another anti-pattern}
```

Only include this if there are genuine anti-patterns to warn against. Don't invent problems.
