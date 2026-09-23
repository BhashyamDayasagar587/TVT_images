# v2-ungated-enroll (2026.09.18) — changelog vs v1

## Behavior
- `face_enrollment` is now UNGATED: emits `enrollment_capture_event` on any
  usable face (no `B->A` line-cross required). Cooldown 1s, cap 400/run.
  `crossed`/`line` attached only when a recent accepted crossing exists
  (schema-legal: `shared/schemas.check_crossing` allows both absent).
- `face_recognition` + `anpr` unchanged (still crossing/zone-gated).
- Code: only `edge/person_app.py` `EnrollmentApp` (+31 lines `run_frame`
  override, docstring, `_last_emit/_cooldown_s`). `_emit`, models, thresholds
  (`yaw<=45`, `px>=60`, `det>=0.65`), contracts, schemas untouched.

## Files changed vs v1
- `manifests/tvt-edge.yaml`: 1 line — image
  `...:intel-285h-2026.09.18-v1` →
  `...:intel-285h-2026.09.18-v2-ungated-enroll`.
- `image-contract.yaml`: `spec.image.reference` + `spec.image.archive` bumped
  to v2 names. Parent block unchanged.
- `README.md`: v2 `podman load` block added.
- `SHA256SUMS`: refreshed for the above + v2 tar (+ `.tar.gz`) entries.

## Packaging notes (known deviations)
- Export tool is `docker save` (no skopeo/podman on build box), so the tar
  carries Docker-style `manifest.json` (full `RepoTags`) alongside OCI
  `index.json` whose `org.opencontainers.image.ref.name` annotation is the
  bare tag. Both resolve to image ID `96053a19826b`; load with
  `docker load` or `podman load` (or `k3s ctr images import`).
- Size `~967MB (v1) → ~923MB (v2 tar)`: same scale. (First v2 export
  ballooned to ~1.9GB by re-COPYing vendor/models already in the parent —
  accidental double-pack; rebuilt minimal: 1 small top layer with only
  `edge/person_app.py` over parent ID `c50d4996fcec`. Final v2 image ID
  `ced19bc19c0b`.)
- Build (offline): `docker build -f Containerfile.v2minimal --pull=false
  --network=none
  --build-arg TVT_PARENT=localhost/tvt-edge-runtime:intel-285h-2026.09.18-v1`.
  `py_compile` clean.

## Test status
- Mock test PASS: no-crossing usable face → 1 ungated enrollment event;
  cooldown blocks immediate second; `FaceRecognitionApp` still silent
  without crossing.
- Live-camera e2e PENDING: office RTSP `192.168.1.10:554` refused connection
  at build time (0 frames), so no live events observed yet.
- `test_scenario.py gate6_flip` (crossed-only enrollment) fails BY DESIGN
  on v2.
