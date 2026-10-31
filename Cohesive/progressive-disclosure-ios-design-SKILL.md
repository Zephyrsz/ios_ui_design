---
name: progressive-disclosure-ios-design
description: Design, review, or refactor iOS and SwiftUI pages using progressive disclosure. Use when creating screen information architecture, deciding what belongs on the first screen versus secondary views, simplifying dense interfaces, organizing advanced settings, choosing between inline expansion, sheets, navigation, and "view all" flows, or generating implementation-ready page specifications and SwiftUI plans.
---

# Progressive Disclosure for iOS Page Design

## Purpose

Apply progressive disclosure to make an iOS interface easier to understand without removing useful capability.

Progressive disclosure means:

> Show the minimum information and controls needed for the user's current task, then reveal more detail when the user intentionally asks for it or when the workflow makes it relevant.

The goal is not to hide complexity. The goal is to present complexity at the right time, in the right context, and at the right level.

Use this skill for:

- Designing a new iOS or SwiftUI screen.
- Refactoring a crowded screen.
- Reviewing wireframes, screenshots, mockups, or existing SwiftUI/UIKit code.
- Deciding which content belongs on the primary screen.
- Designing summaries, "View All" flows, expandable sections, detail pages, filters, settings, and advanced controls.
- Producing an implementation-ready design specification.

Do not use progressive disclosure as a default excuse to hide important information, required actions, pricing, privacy implications, safety information, errors, or irreversible consequences.

## Core Principle

Optimize the amount of complexity visible **at one moment**, not the total number of features in the product.

A successful design lets the user quickly answer:

1. Where am I?
2. What can I do here?
3. What is the primary next step?
4. Where can I find more detail?
5. What will happen if I open or execute it?

## Required Inputs

Inspect all available context before proposing a design:

- User goal and primary task.
- Target audience and expected expertise.
- Screen content and controls.
- Content volume and likely growth.
- Usage frequency for each action.
- Dependencies between steps.
- Risks and consequences of actions.
- Existing navigation architecture.
- Platform and minimum iOS version.
- Existing design system, components, and code conventions.

If some information is missing, make reasonable assumptions, state them briefly, and continue. Ask a question only when the missing detail would materially change navigation, safety, data handling, or the primary workflow.

## Design Workflow

### Step 1: Define the page job

Write one sentence:

> This page helps the user [perform a task] by [working with specific content].

Then identify:

- Primary user goal.
- Primary action.
- Primary content.
- Completion state.
- Most likely next step.

If the page has several unrelated primary goals, recommend splitting or restructuring it before styling it.

### Step 2: Inventory all information and actions

List every element currently proposed or present:

- Navigation.
- Titles and context.
- Status and feedback.
- Primary action.
- Secondary actions.
- Content items.
- Metadata.
- Filters and sorting.
- Help and explanatory text.
- Advanced options.
- Destructive actions.
- Empty, loading, error, and permission states.

Do not decide placement until the inventory is complete.

### Step 3: Classify each element

Assign each item to one of four disclosure levels.

#### Level 1: Immediate

Show directly on the primary screen when the item is:

- Required to understand the page.
- Required to begin or complete the primary task.
- High frequency and high value.
- A critical status or error.
- Necessary for an informed decision.
- A consequence involving payment, privacy, data loss, permissions, or irreversible action.

#### Level 2: Preview

Show a concise summary or representative subset when the item is:

- Useful for orientation but too large for the first screen.
- Frequently consulted but not always acted upon.
- Naturally summarized by recent, recommended, active, favorite, or incomplete items.
- Part of a larger collection that can be opened with "View All".

#### Level 3: On demand

Reveal after an explicit user action when the item is:

- Secondary detail.
- A complete list behind a preview.
- An infrequent option.
- Context-specific metadata.
- A filter, sort, or customization control not required initially.

#### Level 4: Advanced

Place in an advanced area when the item is:

- Expert-oriented.
- Rarely used.
- Technically complex.
- Potentially distracting to most users.
- Meaningful only after basic configuration is complete.

For every Level 2–4 item, explain why delaying it improves the primary task.

### Step 4: Select the disclosure pattern

Choose the lightest pattern that preserves context and predictability.

#### Inline expansion

Use for short details that can appear without disrupting the task.

Good for:

- Optional explanations.
- A few additional settings.
- Secondary metadata.
- Small FAQ answers.

Avoid when expansion produces a very long page, changes several unrelated regions, or makes scroll position unpredictable.

#### Section preview plus View All

Use when a larger collection can be represented by a small subset.

Good for:

- Recent items.
- Albums, groups, categories, projects, or playlists.
- Recommendations.
- Activity history.

The preview should communicate what the full destination contains. Keep item style and ordering consistent between preview and full list.

#### Navigation to a detail or complete-list screen

Use when the disclosed content has its own title, state, actions, search, filtering, or substantial information.

The destination must provide:

- A clear navigation title.
- A predictable back path.
- Consistent item representation.
- Context-relevant toolbar actions.

#### Sheet

Use for a focused temporary task that the user completes or dismisses before returning.

Good for:

- Creating or editing one item.
- Choosing among a limited set of options.
- A focused filter configuration.
- A short confirmation flow.

Do not use a sheet as a substitute for a substantial navigation hierarchy.

#### Menu or context menu

Use for compact, secondary commands whose meaning is predictable from their labels.

Do not hide the primary action, required navigation, or an action users must discover to complete the task.

#### Disclosure group

Use for a small group of related settings or explanatory content when expanding in place preserves context.

Avoid nesting disclosure groups deeply.

#### Staged flow

Use when later inputs depend on earlier choices or when a complex task has a meaningful sequence.

Show progress and preserve entered data. Let users go back and edit prior choices.

### Step 5: Design the primary screen

Build the primary screen in this order:

1. Navigation context and page title.
2. Critical state, warning, or permission information.
3. Primary content or primary action.
4. A limited number of high-value previews.
5. Clearly labeled paths to more content.
6. Secondary actions.

Do not make every section visually dominant. Define one primary visual anchor.

Prefer familiar iOS patterns and system components unless customization creates clear product value.

### Step 6: Define reveal triggers and labels

Every hidden or deferred item needs a discoverable trigger.

Use labels that predict the result:

- View All
- Show More
- More Details
- Advanced Options
- Edit Filters
- See Full History

Avoid vague triggers such as an unlabeled chevron or ellipsis when users cannot infer the result.

Specify for every trigger:

- Trigger label or icon.
- Resulting state or destination.
- Whether the action is reversible.
- How the user returns or collapses it.
- Whether state persists after leaving the page.

### Step 7: Preserve continuity

The revealed content should feel like an expansion of what the user already saw.

Maintain consistency in:

- Naming.
- Item layout.
- Icons and imagery.
- Sort order.
- Selection state.
- Terminology.
- Animation direction.
- Navigation hierarchy.

A preview list should not become an unrelated grid on the full page unless the layout change has a clear user benefit.

### Step 8: Cover all states

Specify behavior for:

- Loading.
- Empty state.
- Partial data.
- Error and retry.
- Offline state when relevant.
- Permission not granted.
- Very large content volumes.
- Search with no results.
- Expanded and collapsed state restoration.

Progressive disclosure must remain understandable in every state, not only with ideal sample data.

### Step 9: Validate accessibility and adaptability

Check:

- Dynamic Type, including accessibility sizes.
- VoiceOver names, hints, values, and expanded/collapsed state.
- Sufficient touch targets.
- Keyboard and switch control navigation where applicable.
- Light mode, dark mode, and increased contrast.
- Reduce Motion.
- Localization and long text.
- Right-to-left layouts.
- Small screens and landscape.

Do not rely only on color, animation, or chevron direction to communicate disclosure state.

### Step 10: Produce implementation guidance

For SwiftUI, map the design to the simplest suitable primitives, such as:

- `NavigationStack`
- `List`
- `Section`
- `DisclosureGroup`
- `NavigationLink`
- `.sheet`
- `Menu`
- `ContentUnavailableView`
- `.toolbar`
- `@State`, `@Binding`, or observable model state

Keep state ownership explicit. Persist disclosure state only when it benefits task continuity; do not preserve incidental expansion forever by default.

Do not generate code unless the user asks for code or implementation. When code is requested, follow the existing project architecture and deployment target.

## Decision Rules

Apply these rules when deciding visibility:

- High frequency + required for the current task: show directly.
- High frequency + large content set: show a useful preview and View All.
- Low frequency + easy to understand: reveal on demand.
- Low frequency + expert complexity: place under Advanced Options.
- Critical to trust, consent, cost, safety, or irreversible consequences: never defer beyond the decision point.
- Needed only after another selection: reveal conditionally after that selection.
- Useful only for one content item: place in that item's context, not globally.

## Content Sizing Heuristics

Use judgment rather than rigid numbers, but begin with these defaults:

- Preview enough items to explain the section and offer immediate value.
- Avoid a preview so long that it delays the next major section below the fold.
- Prefer one clear primary action per screen.
- Group related secondary actions instead of presenting a flat wall of controls.
- Avoid more than one nested disclosure level unless the domain demands it.
- If users repeatedly open the same hidden area, reconsider promoting it.

## Anti-Patterns

Flag and correct these problems.

### Hiding essential information

Do not defer:

- Price or subscription terms before purchase.
- Privacy impact before consent.
- Data deletion consequences before confirmation.
- Required fields without indicating they are required.
- Blocking errors or important status.

### Mystery meat disclosure

Do not use icons or vague labels whose result cannot be predicted.

### Excessive nesting

Do not create a chain such as More → Advanced → Details → Additional Settings for one common task.

### False simplicity

Do not make the first screen visually clean by moving ordinary tasks into hard-to-find menus.

### Modal overuse

Do not stack sheets or use a modal for deep browsing.

### Layout discontinuity

Do not change naming, item representation, or ordering unexpectedly between preview and full content.

### Unstable expansion

Do not cause major scroll jumps, cover the user's current focus, or collapse entered data unexpectedly.

### Disclosure without feedback

After a trigger, make the changed state obvious and announce it accessibly.

## Review Checklist

Before finalizing, verify:

### Task clarity

- The page has one clearly stated job.
- The primary action is visible.
- The user can understand the page without opening secondary content.

### Disclosure quality

- Every hidden item has a reason to be deferred.
- Every trigger is discoverable and accurately labeled.
- The revealed result matches the user's expectation.
- Returning or collapsing is easy.

### Information safety

- Cost, privacy, destructive consequences, required status, and critical errors remain visible at the correct decision point.

### Navigation

- The user always knows the current location.
- Full-list and detail destinations have titles and back paths.
- Tab bars represent destinations, not actions.
- Toolbars contain page-relevant actions.

### Visual hierarchy

- One element or region acts as the primary visual anchor.
- Preview sections do not compete equally for attention.
- Secondary controls are visually quieter but still discoverable.

### Adaptability

- Dynamic Type does not hide triggers or truncate essential labels.
- VoiceOver communicates expanded/collapsed state.
- Long localized strings remain usable.
- Empty, error, and loading states preserve the disclosure logic.

### Implementation

- State ownership is clear.
- Disclosure does not cause unexpected data loss.
- Analytics can measure whether users find and use deferred content.

## Required Output Format

Unless the user requests another format, produce the following sections.

### 1. Page goal

State the primary task, audience, and assumptions.

### 2. Information hierarchy

For each item, classify it as:

- Immediate.
- Preview.
- On demand.
- Advanced.

Explain the reasoning briefly.

### 3. Recommended page structure

Describe the screen from top to bottom, including navigation, sections, primary action, previews, and disclosure triggers.

### 4. Disclosure map

For each deferred item, specify:

- Entry point.
- Pattern used.
- Revealed content.
- Return or collapse behavior.
- Persisted state, if any.

### 5. Interaction details

Describe loading, empty, error, expanded, collapsed, and completion behavior.

### 6. Accessibility and adaptation

Cover Dynamic Type, VoiceOver, localization, contrast, and device-size behavior.

### 7. SwiftUI implementation plan

Name suitable SwiftUI primitives, state boundaries, view decomposition, and data-flow considerations. Provide code only when requested.

### 8. Risks and tradeoffs

Identify anything that may be hidden too deeply, create extra taps, reduce discoverability, or produce navigation complexity.

### 9. Final checklist

Give a concise pass/fail checklist for implementation review.

## Compact Example

User request:

> Design a record collection home page containing groups, recent records, search, add record, filters, statistics, and advanced import settings.

Recommended hierarchy:

- Immediate: page title, search, Add Record, recent records, critical sync state.
- Preview: top groups with View All, short statistics summary.
- On demand: full groups list, filters, complete statistics.
- Advanced: import format, conflict handling, metadata matching rules.

Recommended structure:

1. Navigation title: Records.
2. Toolbar action: Add Record.
3. Search field.
4. Groups preview with three useful groups and View All.
5. Recent Records collection.
6. Statistics summary linking to a detail page.
7. Import settings under a dedicated settings or import workflow, not on the home screen.

Rationale:

- Adding a record is a contextual action, not a tab destination.
- A small Groups preview introduces organization without pushing records below the first screen.
- Full statistics and import controls remain available without competing with the primary browsing task.

## Final Principle

A good progressive-disclosure design is not merely clean at first glance. It remains discoverable, predictable, accessible, and efficient as the user moves from the simple view into deeper capability.
