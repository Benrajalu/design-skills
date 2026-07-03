# Accessibility Page Template

The Accessibility page explains how the component works for users of assistive technologies. Unlike Figma's implementation-focused accessibility spec, this page is written for designers who need to understand the user experience.

## Structure

```markdown
## Screen reader experience

{Brief intro: How do screen reader users perceive and interact with this component?}

### Announcement

When a screen reader user encounters {Component name}, they hear:

> "{Example announcement text}"

{Explanation of what's announced and in what order.}

### Interaction

{If interactive:}
- **Focus**: {How does the component receive focus?}
- **Activation**: {What happens when activated? What keys work?}
- **Feedback**: {What does the user hear after interaction?}

{If non-interactive:}
> {Component name} is not interactive. Screen reader users navigate past it as they would static text.

## Keyboard navigation

{If interactive:}

| Key | Action |
|-----|--------|
| Tab | {What happens} |
| Enter / Space | {What happens} |
| Escape | {What happens, if applicable} |

{If non-interactive:}
> {Component name} is not focusable and does not require keyboard interaction.

## Design considerations

- {Bullet: Key a11y consideration for designers}
- {Bullet: Another consideration}

### Color and contrast

- {Guidance on color reliance}
- {Contrast requirements, if relevant}

### Motion

{If the component animates:}
- {Motion considerations}
- {Reduced motion behavior}

{If no animation:}
> {Component name} does not use motion.
```

## Content Guidelines

### Screen reader experience

Write this for designers who don't use screen readers daily. Help them understand:
1. **What users hear** — The actual announcement, in quotes
2. **What users can do** — Available interactions
3. **What users expect** — Mental model for this component type

**Example (Nudge):**
> When a screen reader user encounters a Nudge, they hear the content read as a single text block. If the Nudge is clickable, it's announced as a button.
>
> **Clickable Nudge:** "3 seats left, button"
> **Non-clickable Nudge:** "3 seats left"

### Interaction (for interactive components)

Explain the interaction model in plain language:
- **Focus**: How does it get focus? Is it in the tab order?
- **Activation**: What triggers the action? (tap, Enter, Space)
- **Feedback**: What confirmation does the user receive?

**Example (Clickable Nudge):**
- **Focus**: Nudge receives focus in the normal tab order.
- **Activation**: Press Enter or Space to activate (same as tapping).
- **Feedback**: Navigates to the destination, same as any button.

### Keyboard navigation

Use a table for quick reference. Only include keys that actually do something.

Common patterns:
| Key | Typical action |
|-----|----------------|
| Tab | Move focus to/from the component |
| Shift+Tab | Move focus backward |
| Enter | Activate buttons, submit forms |
| Space | Activate buttons, toggle checkboxes |
| Escape | Close modals, dismiss popovers |
| Arrow keys | Navigate within composite widgets |

### Design considerations

Highlight decisions designers control that affect accessibility:

1. **Content clarity**: Is the text understandable without visual context?
2. **Color independence**: Is meaning conveyed without relying solely on color?
3. **Focus visibility**: Is the focus state clearly visible?
4. **Touch target**: Is the interactive area large enough? (44×44pt minimum)

**Example (Nudge):**
- Don't rely on variant color alone to convey meaning: a Warning Nudge should have warning *content*, not just a warning color.
- Clickable Nudges must have a visible focus state. Verify this is visible on your background color.

### Color and contrast

**For informational components:**
> Text must meet WCAG AA contrast (4.5:1 for body text, 3:1 for large text). The component's built-in colors are designed to pass; don't override them.

**For status/semantic components:**
> Don't rely on color alone. Warning Nudge uses yellow, but the *content* should also indicate warning (e.g., "Only 3 left" vs. just "3 left").

### Motion

Most components don't animate. For those that do:
- Does animation convey meaning? If so, provide a non-animated alternative.
- Does the component respect `prefers-reduced-motion`?

## Platform-Specific Details (Optional)

If designers need to know about platform differences:

```markdown
## Platform notes

### iOS (VoiceOver)
- {iOS-specific behavior}

### Android (TalkBack)
- {Android-specific behavior}

### Web (Screen readers)
- {Web-specific behavior}
```

Only include this section if there are meaningful differences that affect design decisions. Implementation details belong in the Code page, not here.
