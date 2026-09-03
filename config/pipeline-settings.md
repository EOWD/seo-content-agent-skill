# Pipeline settings

## approval_mode: auto

- **auto** (current): the pipeline runs end-to-end with NO mid-run approval
  stops. Plan, research, drafting, and gates proceed automatically. The
  review artifacts (plan-review, research-review, dashboard, previews) are
  still published at every stage — as a live audit trail the SEO manager can
  watch and interrupt at any time, not as blockers. The run ends at
  review-ready drafts + review pack.
- **checkpoint**: the pipeline pauses for explicit SEO-manager approval after
  the cluster plan and after research (fact-sheet freeze). Use for brand-new
  topic areas, sensitive subjects, or when onboarding a new operator.

Switch modes by editing this line or telling Claude "run in checkpoint mode".

## The one gate that never automates

Human medical + WHO Code compliance review (Gate 4) happens before anything
is published. In the current drafts-only scope this is simply: a human reads
the review pack before manually publishing. If auto-publishing is ever built
(roadmap Phase 3), Gate 4 becomes a hard blocking step in the pipeline —
automation never removes it.

## Interrupting an auto run

The manager can reply at any time during a run ("stop", "change the angle on
article 3") — the artifacts show enough state to steer without pre-approval
stops.
