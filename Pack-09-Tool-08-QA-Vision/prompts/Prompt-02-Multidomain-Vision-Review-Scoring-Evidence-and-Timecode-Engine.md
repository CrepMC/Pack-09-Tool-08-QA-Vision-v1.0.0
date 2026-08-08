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

# PROMPT 2/3 — MULTI-DOMAIN VISION REVIEW, TEMPORAL EVIDENCE, SCORING AND CORRECTION-TARGET ENGINE

Keep Prompt 1 intact. Add the real QA review intelligence.

## 1. Review orchestration

Use a deterministic workflow:

1. Validate video and contracts.
2. Read hard constraints.
3. Inspect metadata.
4. Sample timeline.
5. Run ProductQA.
6. Run CameraQA.
7. Run MotionQA.
8. Run EnvironmentQA.
9. Run CinematicQA.
10. Correlate cross-domain issues.
11. Dedupe issues.
12. Classify severity/blocking.
13. Build scores.
14. Preserve passing constraints.
15. Build correction targets.
16. Decide PASS/FAIL/REVIEW.

Do not ask one model:
“Is this video good?”.

## 2. ProductQA — identity

Compare generated video across time to ProductDNA.

For each identity-critical field, inspect multiple relevant frames.

### Neckline/collar
Detect:
- V-neck becomes rounded;
- collarless becomes folded collar/lapel;
- neckline edge changes during hand interaction.

### Pattern
Detect:
- dots disappear;
- stripes direction changes;
- pattern scale changes severely;
- logo/graphic morphs.

### Closure/buttons
Detect missing/new buttons when identity-critical.

### Accessories
Detect:
- hat/bag disappears;
- changes design;
- duplicates;
- moves unnaturally.

### Silhouette
Detect large product shape changes.

## 3. Product temporal consistency

One correct start frame is not enough.

A garment can mutate only during:
- turn;
- hand interaction;
- blur;
- late frames.

QA must inspect throughout the clip.

If mutation appears transiently:
still issue with time range.

## 4. Product occlusion vs mutation

Hand covering neckline briefly can be:
Motion/Product occlusion.

If shape is different after hand leaves:
Product mutation.

Allow secondary domains.

## 5. Product color

Use ProductDNA semantic color family + CinematicPlan locks.

Detect obvious semantic drift:
blue → teal/green;
beige-cream → orange/yellow.

Do not pretend exact colorimetry if vision capability cannot measure it.

Use confidence.

## 6. Product material

Check visual material category:
matte vs glossy/satin-like;
denim-like;
leather-like;
unknown.

Do not infer fiber composition.

## 7. CameraQA — aspect ratio

Compare decoded geometry to CameraPlan.

Hard lock mismatch:
critical.

Also verify:
orientation;
black bars/canvas issues if relevant.

## 8. CameraQA — movement

Evaluate planned movement family:
static;
dolly-back;
tracking;
etc.

If CameraPlan expects dolly-back but video is static:
Camera major/critical depending project rule.

If video uses zoom instead of camera translation and zoom is forbidden:
blocking/major-critical.

## 9. CameraQA — subject scale

Inspect subject size over time.

Compare to:
stable;
gradual growth;
start-far/end-near.

Detect:
rapid scale drift;
unexpected close-up;
subject leaving frame.

If metrics unavailable:
use visual/semantic estimate with confidence.

## 10. CameraQA — crop

Critical crop regions from CameraPlan:
feet;
pants;
neckline;
bag;
dress hem.

If required region leaves frame:
issue with time range.

## 11. CameraQA — start/end relation

If CameraPlan says far_to_near:
verify start visually farther/smaller than end.

Detect reversal.

## 12. CameraQA — continuity

Single take:
detect obvious cuts/angle jumps/discontinuous scene reset.

Do not overclaim exact edit detection if capability weak.

## 13. MotionQA — product contacts

Use MotionPlan forbidden contacts.

Examples:
hand touches neckline;
pulls shirt;
adjusts sleeve;
hands in pocket when disallowed;
pulls drawstring;
removes/adjusts hat.

Record exact time.

## 14. MotionQA — path

Check:
forward path;
lateral drift;
safe-zone compatibility.

A large unexpected sideways movement may be Motion + Camera compatibility issue.

## 15. MotionQA — turn

Compare to allowed orientation range.

Unsupported 180-degree spin:
major/critical when product consistency endangered.

## 16. MotionQA — foot sliding

Inspect foot-to-ground relationship across adjacent frames.

Evidence:
weight-bearing foot shifts over floor without coherent step.

Use confidence and do not claim perfect biomechanical tracking if unavailable.

## 17. MotionQA — teleport

Detect discontinuous body location/orientation not explained by a cut when single-take.

Critical continuity issue.

## 18. MotionQA — final state

Review final frames densely.

Required final settled state means:
- no active step;
- both feet stable/support state;
- no active turn;
- body weight visually settled;
- arms no longer transitioning;
- product visible.

If video ends mid-step:
critical.

This directly enforces the project's final-standing-state requirement.

## 19. EnvironmentQA — scene identity

Compare generated scene to EnvironmentPlan/reference.

Detect:
- replacement location;
- major architecture mismatch;
- missing hard anchor.

Strict background replacement:
critical.

## 20. EnvironmentQA — new people/objects

If strict:
detect new pedestrians, furniture, plants, bags, vehicles, props.

Use evidence and confidence.

Tiny uncertain far-background object:
NEEDS_REVIEW if not reliable.

## 21. EnvironmentQA — signage

Compare:
panel identity;
known visible wording if reliable;
text stability across frames.

If source text is partial/unreadable:
do not invent what the “correct” words should be.

Only report obvious text invention/morph.

## 22. EnvironmentQA — anchor persistence

Check hard anchors at visible times.

Temporary subject occlusion is not anchor deletion.

Look before/after occlusion.

## 23. EnvironmentQA — geometry

Detect:
wall bends;
door changes shape;
column disappears;
floor warps;
stairs morph.

Use timecoded evidence.

## 24. EnvironmentQA — ground plane

Ground stability supports MotionQA.

If floor slides/morphs under feet:
Environment issue;
may contribute to perceived Motion foot-slide issue.

Correlate without duplicate over-counting.

## 25. EnvironmentQA — parallax

Compare CameraPlan movement and EnvironmentPlan expected parallax.

Examples:
dolly-back with fixed 3D scene should produce depth-consistent relative movement.

Detect flat background sliding or depth swaps when obvious.

## 26. CinematicQA — product hue

Use ProductDNA + CinematicPlan.

Obvious hue identity drift:
Product + Cinematic issue.

Primary domain can be cinematic when caused by grade; secondary product.

## 27. CinematicQA — material

Detect source matte becoming glossy/satin-like.

Primary cinematic, secondary product.

## 28. CinematicQA — exposure

Detect:
highlight clipping;
shadow crush;
product detail loss;
face clipping.

Use visual evidence; if histogram capability exists, it can supplement.

## 29. CinematicQA — skin

Detect gross plastic/waxy look, extreme smoothing or glow.

Do not critique body traits.

This is rendering artifact quality only.

## 30. CinematicQA — DOF/focus

Check identity-critical details:
neckline;
pattern;
logo;
product edges.

Check high-priority environment anchors.

If blurred past readability:
issue.

## 31. CinematicQA — motion blur

Detect:
limb trails;
garment smear;
pattern smear;
accessory ghosting.

Correlate with MotionPlan speed and CameraPlan movement.

## 32. CinematicQA — bloom/halation

Detect excessive glow washing edges/detail.

Use moderate severity unless it harms product/anchor identity.

## 33. Cross-domain issue correlation

One event can affect multiple domains.

Example:
hand touches neckline → motion violation + product mutation.

Do not generate five duplicate issues for one root event.

Create one primary issue with secondary domains where useful.

## 34. Evidence confidence

Confidence reflects visible evidence, not model certainty theater.

Low confidence on a critical field:
NEEDS_REVIEW.

Do not set 0.99 automatically.

## 35. Time ranges

Use:
start_time,
end_time.

For persistent issue:
range may cover large clip.

For exact final failure:
last second/range.

## 36. Adaptive temporal refinement

If initial sample at 3.0s looks normal and 3.5s mutated:
sample between them to locate onset.

Bound computation.

## 37. Passing-constraint compiler

For each domain, record meaningful passes.

Avoid recording hundreds of trivial passes.

Prioritize:
hard requirements;
identity-critical;
known historical failure points.

## 38. Correction-target generation

Target derives from issues.

Example Product/Motion:
“Prevent any hand contact with the collarless V-neck and preserve its exact geometry.”

Preserve:
9:16;
dolly-back;
background;
color.

Do not write the full prompt.

## 39. Priority levels

P0:
blocking identity/technical.

P1:
major important.

P2:
quality improvement.

P3:
optional polish.

T09 should address P0 before style improvements.

## 40. Score calculation

Use configurable weights but hard blockers separately.

Product generally highest weight in preservation-first pipeline.

Do not hide risk through averaging.

## 41. Threshold

Project target may be >95 consistency.

Implement config default 95.

But domain-specific minimum and blocker logic remain.

## 42. Human review triggers

Examples:
tiny logo uncertain;
partial sign;
material ambiguous;
subtle hue shift near boundary;
foot slide unclear.

Present evidence to user.

## 43. Vision capability adapter

Detect whether runtime supports:
- video-native review;
- frame extraction;
- temporal reasoning;
- region comparison;
- metadata access.

Do not invent functions.

If only frame review:
sample and aggregate.

## 44. No OCR overreach

Do not repeatedly OCR every frame.

Use visual model/text understanding where possible, OCR only if platform already provides a reliable capability and exact text matters.

Partially legible reference remains conservative.

## 45. Performance

Cache:
video hash;
metadata;
frame samples;
domain results keyed by source contract hashes.

Human label changes should not rerun whole vision pass.

## 46. Golden fixture expectations

Known correct:
9:16,
blue shirt,
beige pants,
dolly-back,
settled end,
same environment,
subtle grade.

Known failure variant:
hand touches neckline at 3.1–3.8s and neckline changes.

Expected:
Product critical;
Motion critical/secondary;
Camera PASS;
Environment PASS;
Cinematic PASS;
overall FAIL;
correction target preserves passing areas.

## 47. Prompt 2 tests

Add fixtures for:
- neckline mutation;
- color drift;
- 16:9;
- crop feet;
- unexpected zoom;
- camera static;
- mid-step end;
- hand touch;
- foot slide;
- background replacement;
- new person;
- sign mutation;
- geometry morph;
- hue drift;
- material gloss;
- DOF blur;
- motion smear.

## 48. Completion

Report:
- temporal review;
- domain analyzers;
- evidence;
- issue correlation;
- scoring;
- passing constraints;
- correction targets;
- capability adapter;
- tests.

STOP after Prompt 2.

# EXTENDED IMPLEMENTATION APPENDIX — PROMPT 2

## A. Temporal anomaly detection strategy

Compare observations across neighboring frames.

Mutation is often visible as:
stable → abnormal → stable/different.

Dense review around transition improves evidence.

## B. Identity-critical product fields

Criticality comes from ProductDNA locks, not generic fashion assumptions.

A collarless neckline can be more important than a tiny seam.

## C. Product reference coverage

If rear design is unknown and video turns rear:
QA should distinguish:
unsupported-view hallucination risk
from confirmed wrong rear detail.

Use NEEDS_REVIEW where source truth is insufficient.

## D. Camera zoom vs dolly visual distinction

Possible cues:
subject/background relative scale and parallax.

If evidence is not strong enough to distinguish:
report likely unexpected zoom with confidence or NEEDS_REVIEW.

Do not fake optical-flow certainty.

## E. Crop duration

A one-frame edge touch may be minor.
Sustained critical-region crop can be major/critical.

Hard CameraPlan rules can elevate it.

## F. Motion final-state dense sampling

Inspect final 0.5–1.0s more densely where feasible.

This is necessary for mid-step detection.

## G. Foot-slide evidence

Use floor anchor relationship when visible.

If floor itself morphs, correlate Environment issue.

## H. Signage evidence

Do not claim exact word mutation unless both reference and output are readable enough.

Geometry/shape morph can still be reported without OCR.

## I. Cinematic hue QA

A product can be under warm light while still semantically blue.

Do not flag every color cast.

Flag identity-level drift.

## J. Material QA

Use appearance categories, not imagined fiber labels.

Matte→gloss is observable.
Cotton→polyester usually is not visually reliable.

## K. Score confidence

Domain confidence can be lower if:
- product often occluded;
- low resolution;
- motion blur;
- missing reference coverage.

This can trigger review.

## L. Dedupe rules

If one root event has multiple symptoms:
link secondary domains.

Do not penalize score five times without justification.

## M. Passing constraint threshold

Record a pass only when confidence sufficient.

Unknown is not a pass.

## N. Correction target minimality

Every target should answer:
what to fix;
what to preserve;
what not to change.

This is the key handoff behavior.

## O. Prompt 2 self-check

- Temporal, not thumbnail-only?
- Final frames dense?
- Product criticality source-based?
- No OCR overreach?
- No fake metric claims?
- Cross-domain dedupe?
- Passing constraints explicit?
