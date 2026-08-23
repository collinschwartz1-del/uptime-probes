# uptime-probes

Public, credential-free reachability checks for Collin's member-facing surfaces.

## Why this repo is public

GitHub charges Actions minutes on **private** repositories only — public repositories run
on standard runners for free, with no minute cap. On 2026-08-23 the account hit 100% of
its 2,000 included minutes with nine days left in the billing cycle, and every scheduled
job across every repo started failing at dispatch. The Titan probe alone was 611 runs
that cycle — roughly 31% of the entire account budget — to run six `curl` commands.

Billing rounds every job up to a whole minute, so a seven-second check costs exactly what
a sixty-second one does. Moving the check somewhere minutes are free is the only fix that
does not trade away coverage.

## The rule

> **Never add a secret, token, key, or private hostname to this repository.**

That is not a style preference — it is the single condition that lets this repo be public,
which is the only reason the checks are free. Anything that needs a credential belongs in
the private repo it came from.

Concretely: the weekly Titan **data-path** check submits real rows through the public anon
path using `TITAN_HEALTH_TOKEN`, so it stayed behind in the private
`collinschwartz1-del/titan-vault` repo. Only the read-only probe moved here.

## What runs here

| Workflow | Cadence | Covers |
|---|---|---|
| `titan-probe.yml` | every 15 min | vault bundle + worksheet marker, `/worksheet`, main-site vault link, Reflections login, old-domain redirect, `/api/apply` validation |

Cadence went **up** in the move (15 min, from 30) because minutes are no longer the
constraint. Cron minutes are deliberately offset off `:00/:15/:30/:45` — the round slots
are the congested ones GitHub batches and drops, which is why the old `*/30` schedule only
ever delivered about 27 of its nominal 48 runs per day.

## What this cannot catch

The probe is **read-only**, so it cannot see a broken *write* path. On 2026-07-29 an
over-strict INSERT policy silently 401-rejected real member submissions while every page
still returned 200. Green here does not mean the stack is healthy — that gap is covered by
the weekly `datapath` job in `titan-vault`.

## Keepalive

GitHub disables scheduled workflows in a repository with no commit activity for 60 days,
and nobody has any reason to commit here. A monthly job writes a timestamp to `.keepalive`
so the schedule stays enabled. Without it the probe would go quiet on its own and stay
that way — which is worse than having no check at all.
