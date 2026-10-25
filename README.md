# ansible-bootstrap

Playbooks to take a fresh VPS to a working k3s cluster with Argo CD.
After that, everything on the cluster is managed from
[argocd-config](https://github.com/adrian-olias/argocd-config).

```
playbooks/   what you run
roles/       one role per piece of the server, each one a single tasks/main.yml
vars/        versions and settings, in one place
```

| Playbook | |
|---|---|
| `site.yml` | the whole bootstrap. Safe to run again |
| `kubeconfig.yml` | only copies the kubeconfig, e.g. on a new laptop |
| `check.yml` | read-only: nodes and Argo CD apps |

| Role | |
|---|---|
| `common` | upgrades packages, turns on security updates, SSH with keys only |
| `firewall` | ufw: only 22, 80 and 443 open. The API server stays closed |
| `k3s` | installs a pinned k3s version |
| `argocd` | installs Argo CD and creates the `root` Application |
| `kubeconfig` | copies the kubeconfig to the machine running Ansible |

## Before the first run

1. A VPS with Ubuntu LTS and your SSH key on it.
2. An `inventory.yml` at the root of the repo. It is git-ignored, since it has
   the IP of the server:

   ```yaml
   all:
     children:
       k3s_servers:
         hosts:
           vps:
             ansible_host: <vps-ip>
             ansible_user: ubuntu
             ansible_ssh_private_key_file: ~/.ssh/id_ed25519_vps
   ```

3. A read-only deploy key for argocd-config, while that repo is private:

   ```bash
   ssh-keygen -t ed25519 -N "" -C "argocd" -f ~/.ssh/argocd_config_deploy
   ```

   Add the `.pub` part under argocd-config → Settings → Deploy keys.

## Run

From Linux or WSL, at the root of the repo:

```bash
ansible-galaxy collection install -r requirements.yml
ansible-playbook playbooks/site.yml
```

If the repo lives on a Windows drive (`/mnt/c/...`), Ansible ignores
`ansible.cfg` there, so point to it: `export ANSIBLE_CONFIG=$PWD/ansible.cfg`.

## Reaching the cluster

Port 6443 is closed to the internet, so go through SSH:

```bash
ssh -N -L 6443:127.0.0.1:6443 ubuntu@<vps-ip> &
export KUBECONFIG=~/.kube/vps.yaml
kubectl get nodes
```

Argo CD UI:

```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d
```
