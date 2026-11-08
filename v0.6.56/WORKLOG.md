# WORKLOG — v0.6.56 (local only, never published)

Chain 0.6.55 → 0.6.56. Built 2026-11-08 (night round). Local build +
local tests only: desktop untouched, relay untouched, echo-sync
untouched.

## What happened, in order

1. Read the v0.6.55 pattern (build_pack.py, verify_pack.py, plan.json,
   patch_v55.py, tests/test_v55.py) and the u8-hotapply applier.
2. Base-hash check — FOUND A LINEAGE FORK, stopped to investigate
   before building (see REPORT §1, this is the one deviation from
   the task text):
   - v0.6.50/src/pipeline.py (5d241e64…, 57,293 B) does NOT hash to
     the 0.6.55 manifest entry for app/echo_scene/pipeline.py.
   - The 0.6.55 manifest entry IS 6d8f85d7… (54,596 B) — the 0.6.47-era
     pipeline, identical to v0.6.49/base/pipeline.py, and also the
     entry in the 0.6.47 and 0.6.48 manifests. v0.6.51–0.6.54 (non-UI)
     manifests list the 0.6.50 file; the desktop lineage
     (0.6.47 → 0.6.54-ui → 0.6.55, 9,800 files) never picked it up.
   - stage_present is byte-identical in both pipeline versions
     (verified by extraction + comparison), so the bug and the fix
     are the same either way.
   - Decision: base = v0.6.49/base/pipeline.py (the file that hashes
     exactly to the 0.6.55 manifest entry — the desktop's actual
     file). Shipping the 0.6.50 file instead would have silently
     dragged the unreviewed 0.6.50 depth-threading change into this
     pack and collided with the pending 0.6.50 rebase.
   - base/version.inc = v0.6.55/src/version.inc (d1613650…, matches
     the 0.6.55 manifest). Recovered viewer.html verified:
     sha256 840075ea… (16,342 B), identical to the manifest
     install-root entry.
3. patch_v56.py written and run: stage_present only (template
   fallback + soft-fail), viewer.html copied byte-identical,
   version.inc 0.6.55 → 0.6.56. First run failed on a bad assertion
   (`def stage_fuse` is not defined in pipeline.py — fuse is imported
   lazily inside main); fixed the assertion list, reran clean. Diff
   base→src reviewed line by line: only stage_present changed.
4. build_pack.py written (mirrors v0.6.55) and run: zip 512,848 B,
   sha256 b77cf43c…. verify_pack.py: 50 checks, 0 failures (one
   check fixed during the run: bytes.decode does not translate CRLF,
   the seam check now normalizes endings first).
5. tests/test_v56.py written and run: 38 passed, 0 failed. One test
   design error caught by the run itself: the old-code reference
   cannot use a root-only template (old code never looks there — that
   IS the bug), so the reference now runs old code with the template
   at app/viewer.html; new code with a root-only template produces
   byte-identical scene.html.
6. exp/evidence.json + rerun logs (exp/test_v56_run.log,
   exp/verify_pack_run.log) recorded.

## Files

- base/pipeline.py, base/version.inc — verified base files
- recovered/viewer.html + recovered/PROVENANCE.md — recovered original
- src/pipeline.py, src/viewer.html, src/version.inc — patched files
- patch_v56.py, build_pack.py, verify_pack.py, plan.json,
  echoscene-manifest.json, EchoScene-Update.zip
- tests/test_v56.py (38 checks), exp/evidence.json + run logs
- REPORT.md (this pack's report), WORKLOG.md (this file)

## Not done / not in this pack

- Maximize→restore .NET crash (WER 2026-11-08 20:59, c000041d in
  KERNELBASE.dll): evidence preserved in crash-evidence/, web_app.py
  untouched, NOT fixed here. Separate issue.
- 0.6.48 / 0.6.49 / 0.6.50 rebases onto the 0.6.55 lineage: still
  pending, untouched by this pack (this pack's pipeline base is the
  0.6.47-era file, so the 0.6.50 rebase still applies cleanly on top
  of 0.6.56 when it is done — its base file is unchanged in the
  0.6.56 manifest).
- No desktop install, no relay push, no echo-sync push, no publish.
