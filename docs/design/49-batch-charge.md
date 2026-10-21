# Design proposal: batch charge (#49)

Status: proposal only. No code has been written for this issue, because the
issue asks for maintainer agreement on the design first.

## Problem

A merchant with many due subscriptions pays one transaction fee per `charge`.
A batch call would charge many subscriptions in one transaction.

## Proposal

Add `batch_charge(merchant, ids)`:

1. Require the merchant's authorization once, for the whole batch.
2. For each id, run the same checks as `charge`: the subscription belongs to
   the merchant, is Active, the interval has elapsed, and one more period fits
   under the cap. The token transfer is the same `transfer_from` as `charge`.
3. Cap the batch at a fixed size, so one call stays within the read and write
   budget.

Soroban rolls back a whole invocation when it returns an error, so the batch
needs a chosen failure mode, one of:

- **All or nothing.** Any failing id aborts the batch and nothing is charged.
  Simple and easy to reason about, but one bad id blocks the rest.
- **Skip and report.** A failing id is skipped and its reason is returned in a
  result vector. Needs the per-id result to be returned without aborting, which
  means catching errors inside the batch.

This proposal recommends skip-and-report, with the batch size capped.

## Trade-offs

- A batch saves fees, but the write budget per transaction is the real limit,
  so the cap matters more than the saving.
- Each charge still updates the subscription, the allowance and the registry,
  so a batch does not reduce the state writes, only the transactions.

## Open questions for maintainers

- All or nothing, or skip and report?
- What batch size is acceptable within the network's write budget?
