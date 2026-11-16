# ansible-bootstrap

Ansible playbooks that set up an Ubuntu VPS with k3s, Argo CD and Sealed Secrets.
After that, Argo CD runs the applications from the GitOps repository, and this repo stops touching them.

Run the playbooks from WSL or Linux. The VPS does not need Ansible.

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

- The Sealed Secrets key backup lives in `~/.sealed-secrets/`. Never commit it.
- Deleting the `argocd-applications` ApplicationSet also deletes its applications.
