---
name: Thai Law
description: Mobile-first Thai law study app for lookup, practice, and retention.
colors:
  court-blue: "#3b82f6"
  court-blue-deep: "#2563eb"
  statute-teal: "#14b8a6"
  statute-teal-deep: "#0d9488"
  quiz-indigo: "#6366f1"
  quiz-indigo-deep: "#4f46e5"
  success-green: "#10b981"
  warning-amber: "#f59e0b"
  danger-red: "#ef4444"
  neutral-ink: "#1f2937"
  neutral-body: "#4b5563"
  neutral-muted: "#6b7280"
  neutral-border: "#e5e7eb"
  neutral-surface: "#f9fafb"
  neutral-canvas: "#f3f4f6"
  neutral-white: "#ffffff"
typography:
  display:
    fontFamily: "-apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif"
    fontSize: "2.5rem"
    fontWeight: 700
    lineHeight: 1.2
    letterSpacing: "normal"
  headline:
    fontFamily: "-apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif"
    fontSize: "1.875rem"
    fontWeight: 700
    lineHeight: 1.25
    letterSpacing: "normal"
  title:
    fontFamily: "-apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif"
    fontSize: "1.5rem"
    fontWeight: 700
    lineHeight: 1.3
    letterSpacing: "normal"
  body:
    fontFamily: "-apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif"
    fontSize: "1rem"
    fontWeight: 400
    lineHeight: 1.6
    letterSpacing: "normal"
  label:
    fontFamily: "-apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif"
    fontSize: "0.875rem"
    fontWeight: 600
    lineHeight: 1.4
    letterSpacing: "0.1em"
rounded:
  xs: "4px"
  sm: "8px"
  md: "12px"
  lg: "16px"
  pill: "9999px"
spacing:
  xs: "4px"
  sm: "8px"
  md: "12px"
  lg: "16px"
  xl: "24px"
  xxl: "32px"
components:
  button-primary:
    backgroundColor: "{colors.quiz-indigo}"
    textColor: "{colors.neutral-white}"
    rounded: "{rounded.md}"
    padding: "16px 24px"
  button-primary-hover:
    backgroundColor: "{colors.quiz-indigo-deep}"
    textColor: "{colors.neutral-white}"
    rounded: "{rounded.md}"
    padding: "16px 24px"
  button-success:
    backgroundColor: "{colors.success-green}"
    textColor: "{colors.neutral-white}"
    rounded: "{rounded.md}"
    padding: "16px 24px"
  card-default:
    backgroundColor: "{colors.neutral-white}"
    textColor: "{colors.neutral-ink}"
    rounded: "{rounded.lg}"
    padding: "24px"
  card-interactive:
    backgroundColor: "{colors.neutral-surface}"
    textColor: "{colors.neutral-ink}"
    rounded: "{rounded.lg}"
    padding: "24px"
  input-default:
    backgroundColor: "{colors.neutral-surface}"
    textColor: "{colors.neutral-body}"
    rounded: "{rounded.sm}"
    padding: "12px 16px"
---

# Design System: Thai Law

## 1. Overview

**Creative North Star: "The Thai Law Study Console"**

Thai Law uses a calm, trustworthy, and task-forward interface where legal content remains the center of attention. Surfaces are clean and predictable, while accent gradients are reserved for actions and study milestones that need immediate recognition. The visual system should feel academically grounded, not cold, and approachable, not playful.

This is a mobile-first study product, so controls are touch-sized, hierarchy is explicit, and text blocks prioritize Thai readability first. Visual energy is delivered through selective color commitment and state feedback, not through decorative complexity.

The system explicitly rejects neon presentation and generic AI-looking template aesthetics. It also rejects visual noise that weakens legal credibility.

**Key Characteristics:**
- Trust-first neutral structure with restrained accent usage.
- Strong tap targets and card-based information grouping for mobile study flow.
- Clear bilingual hierarchy with Thai content leading and English support secondary.
- Gradient usage tied to state and action, never as ambient decoration.

## 2. Colors

The palette balances legal-reference neutrality with focused accent channels for progress, action, and correctness states.

### Primary
- **Court Blue** (#3b82f6): Header identity and active navigation emphasis for default study flows.
- **Court Blue Deep** (#2563eb): Hover and pressed state for primary blue controls.
- **Statute Teal** (#14b8a6): Flashcard front surfaces and content-learning emphasis.
- **Statute Teal Deep** (#0d9488): Depth anchor for teal gradients and interactive reinforcement.

### Secondary
- **Quiz Indigo** (#6366f1): Primary quiz action surfaces and progress-driven emphasis.
- **Quiz Indigo Deep** (#4f46e5): Elevated quiz CTA and hover depth.

### Tertiary
- **Success Green** (#10b981): Correctness confirmation and completion actions.
- **Warning Amber** (#f59e0b): Caution messages, fallback data notices, and medium-risk status.
- **Danger Red** (#ef4444): Error states and incorrect-answer feedback.

### Neutral
- **Legal Ink** (#1f2937): High-contrast headings and primary text.
- **Reference Body** (#4b5563): Standard body and support text.
- **Muted Guidance** (#6b7280): Secondary descriptions and subdued metadata.
- **Line Divider** (#e5e7eb): Borders, card outlines, and low-emphasis separators.
- **Paper Surface** (#f9fafb): Elevated neutral card backgrounds.
- **Canvas Base** (#f3f4f6): App-level background transitions.
- **White Surface** (#ffffff): Foreground card and dialog base.

### Named Rules
**The One-Accent-Per-Surface Rule.** Any single screen region should emphasize one dominant accent family at a time. Blue, teal, and indigo should not compete in equal weight.

**The Contrast-First Rule.** Body text must remain at or above AA contrast against its immediate background. If contrast is borderline, darken text before adjusting saturation.

## 3. Typography

**Display Font:** System Sans Stack (-apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen, Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif)
**Body Font:** System Sans Stack (-apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen, Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif)
**Label/Mono Font:** System Sans Stack (same family for consistency)

**Character:** Clear and direct. The type system favors readability and stable rhythm over stylistic contrast.

### Hierarchy
- **Display** (700, 2.5rem, 1.2): Welcome headers and key section titles that establish context quickly.
- **Headline** (700, 1.875rem, 1.25): Primary screen headings such as app title and main section headers.
- **Title** (700, 1.5rem, 1.3): Card titles, feature labels, and item headers.
- **Body** (400, 1rem, 1.6): Thai and English reading content, descriptions, and answer text.
- **Label** (600, 0.875rem, 0.1em letter spacing): Hints, compact badges, and compact control labels.

### Named Rules
**The Thai-First Legibility Rule.** Thai reading blocks should never drop below 1rem at default viewport widths, and line-height should remain at 1.6 or greater for continuous text.

**The Stable Scale Rule.** Product UI uses fixed rem steps, not fluid display scaling, to keep hierarchy predictable across dense study screens.

## 4. Elevation

Thai Law uses a hybrid depth model: flat neutral layers for default reading, then soft shadow lift for interactive cards, action buttons, and flashcards. Depth is functional and state-driven, never ornamental.

### Shadow Vocabulary
- **Interactive Lift** (`box-shadow: 0 4px 12px rgba(59, 130, 246, 0.15)`): Hover feedback for selectable cards.
- **Flashcard Depth** (`box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2)`): Dimensional separation for flip-card interaction.
- **Modal/Result Focus** (`box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1)`): Attention framing for result and summary panels.

### Named Rules
**The Flat-at-Rest Rule.** Static containers remain visually calm. Elevation appears when an element becomes interactive, selected, or state-critical.

## 5. Components

Components should feel confident and direct, with clear affordances and strong state readability.

### Buttons
- **Shape:** Rounded rectangular with medium radius (12px) for primary CTAs; pill form (9999px) for compact badges only.
- **Primary:** Indigo or green gradient-backed actions with white text and medium-strong weight.
- **Hover / Focus:** Slight upward movement and soft shadow amplification on hover, clear focus-visible ring for keyboard users.
- **Secondary / Ghost:** Neutral gray backgrounds with darker text for non-destructive actions.

### Chips
- **Style:** Rounded pills (16px-9999px) with low-noise background fills.
- **State:** Selected chips must use explicit border or tone change, not color shift alone.

### Cards / Containers
- **Corner Style:** Large rounded cards (16px) for menu and category surfaces.
- **Background:** White or neutral-surface base.
- **Shadow Strategy:** Minimal by default, elevated on hover/selection.
- **Border:** 1px-2px neutral border for separation.
- **Internal Padding:** 16px to 24px depending on density.

### Inputs / Fields
- **Style:** Neutral surface with visible border and compact radius (8px).
- **Focus:** Border-color shift to active accent with optional subtle shadow.
- **Error / Disabled:** Error uses red border plus readable foreground; disabled uses neutral-muted foreground on neutral-border background.

### Navigation
- **Header:** Blue gradient header with clear title/subtitle hierarchy and optional back button.
- **Main Menu Navigation:** Full-width tappable cards with icon, title, description, and directional arrow.
- **Mobile Treatment:** Maintain comfortable horizontal padding (12px minimum) and direct single-column flow.

### Signature Component
- **Flip Flashcard:** Two-sided gradient card with 3D flip transition and persistent tap hints. The component must remain readable in both states and never hide core text behind animation timing.

## 6. Do's and Don'ts

### Do:
- **Do** keep primary reading text on Legal Ink (#1f2937) or stronger against light surfaces.
- **Do** use 16px minimum tap-space padding on primary menu and answer controls.
- **Do** reserve gradient intensity for interaction and progress moments.
- **Do** keep Thai content first in hierarchy and maintain consistent bilingual pairing.
- **Do** keep card radii and spacing tokens consistent across views.

### Don't:
- **Don't** use neon color treatments or high-saturation glow effects that feel game-like.
- **Don't** introduce generic AI template patterns such as repetitive lookalike card grids with decorative gradients.
- **Don't** rely on left or right accent stripe borders above 1px for emphasis.
- **Don't** use gradient text for headings or labels.
- **Don't** reduce body contrast for aesthetic softness, legal-study screens require immediate legibility.
- **Don't** apply decorative motion that does not communicate state or interaction outcome.