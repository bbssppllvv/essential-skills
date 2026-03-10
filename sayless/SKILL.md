---
name: sayless
description: >
  Write and review product copy, UX microcopy, and marketing text. Use this skill when the user asks to write button text, error messages, empty state copy, landing page headlines, onboarding flows, notification copy, or any product-facing text. Also use when reviewing or improving existing copy for clarity, brevity, and impact. Triggers on: "write copy", "button text", "error message copy", "landing page copy", "review this copy", "UX writing", "microcopy", "marketing headline", "empty state text", "improve this copy", "make this copy better", "copywriting help", "rewrite this", "too wordy", "shorten this text", "notification copy", "onboarding copy", "CTA text", "tooltip text".
---

# sayless

Write copy people read, trust, remember, and act on.

Read [references/copywriting-guide.md](references/copywriting-guide.md) before writing any copy. It contains the full methodology.

## Core rule

**Clarity > Respect > Character.** Always in this order.

## Workflow

### 1. Classify the context

Determine what is being written:

| Type | Key sections from guide |
|------|------------------------|
| UI microcopy (buttons, errors, empty states) | Sections 6, 8 |
| Product flows (onboarding, settings) | Sections 3, 6, 8 |
| Marketing / landing page | Sections 9, 11, 12, 15, 17, 18 |
| Full page or mixed | All sections apply |

### 2. Apply the universal structure

Every piece of copy must answer (explicitly or implicitly):

1. **Context** — where is the user?
2. **Job** — what are they trying to do?
3. **Solution** — what do you offer?
4. **Outcome** — what changes?
5. **Conditions** — limits, tradeoffs
6. **Next step** — what to do now

### 3. Write living words, kill dead ones

Before outputting any line, check: could this appear on 1,000 websites unchanged? If yes, rewrite with a concrete detail, number, or scenario.

### 4. Match tone to stakes

- **Low stakes** (onboarding, tooltips, success): warmth and personality allowed
- **High stakes** (errors, payments, deletion, security): calm, clear, zero personality

### 5. Self-check before output

Run the 7-point checklist (guide section 7):
1. Understandable in 3 seconds?
2. "Why should I care?" answered?
3. At least one concrete detail?
4. Can cut 30% without losing meaning?
5. Expectations honest?
6. Next step obvious?
7. Sounds human?

For marketing copy, also run the memorability checklist:
1. Is there a line worth quoting?
2. Can I visualize a scene?
3. Is there contrast or surprise?
4. Does this have a clear point of view?
5. Does it sound like a person, not a committee?
6. Can I remove another 20% and sharpen the punch?

If most are "no" — the text will be understood, but forgotten. Rewrite.

### 6. Output format
- Return the final copy first, clearly formatted
- Follow with a brief rationale (which principles applied, what was cut/changed)
- If revising existing copy, show before → after

## Quick patterns

**Buttons**: action + object ("Save changes", "Start free trial")

**Errors**: what happened + what to do ("You're offline. Reconnect to save changes.")

**Empty states**: teach + guide + one CTA ("Projects will appear here. Create your first one.")

**Hero section**: headline (outcome) + subhead (how) + proof/constraint + CTA

**Sticky lines**: "Not another X. Finally Y." / "Less noise. More decisions." / "Stop guessing. Start seeing."

**Before/After**: pain -> intervention -> concrete win

## Never use

Hype: revolutionary, seamless, cutting-edge, best-in-class, next-generation

Filler: very, really, just, actually, basically, literally, simply

Corporate: leverage, synergy, ecosystem, paradigm, holistic, empower

AI slop: "In today's fast-paced world", "Say goodbye to", "Supercharge your", "Take X to the next level"

Full banned list in guide section 21.

---

## Worked example: failed payment error message

**Request**: "Write an error message for when a user's credit card payment fails at checkout."

### Step 1 — Classify the context
UI microcopy (error message). Key guide sections: 6, 8. High-stakes moment — user is trying to pay and something went wrong.

### Step 2 — Apply the universal structure
1. **Context**: checkout page, payment just attempted
2. **Job**: complete the purchase
3. **Solution**: explain the failure and offer a path forward
4. **Outcome**: user can retry or fix the issue
5. **Conditions**: could be expired card, insufficient funds, or network error — we may not know which
6. **Next step**: update payment method or try again

### Step 3 — Write living words, kill dead ones
- Dead: "Transaction could not be processed." (could appear on 1,000 sites)
- Living: "Your card was declined. Update your payment method or try a different card."

### Step 4 — Match tone to stakes
High stakes (payment failure, user may be anxious). Zero personality. Calm, direct, helpful.

### Step 5 — Self-check
1. Understandable in 3 seconds? Yes.
2. "Why should I care?" answered? Yes — your payment didn't go through.
3. At least one concrete detail? Yes — "card was declined."
4. Can cut 30% without losing meaning? No, already tight.
5. Expectations honest? Yes — no false promise it will work next time.
6. Next step obvious? Yes — update method or try different card.
7. Sounds human? Yes.

### Step 6 — Output

**Final copy**:

> Payment failed — your card was declined. Update your payment method or try a different card.

**Rationale**: High-stakes moment, so tone is calm and direct with zero personality (guide section 6). Follows the error pattern: what happened ("card was declined") + what to do ("update your payment method or try a different card") from guide section 8. Avoided vague phrasing like "something went wrong" or "transaction could not be processed" — named the specific problem instead.
