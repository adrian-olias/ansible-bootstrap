# ansible-bootstrap

Ansible playbooks that set up an Ubuntu VPS with k3s, Argo CD and Sealed Secrets.

Run them from WSL or Linux. The VPS does not need Ansible.

## Playbooks

- `site.yaml`: sets up the server. Safe to run again.
- `bootstrap-applicationset.yaml`: connects Argo CD to the GitOps repository.
- `kubeconfig.yaml`: copies the kubeconfig to your computer.
- `check.yaml`: shows the node and Argo CD status.

## Usage

The playbooks connect to the SSH host `vps` from `~/.ssh/config`.

```bash
ansible-galaxy collection install -r collections/requirements.yaml
ansible-playbook -i 'vps,' site.yaml
ansible-playbook -i 'vps,' bootstrap-applicationset.yaml
```

## Notes

- Port 6443 (Kubernetes API) is closed. Reach it with an SSH tunnel:
  `ssh -N -L 6443:127.0.0.1:6443 vps`
- The Sealed Secrets key backup lives in `~/.sealed-secrets/`. Never commit it.
- Deleting the `argocd-applications` ApplicationSet also deletes its applications.
