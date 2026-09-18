# TVT Mills pilot handoff

This bundle is the authoritative v1 interface between the TVT edge image and
the management/UI system. The image runs analytics only; manager, database,
retention, identity resolution, and UI remain outside it.

The manager writes the complete desired state atomically to
`/configs/desired_state.json`. Every content change must increment
`revision`; an identical same-revision replay is accepted as idempotent.
Camera assignment comes only from each camera's `apps`. To register people at
the back gate, change only that camera to `["face_enrollment"]`; all other
cameras keep their current apps. A camera cannot run enrollment together with
recognition or ANPR.

Zone and line edits use the same desired-state file. The worker swaps a new
geometry snapshot between frames: the current frame finishes with the old
coordinates and the next frame uses the new coordinates. Geometry-only edits
do not restart the camera or models. Stable unique IDs are required.

RTSP URLs are secret values, not desired-state values. Create one UTF-8 secret
file per camera under `/run/secrets/apexfabric/<camera-id>.rtsp`. The desired
state contains only its `file:` reference.

Consume `GET /events` as SSE. Persist the last SSE cursor and reconnect with
`Last-Event-ID`. Deduplicate durable ingestion by the event envelope's
`event_id` because reconnects or network retries may redeliver data.
Validate every event with `analytics-event.schema.json`.

The image emits a `camera_snapshot_event` for each active camera every five
seconds continuously. There is no one-hour pause and the edge does not delete
periodic snapshots. The manager owns archival and retention. The event's
`snapshot_url` is relative to the edge service.

For identity matching, use the normalized 512-value
`payload.embeddings.face`. Body embeddings are intentionally absent in v1.
For ANPR, one event is produced from a per-vehicle vote, so simultaneous or
successive vehicles do not share stabilization state.

UI editors must keep coordinates normalized to 0..1, prevent self-intersecting
polygons, preserve zone/line IDs during coordinate edits, and publish the
entire desired state with a higher revision. The example coordinates are
placeholders and must be calibrated to the real cameras.

Operationally, liveness is `/healthz`; readiness is `/readyz`. Do not send
traffic until readiness returns 200. Kubernetes must run UID/GID 10001, use a
read-only root filesystem, mount writable `/state` and
`/tmp/apexfabric`, and forward SIGTERM.

Build provenance: the original local V8 image contained one layer recorded
with an unsupported uncompressed MIME type. Podman exported and re-imported
that unchanged root filesystem into a private normalized base before the final
Podman OCI build. The original V8 digest remains recorded in
`image-contract.yaml`; no Docker daemon or Docker command is used.
