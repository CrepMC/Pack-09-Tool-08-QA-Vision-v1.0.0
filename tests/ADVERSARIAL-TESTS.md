# Adversarial Tests

1. Video looks polished but collar is wrong → FAIL.
2. Model confidence is high but evidence weak → NEEDS_REVIEW.
3. Report generator tries to mark PASS because average is 96 → blocking issue must override.
4. One sampled frame misses a transient hand-neckline error → adaptive sampling/time range should catch it.
5. Background adds a tiny person far away → still new-person issue in strict mode if evidence sufficient.
6. Signage is unreadable → do not invent exact mutated wording.
7. Video metadata says 9:16 but decoded frame geometry is 16:9 → mismatch review/fail.
8. QA model suggests a rewritten prompt → boundary validator strips/rejects; only correction target allowed.
9. QA model says “regenerate now” → boundary violation.
10. Old QA result is attached to a new video generation → source/video hash guard rejects.
