# Spacing Guide

Spacing is the invisible structure of good UI. Bad spacing is the fastest way to make a design feel "off" without anyone being able to explain why.

---

## 1. Spacing Scale

Use a 4px base unit. Every spacing value should be a multiple of 4. This creates a predictable rhythm that the eye recognizes subconsciously.

### The Scale

```
4px   — Inline icon gap, tight element pairs
8px   — Related elements (label to input, icon to text)
12px  — Compact internal padding (badges, small cards)
16px  — Standard internal padding, form field gaps
24px  — Section sub-groups, card padding (compact)
32px  — Card padding (standard), group separations
48px  — Section spacing within a page region
64px  — Major section breaks
96px  — Page-level section divisions, hero padding
```

### CSS Tokens

```css
:root {
  --space-1:  4px;    /* 0.25rem */
  --space-2:  8px;    /* 0.5rem  */
  --space-3:  12px;   /* 0.75rem */
  --space-4:  16px;   /* 1rem    */
  --space-6:  24px;   /* 1.5rem  */
  --space-8:  32px;   /* 2rem    */
  --space-12: 48px;   /* 3rem    */
  --space-16: 64px;   /* 4rem    */
  --space-24: 96px;   /* 6rem    */
}
```

**Rule:** If you reach for a value not on this scale, question whether you actually need it. Arbitrary values (e.g., 18px, 37px) break visual rhythm.

---

## 2. Component-Level Spacing Patterns

### Cards

```css
/* Compact card (lists, dashboards) */
.card-compact {
  padding: var(--space-4);         /* 16px */
  gap: var(--space-2);             /* 8px between children */
}

/* Standard card */
.card {
  padding: var(--space-6);         /* 24px */
  gap: var(--space-3);             /* 12px between children */
}

/* Spacious card (marketing, feature highlights) */
.card-spacious {
  padding: var(--space-8);         /* 32px */
  gap: var(--space-4);             /* 16px between children */
}
```

### Forms

```css
/* Vertical form layout */
.form-group {
  gap: var(--space-6);             /* 24px between field groups */
}

/* Label to input */
.field {
  gap: var(--space-2);             /* 8px label-to-input */
}

/* Helper/error text below input */
.field-hint {
  margin-top: var(--space-1);      /* 4px */
}

/* Form actions (submit row) */
.form-actions {
  margin-top: var(--space-8);      /* 32px — visual separation */
  gap: var(--space-3);             /* 12px between buttons */
}
```

### Sections

```css
/* Within a page region */
.section-inner {
  gap: var(--space-12);            /* 48px between sub-sections */
}

/* Between major page sections */
.section {
  padding-block: var(--space-16);  /* 64px top and bottom */
}

/* Hero / above-the-fold */
.hero {
  padding-block: var(--space-24);  /* 96px */
}
```

### Lists and Tables

```css
/* List items */
.list-item {
  padding: var(--space-3) var(--space-4);  /* 12px vertical, 16px horizontal */
}

/* Table cells */
.table-cell {
  padding: var(--space-3) var(--space-4);  /* 12px / 16px */
}

/* Compact table (data-dense dashboards) */
.table-cell-compact {
  padding: var(--space-2) var(--space-3);  /* 8px / 12px */
}
```

---

## 3. Spacing and Information Density

Spacing controls how much cognitive weight a page carries. This is a deliberate product decision, not a cosmetic one.

| Density | Internal Padding | Section Gap | Best For |
|---------|-----------------|-------------|----------|
| **Dense** | 8-12px | 24-32px | Dashboards, data tables, developer tools, power-user interfaces |
| **Default** | 16-24px | 48-64px | SaaS products, content apps, standard business tools |
| **Spacious** | 24-32px | 64-96px | Marketing pages, onboarding flows, consumer products |

### Matching Density to Context

- **Dashboards and admin panels** lean dense. Users scan, compare, and act quickly. Generous whitespace wastes screen real estate.
- **Marketing and landing pages** lean spacious. Every element needs room to breathe so visitors absorb messaging without fatigue.
- **Product UI** sits in the middle. Functional areas (sidebars, toolbars) can be denser; content areas (settings, profiles) can be more open.

**Anti-pattern:** Using marketing-level spacing in a dense data tool (everything feels empty) or dashboard-density on a landing page (everything feels cramped).

---

## 4. Proximity and Grouping

The most important spacing principle: **proximity implies relationship.**

```
Closer together = related
Farther apart   = separate groups
```

### Practical Application

The gap between a heading and its content should be **smaller** than the gap between that content and the next heading. This is how you signal structure without extra borders or dividers.

```css
/* Heading is closer to its own content than to the previous section */
h2 {
  margin-top: var(--space-12);     /* 48px — large gap above */
  margin-bottom: var(--space-4);   /* 16px — small gap to content */
}
```

**Test:** Remove all borders, backgrounds, and dividers. Can you still see the groupings from spacing alone? If yes, your spacing is doing its job.

---

## 5. Responsive Spacing Adjustments

Spacing should tighten on smaller screens, but not collapse entirely.

### Scale Factor by Breakpoint

| Breakpoint | Scale Factor | Example: 64px section gap becomes |
|------------|-------------|-----------------------------------|
| < 480px (mobile) | 0.5-0.625x | 32-40px |
| 480-768px (tablet) | 0.75x | 48px |
| 768px+ (desktop) | 1x | 64px |

### Implementation

```css
:root {
  --section-gap: 32px;
  --card-padding: 16px;
  --hero-padding: 48px;
}

@media (min-width: 768px) {
  :root {
    --section-gap: 48px;
    --card-padding: 24px;
    --hero-padding: 64px;
  }
}

@media (min-width: 1200px) {
  :root {
    --section-gap: 64px;
    --card-padding: 32px;
    --hero-padding: 96px;
  }
}
```

### What Changes, What Stays

| Property | Scales? | Notes |
|----------|---------|-------|
| Section gaps | Yes | Reduce by 0.5-0.75x on mobile |
| Card padding | Yes | But keep minimum 12-16px |
| Form field gaps | Slightly | 24px desktop, 16-20px mobile |
| Icon-to-text gaps | No | 4-8px stays fixed |
| Inline spacing | No | Small values stay constant |

**Rule:** Internal component spacing (padding within a button, gap between icon and label) stays fixed. Structural spacing (between sections, around containers) scales with viewport.

---

## Pre-Ship Checklist

- [ ] All spacing values come from the 4px scale (no arbitrary numbers)
- [ ] Proximity clearly signals grouping (no ambiguous gaps)
- [ ] Density matches the product context (dashboard vs. marketing)
- [ ] Headings are closer to their content than to the preceding section
- [ ] Spacing tightens on mobile but doesn't collapse
- [ ] Internal component spacing stays consistent across breakpoints
- [ ] No two adjacent elements share the same gap when they belong to different groups

---

*Spacing is not decoration. It is the primary tool for communicating structure, hierarchy, and relationships in an interface.*
