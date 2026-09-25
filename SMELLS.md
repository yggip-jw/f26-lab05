# reservation-service: Smells and One Fix

Fill in each section. One section per milestone. Keep it short and specific. Point at files
and methods, not adjectives.

---

## Milestone 1: Three smells

Three smells, each in a different part of the module. For each one, fill in all five parts.

### Smell 1

**The smell.** Duplication over reuse: the booking and reporting paths independently
implement the same pricing rules.

**Classic or agent-specific.** Agent-specific, using the lecture's "Duplication over reuse"
category (slide 41). The likely cause is missing context: the reporting implementation
rebuilds pricing logic already present in the booking manager. This is an inference from
the duplicated code, not a verified account of how it was generated.

**Where in the code.** `src/reservationManager.ts`, `calculatePrice()` and
`applyDiscounts()`, and `src/reportGenerator.ts`, `priceOf()`. Both paths calculate the
hourly charge, apply a 1.15 premium multiplier, a 0.9 multiplier for bookings lasting
at least 180 minutes, and a 0.95 multiplier for bookings starting at or after 17:00,
rounding after each step. The policy constants are also duplicated in these files.

**The principle it violates.** DRY: one pricing policy has two independently maintained
representations. The pricing decision is not hidden behind a shared module boundary,
so a policy change cannot stay local to one implementation.

**What it makes expensive.** Changing the long-booking discount from 10% to 15% requires
updating both implementations. If only the manager is updated, a new three-hour booking
in a non-premium room at $10/hour, starting before 17:00, costs $25.50, while the revenue
report still computes $27.00 for that same booking. Every pricing change therefore
requires finding and synchronizing both copies, including their rounding order.

### Smell 2

**The smell.**

**Classic or agent-specific.**

**Where in the code.**

**The principle it violates.**

**What it makes expensive.**

### Smell 3

**The smell.**

**Classic or agent-specific.**

**Where in the code.**

**The principle it violates.**

**What it makes expensive.**

---

## Milestone 2: One small fix

One fix, behavior preserved, suite green, zero test edits.

**Which smell you attacked.** And why that one.

**What changed.** Files and methods you touched, and what the code does differently now.

**What you deliberately did not touch.** Name the scope line you drew and why you drew it
there. "I ran out of time" is not a scope line.

**How you know behavior is preserved.** Point at the suite, say what it actually covers, and
say what it would not catch.

---

## Milestone 3: Two proposals and one false positive

One proposal for each milestone 1 smell you did not fix.

### Proposal A (not coded)

**The problem.** Name it.

**The decomposition.** What are the pieces, what does each own, and where do the rules live?

**One cost.** Something this actually costs. "No real downside" is not a cost.

### Proposal B (not coded)

**The problem.**

**The decomposition.**

**One cost.**

### The thing that looks smelly but is fine

**What it is.** File and method.

**Why it is fine.** Defend it with properties of the code, not with its line count.

**What would flip your verdict.** Name the change that would turn this into a real problem.
