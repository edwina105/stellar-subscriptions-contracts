# Design proposal: optional end ledger for subscriptions (#52)

Status: proposal only. No code has been written for this issue, because the
issue asks for maintainer agreement on the design first.

## Problem

A subscriber may want "for one year only". Today a subscription runs until its
cap is reached or it is cancelled.

## Proposal

Add an optional end ledger, `expires_at: Option<u32>`, to the subscription.
After that ledger `charge` is refused. The expiry can only narrow what the
merchant may do, so it never lets a merchant charge more.

Because `subscribe` takes this as a new argument, the proposal offers two ways
to add it:

- **Change `subscribe`.** Add the argument to `subscribe`. This is the simplest
  API but breaks every caller, which is a major change under `version()`.
- **Add `subscribe_until`.** Keep `subscribe` unchanged and add a second entry
  point that takes the end ledger. No caller breaks. This is the recommended
  option.

Either way:

- `charge` refuses with a new error once the current ledger passes `expires_at`,
  and `charge_status` reports it as its own reason.
- A subscription with no expiry behaves exactly as today.

## Trade-offs

- A second entry point means two ways to create a subscription. Both must share
  one code path, or they can drift.
- Adding a field to `Subscription` changes the stored layout, so `VERSION` must
  be bumped in either option.

## Open questions for maintainers

- Should the expiry be a ledger number, as proposed, or a duration from
  subscription time?
- Is `subscribe_until` acceptable, or should `subscribe` change?
