# ansible-bootstrap

Playbooks to set up an Ubuntu VPS with k3s, Argo CD and Sealed Secrets.

Ansible runs from our computer or from WSL and connects to the VPS over SSH.
The VPS does not need Ansible installed.

## Structure

```text
vars/        connection, server and application configuration
roles/       tasks for each part of the server
```

## Playbooks

- `site.yaml`: sets up the whole VPS. It can be run again.
- `kubeconfig.yaml`: copies the kubeconfig to our computer.
- `check.yaml`: shows the status of the node and Argo CD applications.

## Before starting

We need:

- an Ubuntu VPS;
- SSH access with a key;
- Ansible installed in WSL or Linux;
- a deploy key for `helm-charts-wrappers` if the repository is private.

The playbooks connect to the SSH alias `vps` from `~/.ssh/config`, so the VPS
address stays outside the project. Variables are split by responsibility:

```yaml
# vars/connection.yaml
ansible_host: vps
ansible_user: ubuntu
```

The SSH config contains the VPS address and the local key path. It is not part
of this repository.

The playbooks load only the variable files they need:

- `site.yaml`: connection, server, Argo CD, Sealed Secrets and kubeconfig;
- `check.yaml`: connection and Argo CD;
- `kubeconfig.yaml`: connection and kubeconfig.

If `helm-charts-wrappers` is private, create a deploy key and add it on
GitHub:

```bash
ssh-keygen -t ed25519 -N "" -C "argocd" \
  -f ~/.ssh/helm_charts_wrappers_deploy
```

Add the `.pub` file under **Deploy keys** in the repository.

## First deployment

From WSL, go to this folder and install Ansible and the required collection:

```bash
sudo apt update
sudo apt install -y ansible
ansible-galaxy collection install -r collections/requirements.yaml
```

Check the playbook syntax first:

```bash
ansible-playbook site.yaml --syntax-check
```

If the syntax check passes, run the bootstrap:

```bash
ansible-playbook site.yaml
```

## What it installs

- automatic updates and basic Ubuntu configuration;
- SSH access using a key, with direct `root` login disabled;
- UFW with ports 22, 80 and 443 open;
- k3s;
- Sealed Secrets;
- Argo CD.

Port 6443 is not exposed to the Internet. To access the Kubernetes API:

```bash
ssh -N -L 6443:127.0.0.1:6443 vps
export KUBECONFIG=~/.kube/vps.yaml
kubectl get nodes
```

The Sealed Secrets key backup is stored outside the repository in
`~/.sealed-secrets/`. Do not commit it to Git.

To check the cluster status at any time:

```bash
ansible-playbook check.yaml
```
