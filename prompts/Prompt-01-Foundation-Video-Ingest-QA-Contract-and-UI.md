# ROLE AND SYSTEM BOUNDARY

You are building **Tool 08 — QA Vision** in a multi-tool Google Flow AI fashion-video system.

Inputs:
- the actual generated video;
- T02 ProductDNA and ConsistencyLockSpec;
- T03 CreativeBrief;
- T04 CameraPlan;
- T05 MotionPlan;
- T06 EnvironmentPlan;
- T07 CinematicPlan.

Output:
- QAReport only.

Downstream:
- T09 Prompt Optimizer consumes failures, evidence, correction targets and passing constraints.

# ABSOLUTE QA RULE

Judge the generated video against the contracts.

Do not judge only whether the video “looks good”.

Product identity and hard technical requirements can fail an otherwise attractive video.

# ABSOLUTE BOUNDARY

QA Vision does not:
- rewrite the generation prompt;
- regenerate the video;
- edit the video;
- edit ProductDNA;
- edit CameraPlan;
- edit MotionPlan;
- edit EnvironmentPlan;
- edit CinematicPlan;
- silently repair artifacts.

It identifies what failed, where it failed, and what a downstream repair tool must target.

# EVIDENCE RULE

Major and critical findings should contain evidence:
- timecode or range;
- frame or region reference when available;
- expected behavior;
- observed behavior;
- confidence.

If evidence is insufficient, use UNKNOWN or NEEDS_REVIEW rather than guessing.

# PASS RULE

No blocking issue may coexist with PASS.

A high average score never overrides a hard-constraint failure.

# PROMPT 1/3 — FOUNDATION, VIDEO INGEST, QA CONTRACT, SOURCE VALIDATION AND UI

Implement the foundation of Tool 08 without modifying the behavior of completed T02–T07 tools.

## 1. Architecture

Create or equivalent separated responsibilities:
- `domain/qaReport`
- `domain/qaIssue`
- `domain/qaEvidence`
- `domain/domainScore`
- `domain/correctionTarget`
- `domain/passingConstraint`
- `domain/errors`
- `adapters/videoReviewAdapter`
- `adapters/upstreamContractAdapter`
- `adapters/visionCapabilityAdapter`
- `services/inputValidator`
- `services/videoMetadataService`
- `services/frameSamplingService`
- `services/sourceContractService`
- `services/issueClassifier`
- `services/blockingRuleService`
- `services/scoreService`
- `services/correctionTargetService`
- `services/exportService`
- `state/qaVisionStateMachine`
- UI components for video player, timeline, overall result, scorecards, issue list, evidence inspector, passing constraints, correction targets and export.

No god component.

## 2. Inbound validation

Require:
- generated video reference;
- ProductDNA;
- CreativeBrief;
- CameraPlan;
- MotionPlan;
- EnvironmentPlan;
- CinematicPlan.

Validate:
- artifact versions;
- project/source IDs;
- whether artifacts belong to the generated video's compilation context if such metadata exists;
- stale/mismatched artifacts;
- video accessibility.

If upstream is missing:
do not produce a false complete QA report.

Use `NEEDS_REVIEW` or block depending severity.

## 3. Video metadata

Inspect where capability exists:
- duration;
- width/height;
- aspect ratio;
- frame rate;
- rotation/orientation metadata;
- codec/container as optional diagnostic only.

Aspect ratio should also be checked against decoded/rendered frame geometry when possible, not only metadata.

## 4. Historical 9:16 safeguard

CameraPlan hard ratio = 9:16.

If actual decoded output is 16:9:
critical Camera failure.

Do not trust a filename or metadata tag saying portrait when pixels are landscape.

## 5. Sampling foundation

Do not judge from one thumbnail.

Create sampling strategy with:
- start frame;
- early motion;
- mid motion;
- phase transitions;
- late motion;
- final frames;
- optional regular interval samples.

Config:
`max_sample_gap_seconds` default around 0.5, adjustable.

If runtime supports richer temporal review, use it.

## 6. Adaptive sampling

When a suspicious issue appears:
sample more densely around that time.

Examples:
- neckline briefly changes;
- hand crosses chest;
- foot slides;
- sign morphs;
- camera zoom appears.

Do not rescan whole video at maximum density unnecessarily.

## 7. Frame review record

Each sampled record:
- frame_id;
- timestamp;
- observations;
- evidence refs;
- review confidence.

Observations should be concise, not chain-of-thought.

## 8. QA domain structure

Create independent domain review modules:
ProductQA
CameraQA
MotionQA
EnvironmentQA
CinematicQA.

Each can emit:
issues[];
passing constraints[];
domain score;
confidence;
needs review.

Global aggregator determines final status.

## 9. ProductQA shell

Product QA consumes ProductDNA/locks.

Check framework:
- category;
- silhouette;
- neckline;
- collar;
- sleeve;
- closure;
- buttons;
- pockets;
- color;
- pattern;
- logo/text;
- material;
- companion;
- accessories;
- crop/occlusion.

Do not implement all advanced temporal logic yet; Prompt 2 will.

## 10. CameraQA shell

Consume CameraPlan.

Check:
ratio;
orientation;
framing;
crop;
camera movement;
zoom;
tracking;
subject scale;
start/end states;
continuity.

## 11. MotionQA shell

Consume MotionPlan.

Check:
path;
orientation;
 product contact;
gesture;
foot sliding;
teleport;
phase completion;
final settled state.

## 12. EnvironmentQA shell

Consume EnvironmentPlan.

Check:
scene identity;
anchors;
signage;
new people/objects;
geometry;
ground;
parallax;
collision.

## 13. CinematicQA shell

Consume CinematicPlan.

Check:
hue;
material;
exposure;
skin;
focus/DOF;
motion blur;
bloom;
anchor readability.

## 14. Evidence object

Use:
evidence_id;
type;
time_start;
time_end;
frame_ref;
region;
description;
confidence.

Evidence can be:
frame;
frame range;
visual region;
metric;
contract diff;
manual confirmation.

## 15. Issue object

Issue requires:
issue_id;
domain;
secondary domains optional;
issue_type;
severity;
blocking;
title;
description;
expected;
observed;
evidence;
recommended_target;
confidence.

`recommended_target` is NOT a full replacement prompt.

It is a repair objective.

## 16. Severity

info:
diagnostic.

minor:
visible but low impact.

major:
material quality/contract failure that should be fixed.

critical:
hard identity/technical failure.

Blocking state can be true because of hard constraints even if a visual difference seems small.

## 17. Blocking rules foundation

Examples critical/blocking:
- hard ratio mismatch;
- main product category/silhouette/neckline mutation;
- required accessory missing;
- strict background replacement;
- final settled state missing when required;
- severe product hue/material transform.

Do not use one fixed numerical score to determine these.

## 18. Domain score

Score 0–100.

Also store:
status;
blocking count;
major count;
confidence;
notes.

Score is a summary, not an override.

## 19. Overall decision

Possible:
PASS;
FAIL;
NEEDS_REVIEW;
UNKNOWN.

PASS conditions:
- no blockers;
- configured thresholds pass;
- sufficient evidence/confidence.

FAIL:
- blocker;
- threshold failure with enough evidence.

NEEDS_REVIEW:
- potentially critical but evidence insufficient;
- human confirmation required.

## 20. Passing constraints

Record explicit successful constraints.

Examples:
- `cam_ratio_9x16`;
- `cam_dolly_back`;
- `env_scene_identity`;
- `cine_blue_lock`;
- `motion_final_settled`.

These become preservation requirements for T09.

## 21. Correction target shell

Each target:
target_id;
domain;
priority P0–P3;
objective;
issue_refs;
preserve_passing_constraints;
do_not_change.

Do not write the final prompt.

## 22. UI

### Video Player
scrub/seek.

### Timeline
issue markers.

### Overall
PASS/FAIL/REVIEW.

### Domain scorecards
Product, Camera, Motion, Environment, Cinematic.

### Issue list
severity/domain/timecode.

### Evidence inspector
click issue seeks video.

### Passing constraints
what must remain unchanged.

### Correction targets
downstream repair objectives.

### Export
only locked QAReport.

## 23. State machine

WAITING_INPUT
VALIDATING_INPUT
INPUT_REJECTED
READY_FOR_REVIEW
ANALYZING_VIDEO
REVIEW_PARTIAL
BUILDING_QA_REPORT
NEEDS_HUMAN_REVIEW
READY_FOR_DECISION
PASS
FAIL
LOCKED
EXPORTING
EXPORTED
BLOCKED.

Analysis/model failure cannot route to PASS.

## 24. User/human review

Human can:
confirm issue;
dismiss false positive;
change severity with audit note;
confirm uncertain product detail.

Do not silently overwrite original model observation; preserve audit.

## 25. Boundary validator

If QA model returns:
“Change the prompt to ...”
strip/reject full prompt content and reduce it to a structured target.

If it says:
“regenerate the video”
do not execute.

T09/orchestration owns next action.

## 26. Error catalog

At minimum:
QA_INPUT_VIDEO_MISSING
QA_INPUT_CONTRACT_MISSING
QA_SOURCE_MISMATCH
QA_VIDEO_UNREADABLE
QA_VIDEO_METADATA_INVALID
QA_ANALYSIS_CAPABILITY_UNAVAILABLE
QA_FRAME_SAMPLE_FAILED
QA_EVIDENCE_INSUFFICIENT
QA_REPORT_SCHEMA_INVALID
QA_FALSE_PASS_GUARD
QA_PROMPT_REWRITE_BOUNDARY
QA_REGENERATION_BOUNDARY
QA_HANDOFF_FAILED.

## 27. Accessibility

- video controls keyboard accessible;
- timeline markers labeled;
- issue severity not color-only;
- scorecards text;
- evidence captions;
- mobile issue list/player usable.

## 28. Prompt 1 tests

Test:
1. valid video/contracts;
2. missing video;
3. missing ProductDNA;
4. source mismatch;
5. 9:16 metadata;
6. 16:9 mismatch;
7. sampling includes start/end;
8. issue click seeks time;
9. blocker prevents PASS;
10. low confidence → NEEDS_REVIEW;
11. passing constraint created;
12. correction target is not a final prompt;
13. no regeneration action.

## 29. Definition of Done

The app can load a fixture video + contracts, create a structured shell report and demonstrate blocking PASS logic without needing the full Prompt 2 vision review.

Run build/typecheck/tests.

STOP after Prompt 1.

# EXTENDED IMPLEMENTATION APPENDIX — PROMPT 1

## A. Artifact ownership registry

T08 owns:
observations, issues, evidence, scores, QA decision, passing constraints, correction targets.

T09 owns:
prompt repair.

Orchestrator/generation layer owns:
regeneration.

Make this explicit in code validation.

## B. Source truth precedence

For QA comparison:
ProductDNA > CameraPlan/MotionPlan/EnvironmentPlan/CinematicPlan > Creative intent.

QA must not use a creative adjective to excuse a hard product mutation.

## C. Video identity

Prefer stable:
generation_id + content hash if available.

Filename is insufficient.

## D. Sampling manifest

Store sampled timestamps, not raw images in QA artifact.

Frame refs may point to temporary/internal samples.

## E. Start/end density

Always inspect:
first meaningful frames;
last several frames.

Historical start/end errors require this.

## F. Critical-region registry

ProductDNA can compile:
neckline;
logo;
pants feet;
accessory;
etc.

Use regions to prioritize review.

## G. Score UI

Show risk clearly:
Product 89 FAIL
Camera 97 PASS
not just an overall 94.

## H. Needs-review UX

For uncertain critical issue:
show “Needs Review” with evidence.

Do not force user to inspect all normal frames.

## I. Passing constraints are first-class

They are not merely absent issues.

Explicitly record important success to guide repair.

## J. Prompt 1 self-check

- Can critical issue override score?
- Can incomplete analysis avoid false PASS?
- Is video tied to contracts?
- Are evidence/timecode types present?
- Is prompt rewriting blocked?
- Is regeneration blocked?
