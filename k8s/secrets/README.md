# SOPS-encrypted Secrets live here.

Files matching `k8s/secrets/*.yaml` are fully managed by SOPS+age:

- Encrypt + edit: `sops k8s/secrets/<name>.yaml`
- `.sops.yaml` at repo root lists the two age recipients (cluster + workstation)
- ArgoCD decrypts at sync time via KSOPS (needs plugin enablement — pending goal
  "SOPS+age setup for secrets" on ticket 7525543657412235264)

Pending secret: `tailscale-oauth.yaml` → Secret `tailscale` (ns `tailscale`),
keys: `oauth_client_id`, `oauth_client_secret`.
Until that pipeline is live, create the Secret manually on the cluster.
