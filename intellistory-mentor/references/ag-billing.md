<!-- Generated from the IntelliStory agent knowledge base (article `ag-billing`, last edited 2026-08-06). Do not edit here — edit the KB and rebuild. -->
# Credits, Cost and Spend

Generation bills a real account. This is live, not a sandbox.

## Before a large batch

`get_billing_status` returns the plan and current standing. Check it before
committing a user to something expensive — a fifty-shot storyboard run is not a
free action, and discovering the limit halfway through leaves a half-boarded
sequence.

`get_spend_analytics` shows where the money has gone. Useful when a user asks why
their balance moved.

## Always Ask — estimate before you spend

`generate_image` and `generate_video` take `estimate_only: true`. That call
queues nothing and returns `estimate_usd` for the exact model + settings you'd
run. The workflow for spending on a user's behalf is:

1. Call the generate tool with `estimate_only: true`.
2. Show the user the number and what it buys ("~$0.61 for a 5s 1080p take").
3. On their go-ahead, call again without `estimate_only`.

Real runs return `estimated_cost_usd` alongside `job_id`. If the wallet can't
cover the estimate, the call returns `error: "insufficient_funds"` with the
estimate and queues nothing — relay that plainly and stop; don't retry.

## Estimate vs actual

The estimate shown before a job and the amount finally billed can differ. The
estimate is a reservation ceiling — actual cost comes back from the provider
once the work is done, and that (at or below the estimate) is what gets
charged. Image cost in particular varies with resolution.

So: quote an estimate as an estimate. Don't tell a user a batch "will cost X" as
though it were fixed.

## Record cost on every version

`cost_usd` or `cost_credits` in the version metadata, every time. See
`ag-versions.md`. Without it there's no way to reconcile a project's spend against
the bill, and the user's own analytics under-report.

## Exemptions exist

Some accounts don't get metered. Don't infer a user's billing situation from
whether a generation succeeded, and don't tell a user they have or haven't been
charged — read `get_billing_status` or say you're not sure.

## What to say

Plain and specific: "that's about 40 credits for the twelve frames — want me to
go ahead?" Confirm before spending at scale. A user who didn't expect a charge is
a support ticket.

Don't describe the billing mechanism, the providers behind it, or how rates are
derived. What it costs and what they get is the whole of what they need.

Related: `ag-generation.md`, `ag-versions.md`.
