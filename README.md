# home-config — jojee home server

Single-node k3s on the Mini PC (`jojee.iptime.org`), provisioned by Ansible
(day-0 only) and then managed entirely by ArgoCD (GitOps).

## Layout

```
ansible/          day-0 bootstrap only
  playbook.yaml   k3s install -> ArgoCD install -> app-of-apps
  inventory.yaml  host: jojee (home)
  templates/      backup script
  group_vars/     pinned versions + paths
k8s/
  bootstrap/      root.yaml (root Application, applied once by Ansible)
  apps/           Application manifests — the real GitOps layer. Add apps here.
  secrets/        SOPS-encrypted Secrets (decrypted in-cluster by KSOPS)
```

## Pinning

- k3s: `v1.37.2+k3s1`
- ArgoCD: `v3.6.1`
- tailscale-operator chart: `1.104.4`

## Provision (day-0)

```bash
ansible-galaxy collection install -r ansible/requirements.yaml
# put GitHub deploy key (private part) at ansible/keys/home-config.deploykey
ansible-playbook ansible/playbook.yaml -i ansible/inventory.yaml
```

Ansible does not manage the cluster afterward. Everything else is a commit in
`k8s/apps/` — including ArgoCD upgrades (edit the pinned version) later.

## Access (workstation)

1. Install Tailscale, join tailnet
2. `tailscale configure kubeconfig https://jojee.iptime.org` — authenticates
   via tailnet identity against the operator's API proxy (no copied
   kubeconfig). RBAC: `ClusterRoleBinding` for your tailnet user, see
   ticket 7525538279718916096.
3. ArgoCD UI: `kubectl port-forward -n argocd svc/argocd-server 8080:80`
   (until exposed via Tailscale), `argocd admin initial-password`.

## Backups

Not set up by choice for now. Single-node k3s keeps everything in
`/var/lib/rancher/k3s/server/` (SQLite + TLS); revisit later. If the cluster
is lost, the Ansible playbook can rebuild it, and Git holds all app config.
