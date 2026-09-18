# TVT edge delivery

Files:

- `image-contract.yaml`: deployable image/runtime contract.
- `desired-state.schema.json` and `desired-state.example.json`: manager input.
- `analytics-event.schema.json`: authoritative edge output.
- `management-api.openapi.yaml`: HTTP/SSE surface.
- `MANAGER-UI-HANDOFF.md`: manager and UI behavior.
- `manifests/tvt-edge.yaml`: Kubernetes starting manifest.
- `SHA256SUMS`: integrity list generated at delivery time.

Load the archive with:

```sh
podman load -i tvt-edge-runtime-intel-285h-2026.09.18-v1.oci.tar
```

The example RTSP URL and geometry are placeholders. Replace them through the
cluster's secret/config delivery process before deployment.
