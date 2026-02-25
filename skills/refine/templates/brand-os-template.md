# Brand Operating System: {Product Name}

**Status:** {Draft | In Review | Approved}
**Owner:** {Name}
**Date:** {YYYY-MM-DD}
**Companion PRD:** {Link to PRD if exists, or "Standalone"}

---

## 1. Decision Principles

This section defines how the brand makes decisions. A functional Brand OS enables anyone to make a brand-consistent decision without checking with the founder.

### 1.1 Decision-Making Principles

{3-5 principles that guide brand decisions. Each should be specific enough to resolve an ambiguous situation. Format: "We [verb] [action] because [reason]." Avoid generic values like "quality" or "innovation" — state what makes THIS brand's decisions distinctive.}

> Example: "We choose clarity over cleverness in every interface because our users are making time-sensitive decisions." / "We default to restraint — every element must earn its place."

### 1.2 Value Priority Hierarchy

{When brand values conflict, which wins? List values in priority order with brief rationale. This resolves real tradeoffs — e.g., "Should we add personality to this error message or keep it purely functional?"}

> Example: 1. Clarity (never sacrifice understanding for style), 2. Warmth (approachable over authoritative), 3. Efficiency (respect the user's time over visual delight), 4. Personality (distinctive but never at the cost of the above)

### 1.3 Brand Litmus Tests

{3-5 yes/no questions the team can ask to determine if a decision is on-brand. These should be practical, not aspirational.}

> Example: "Would a calm, experienced mentor say this?" / "Does this respect the user's time?" / "Could we remove this and still communicate effectively?"

---

## 2. Core Identity

The strategic foundation. Every visual and verbal decision traces back to this section.

### 2.1 Mission & Positioning

{State the brand's mission in one sentence. Then state the positioning: what category does this product occupy, what makes it different, who is the target user, and what is the emotional promise.}

> Example: "Cadence helps people maintain meaningful relationships through gentle, intelligent reminders. Positioned as a personal relationship coach — not a task manager, not a social network. For thoughtful people who value their connections but struggle with consistency."

### 2.2 Brand Archetype

{Choose and describe the brand archetype as a decision shortcut — not mystical, but a practical tool for resolving "would we do this?" questions. Describe in 2-3 sentences with specific behavioral implications.}

> Example: "The Sage — calm, knowledgeable, never preachy. Cadence knows when to speak and when to stay quiet. It offers guidance without pressure, insight without judgment."

### 2.3 Emotional Tone

{Define the emotional register. What should users FEEL when interacting with the product? List 3-5 emotional attributes with brief descriptions of how each manifests in the product.}

> Example: Calm (no urgency, no anxiety-inducing notifications), Warm (personal, not corporate), Confident (opinionated defaults, not endless options), Encouraging (celebrates consistency, never shames gaps)

### 2.4 Accessibility as Brand Principle

{State the brand's accessibility commitment as a design value, not just compliance. Specify minimum standards (e.g., WCAG 2.1 AA) and how brand decisions are validated against accessibility requirements.}

> Example: "Accessibility is not an afterthought — it's a design constraint we embrace. Every color choice, font size, and interaction pattern must meet WCAG 2.1 AA. We never choose aesthetics over usability."

### 2.5 Brand Architecture & Naming

{Define naming conventions for the product, its features, and any sub-brands. How does the brand name relate to feature names? What naming patterns are followed and what should be avoided?}

> Example: "Product name: Cadence. Feature naming: verb-first, human language (e.g., 'Check In', 'Catch Up', not 'Contact Sync'). Avoid technical jargon in user-facing names. Sub-features do not carry the brand name — they are unnamed capabilities."

---

## 3. Visual Identity

The expression system. These decisions produce the visual language builders implement.

### 3.1 Logo System

{Describe the logo system: primary logo, wordmark, icon-only mark, and variants. Specify usage rules — when to use which variant, minimum sizes, clear space, and what to avoid. If logos do not yet exist, describe the intended direction and constraints.}

### 3.2 Color System

{Define the color hierarchy with roles, not just hex values. Specify: primary, secondary, accent, neutral scale, and semantic colors (success/warning/error/info). For each, explain the role and usage context. Include light mode and dark mode considerations and contrast requirements.}

> Example:
> - Primary: Warm coral — primary CTAs and key brand moments. Conveys energy and anticipation.
> - Surface: Deep charcoal — OLED-friendly dark base. Premium without corporate.
> - Accent: Soft amber — secondary highlights, progress indicators. Warmth without urgency.
> - Neutral: 5-step gray scale. Text, borders, disabled states.
> - Semantic: Green/Amber/Red/Blue for standard status meanings. No brand color overrides for semantic use.

### 3.3 Typography System

{Define the type hierarchy: primary typeface (brand personality), secondary typeface (UI readability), and special-purpose faces if needed (numeric, code). Specify the scale (headline, subhead, body, caption, label) with relative sizing. Note platform compatibility and performance considerations.}

> Example: "Primary: SF Pro Display for headlines — system font, excellent rendering. Body: SF Pro Text for small sizes. Scale: Title (28pt), Headline (22pt), Body (17pt), Caption (13pt), Label (11pt)."

### 3.4 Layout & Spacing

{Define the spatial system: grid, spacing scale, corner radii, shadows, and UI density. Specify the base unit and how it scales. These rules separate polished UI from amateur.}

> Example: "8pt grid system. Spacing scale: 4, 8, 12, 16, 24, 32, 48, 64. Corner radii: small (8pt), medium (12pt), large (16pt), pill (full). Shadows: subtle only — elevation through layering and color, not heavy shadows."

### 3.5 Iconography

{Define the icon style: stroke weight, corner treatment, fill vs outline, size grid, and consistency rules. The icon style should match the brand's emotional tone from Section 2.3.}

> Example: "SF Symbols as primary icon set. Weight: regular (matching body text). Style: outline for navigation, filled for selected/active states. Custom icons follow SF Symbols grid and stroke conventions."

### 3.6 Photography & Illustration Style

{If the product uses photography or illustration, define the style direction. Photography: mood, lighting, composition, color treatment. Illustration: style, when to use, mood. If the product is purely UI with no imagery, state "N/A — product does not use photography or illustration" with brief rationale.}

### 3.7 Data Visualization Style

{If the product displays charts, graphs, or data visualizations, define the visual language: chart types, color usage in data, labeling standards, brand expression in data displays. If no data visualization, state "N/A" with rationale.}

### 3.8 Token Intent Map

{Map brand decisions to design token semantic names. Define the intent, not the value — the design system holds actual values. This is the bridge between Brand OS and implementation.}

| Brand Decision | Token Name | Intent |
|---|---|---|
| {Decision from sections above} | {semantic.token.name} | {How and where this token is used} |

> Example:
> | Warm accent for energy | `color.accent.warm` | Primary CTAs, emotional highlights. Max 2 per screen. |
> | Dark base for premium feel | `color.background.primary` | Main surface. OLED-friendly. Never pure black (#000). |
> | Gentle state transitions | `motion.duration.standard` | Default transition for state changes. Calm, not snappy. |

---

## 4. Motion & Interaction

For a product, motion IS the brand. How the interface responds to input, how elements enter and exit, and how state changes feel — these create brand personality more powerfully than static visuals.

### 4.1 Motion Principles

{Define 3-5 principles governing animation and transitions. Include timing philosophy (fast and responsive? slow and deliberate?), easing preferences, and what motion communicates about the brand personality.}

> Example: "1. Responsive, not reactive — animations acknowledge input immediately but resolve calmly (ease-out, never bounce). 2. Purposeful — every animation communicates information. No decorative animation. 3. Brief — 200-300ms standard. Users never wait for an animation."

### 4.2 Interaction Patterns

{Define how the product responds to user input: tap/click feedback, swipe behaviors, long-press patterns, pull-to-refresh style. What interaction patterns are signature to this brand?}

> Example: "Tap: subtle scale (0.97) with spring animation. Swipe: momentum-based with haptic detents. No bounce effects — spring with dampingFraction: 1.0."

### 4.3 Sound Identity

{If the product uses audio (notification sounds, UI feedback, haptics), define the sonic identity. What do notifications sound like? What haptic patterns are used? If the product is silent, state "N/A — product does not use audio cues" with rationale.}

---

## 5. Voice & Language

How the brand speaks. For app products, this is primarily UI copy — not marketing copy.

### 5.1 Brand Voice

{Define the brand's voice in 3-5 attributes with specific do/don't guidance for each. Voice is constant — it does not change by context (tone does).}

> Example:
> - Clear: Short sentences, common words, no jargon. DO: "You haven't talked to Sam in a while." DON'T: "Engagement metrics indicate decreased interaction frequency."
> - Warm: First person plural or second person. DO: "We'll remind you." DON'T: "A reminder has been scheduled."
> - Confident: Direct language, opinionated defaults. DO: "Try reaching out today." DON'T: "You might want to consider reaching out sometime."

### 5.2 Tone Spectrum

{Define how voice adapts across contexts. Tone changes by situation, urgency, and emotional weight. Provide 3-5 context-tone mappings with example copy.}

> Example:
> - Success: Warm, brief, genuine. "Nice — you're on a 7-day streak."
> - Error: Calm, helpful, no blame. "Something went wrong. Let's try that again."
> - Onboarding: Encouraging, action-oriented. "Let's start with the people who matter most."
> - Notification: Gentle, human, never urgent. "It's been a while since you talked to Sarah."
> - Empty state: Optimistic, guiding. "No check-ins yet. Start with someone you've been thinking about."

### 5.3 Micro-Copy Guidelines

{Define patterns for common UI text types. These are the highest-value guidelines for builders writing interface copy.}

Cover at minimum: button labels, error messages, empty states, loading states, confirmation dialogs, placeholder text, and notification copy. For each, state the pattern and provide an example.

> Example:
> - Buttons: Verb-first, 1-3 words. "Send Message", "Add Friend" — not "Submit" or "OK"
> - Errors: What happened + what to do. "Couldn't save. Check your connection and try again."
> - Empty states: Explain + guide action. "No friends added yet. Search for people you know."
> - Loading: Skeleton screens preferred. If text needed: "Loading..." — never "Please wait..."

### 5.4 Naming Conventions

{Define how features, screens, and UI elements are named within the product. What naming patterns create consistency? What should be avoided? This ensures builders and copywriters use the same vocabulary across the product.}

---

## 6. Brand Deliverables

Artifacts to be produced from this Brand OS. The implementation plan should decompose these into actionable tasks.

- [ ] Logo variants (primary, wordmark, icon-only, dark/light, monochrome)
- [ ] Color palette export with HEX/RGB values and usage rules
- [ ] Typography specimen with scale, pairings, and platform compatibility
- [ ] Design token definitions mapped from Section 3.8
- [ ] Icon set or icon style reference sheet
- [ ] {App store screenshot templates — if mobile app}
- [ ] {Social media templates — if social presence planned}
- [ ] {Landing page style reference — if web presence planned}
- [ ] {Onboarding copy deck — if onboarding flow defined}
- [ ] {Motion/animation reference — if Section 4 defines custom animations}

{Add or remove items based on the product's needs. Each checked item becomes a task in the implementation plan.}

---

## 7. Governance & Evolution

How this Brand OS stays alive and accurate over time.

### 7.1 Ownership & Review Cadence

{Define who owns brand decisions and how often this document is reviewed. For small teams, this may be the founder. The goal: make the document good enough that the owner is rarely consulted.}

> Example: "Owner: Product lead. Review: Quarterly, or when shipping a major new feature. Ad-hoc when a builder encounters a situation not covered here."

### 7.2 Immutable vs. Mutable Elements

{Classify brand elements as immutable (never change without a major brand revision) or mutable (can evolve per feature or context). Target: 80% immutable, 20% mutable.}

| Element | Classification | Rationale |
|---|---|---|
| {e.g., Brand voice attributes} | {Immutable / Mutable} | {Why this classification} |

> Example:
> | Mission & positioning | Immutable | Core identity anchor |
> | Primary color | Immutable | Brand recognition |
> | Accent colors | Mutable | Can evolve with features or themes |
> | Micro-copy patterns | Mutable | Adapt as product vocabulary grows |

### 7.3 Evolution Changelog

{Track significant brand decision changes. Each entry: date, what changed, why, who approved.}

| Date | Change | Rationale | Approved By |
|---|---|---|---|
| {YYYY-MM-DD} | Brand OS created | {Initial creation context} | {Name} |

---

## 8. Verification Checklist

Before finalizing this Brand OS, verify:

- [ ] All sections contain specific, actionable decisions (no placeholder text remains)
- [ ] Decision Principles are specific enough to resolve real ambiguities (not generic values)
- [ ] Color system includes both light and dark mode considerations
- [ ] Typography choices are compatible with target platforms
- [ ] Voice guidelines include do/don't examples for each attribute
- [ ] Micro-copy patterns cover at least: buttons, errors, empty states, loading, notifications
- [ ] Token Intent Map connects every major visual decision to a semantic token name
- [ ] Immutable vs. Mutable classification covers all major brand elements
- [ ] Brand Deliverables checklist reflects actual product needs
- [ ] Accessibility standards are stated with specific compliance levels
- [ ] Companion PRD exists or is planned for functional requirements

---

## 9. Glossary

{Define brand-specific terms, naming conventions, or vocabulary used in this document. List alphabetically.}

---

## 10. References

{List design inspiration, competitor analysis, research sources, and related documents. Include link to companion PRD if it exists.}

---

## 11. Approval & Sign-off

| Role | Name | Date | Status |
|------|------|------|--------|
| {Role} | {Name} | {Date} | {Pending/Approved/Rejected} |
