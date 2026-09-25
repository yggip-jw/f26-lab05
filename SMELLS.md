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

**The smell.** Speculative over-abstraction: a configurable factory and mutable channel
registry support a notification system whose only channel is email.

**Classic or agent-specific.** Agent-specific, using the lecture's "Speculative
over-abstraction" category (slide 42), a form of classic speculative generality.
The likely cause is an underspecified request: the generator appears to have assumed
a need for plugin-style extensibility. That cause is inferred from the structure;
the original generation request is not available.

**Where in the code.** `src/notifications/notifierFactory.ts`: `ChannelName` permits
only `'email'`, but `registerChannel()`, `registeredChannels()`, and
`createNotificationChannel()` manage and query a module-level builder registry.
The module registers just `EmailChannel`, and the `ReservationManager` constructor
always calls the factory with `DEFAULT_NOTIFIER_CONFIG`.

**The principle it violates.** YAGNI: decouple changes that are actually needed rather
than paying for speculative extension points. The unnecessary part is the dynamic
registry and selection machinery, not the `NotificationChannel` interface itself,
which can provide a useful boundary for substitution and testing.

**What it makes expensive.** Understanding which notifier a manager receives requires
tracing the default config, factory lookup, and registration side effect instead of
one explicit construction or injected dependency. Giving two managers different test
notifiers is also awkward: the constructor accepts no notifier, so using the registry
requires replacing a shared builder between constructions and restoring it afterward.
That adds test setup and risks order-dependent tests, without a current requirement
for runtime channel registration.

### Smell 3

**The smell.** Phantom complexity: the reservation manager has a cache lookup path
that cannot produce a cache hit through its current public operations.

**Classic or agent-specific.** Agent-specific, using the lecture's "Phantom complexity"
category (slide 44). Missing context is a plausible cause: the cache machinery appears
to have been added without checking whether the service ever populates it. Free volume
could also explain the extra configuration and infrastructure. These are inferred
causes, not verified facts about the generation process.

**Where in the code.** `src/reservationManager.ts`: the constructor creates a private
`QueryCache`, and `listBookingsForRoom()` calls `get()` before querying storage.
However, the manager never calls `set()` or exposes this cache to callers, so its
entries remain empty and every lookup falls through to `storage.findByRoom()`.
`src/cache/queryCache.ts` and `src/cache/cacheConfig.ts` supply TTL, capacity, and
invalidation machinery that provides no caching benefit on this path.

**The principle it violates.** Simplicity (KISS): every additional mechanism should
justify its maintenance cost with useful behavior. This integration adds a dependency,
configuration, and a cache-hit branch without changing the query result or avoiding
any storage reads. The issue is the unused integration, not that caching itself is
inherently unnecessary.

**What it makes expensive.** Investigating slow room queries requires tracing the cache
configuration and lookup path before discovering that no values are ever stored;
tuning the TTL or capacity cannot help. Making this cache useful is also more than
adding a `set()` call: creating or cancelling a booking would then require invalidating
the room's cached list to avoid stale results. The current scaffolding hides that
unfinished design work behind the appearance of a working optimization.

---

## Milestone 2: One small fix

One fix, behavior preserved, suite green, zero test edits.

**Which smell you attacked.** Smell 1, duplication over reuse in pricing. The two
implementations encode the same policy, so sharing that policy removes the need to
synchronize future pricing changes without redesigning the reservation service.

**What changed.** Added `src/pricing.ts` with a pure `calculatePrice(room, start, end)`
function that owns the pricing constants and calculation. In `src/reservationManager.ts`,
the existing public `calculatePrice()` delegates to it; the duplicated constants and
private `applyDiscounts()` were removed. In `src/reportGenerator.ts`, `revenue()` calls
the same function; the duplicated constants, private `priceOf()`, and its now-unused
`durationOf()` helper were removed. Both paths now use one implementation.

**What you deliberately did not touch.** The scope line is sharing the existing pricing
policy, not changing it. Existing public class methods, thresholds, multiplier order,
and rounding after every stage are preserved. Revenue still recalculates prices using
the supplied room data rather than reading `booking.priceCents`; changing that would
alter behavior when room rates change. Validation, report filtering, notifications,
cache integration, and all tests are unchanged. Those concerns are independent of
eliminating pricing duplication and would make this fix harder to review.

**How you know behavior is preserved.** Before and after the change, `npm test` passes
all 39 tests across three files, and `npm run typecheck` passes. No tests were edited.
`tests/booking.test.ts` checks base pricing, the three-hour discount, premium pricing,
and the evening discount separately, alongside booking and cancellation behavior.
`tests/reporting.test.ts` checks revenue totals, averages, per-room totals, exclusion
of cancelled bookings, availability, and occupancy. The suite does not exhaustively
check combined discounts or rounding-sensitive rates, so a green suite alone does not
prove equivalence. Reviewing the extracted calculation confirms the original order:
round the hourly charge, then apply and round the premium, long-booking, and evening
multipliers in that order, with the same conditions and constants.

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
