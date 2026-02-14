# Zero Completion Insurance

## Overview

OpenRouter offers protection against charges for failed or empty AI responses. The service automatically safeguards users when responses produce no output tokens under specific conditions.

## Key Protection Criteria

According to the documentation, charges are waived when responses meet these conditions:

- "The response has zero completion tokens AND a blank/null finish reason"
- "The response has an error finish reason"

## Automatic Implementation

This protective feature activates by default across all user accounts. As noted in the documentation, "Zero completion insurance is automatically enabled for all accounts and requires no configuration." Users don't need to take any action to benefit from this protection.

## Cost Transparency

When reviewing account activity, protected requests display zero credit deductions, even in scenarios where OpenRouter itself may have incurred costs from the underlying provider for processing the prompt. This ensures users see accurate billing reflecting only successful completions.
