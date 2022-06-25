# ansible-roles-k0s

ansible-roles-k0s defines a set of roles that helps you on frequent operational work for k0s.

Unlike another standard alone ansible playbook repo's, where you clone their repo and duplicate your machine configs into their host file as well as aiming fo one-shot run, this repo is designed to:

- Have the roles integrated with your existing playbook
- Be reused for adding more controllers or workers in the future, as your fleet grows

## Installation

To install into your playbook, we can simply use git submodule to clone the repo.

Under your `<playbook>\roles` folder, e.g.: `D:\Code\ansible-demo\playbook\roles`, run:

```bash
git submodule add https://github.com/r12f/ansible-roles-k0s.git k0s
```

For older version of git, we might also need to run:

```bash
git submodule update --init --recursive
```

Since k0s require an initial controller to bootstrap, we need to define an extra variable for our initial controller:

```yaml
k0s_host_is_initial_master: true
```

This can be added into our inventory file as below:

```yaml
initial-master:
  hosts:
  r12f-lab-m0:
    ansible_host: 10.0.0.1
    k0s_host_is_initial_master: true
```

Or using `group_vars`, in this case, our group is initial-master, hence, we can define below in `group_vars/initial-master.yaml` file:

```yaml
k0s_host_is_initial_master: true
```

That's it! All roles are ready to go!

## Creating a k0s cluster

The high level process for creating a k0s cluster is:

1. Create a initial controller for the k0s cluster
2. Ask initial controller to generate the tokens for initializing the rest of controllers and workers.
3. Use the token to initialize the rest of controllers and workers.

Hence, our roles are also defined in this way.

### Step 1: Add roles to your playbook

To create a cluster, we can create a file in your playbooks folder with the following 2 roles, say: `setup-k0s.yaml`

```yaml
---
# Master nodes
- hosts:
    - initial-master
    - masters
  become: true
  gather_facts: true
  roles:
    - k0s/common
    - k0s/install
    - k0s/setup/controller

# Worker nodes
- hosts: workers
  become: true
  gather_facts: true
  roles:
    - k0s/common
    - k0s/install
    - k0s/setup/worker
```

### Step 2: Initialize initial controller

After the role is defined, we can use the following command to initialize the k0s initial controller.

```bash
ansible-playbook playbooks/setup-k0s.yaml --limit initial-master
```

Once all works are done, it will output the generated tokens on console for initialize other controllers and workers, making them connect to the initial controller and join the cluster.

### Step 3: Update tokens

Copy the tokens and add them to our variable files, like below.

```yaml
k0s_controller_join_token: H4sIA...
k0s_worker_join_token: H4sIA...
```

And for security reasons, preferrably these 2 values should be added into [ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html).

### Step 4: Update rest machines

Now, we can update the rest of the fleet by running:

```bash
ansible-playbook playbooks/setup-k0s.yaml --limit masters
ansible-playbook playbooks/setup-k0s.yaml --limit workers
```

If you are using ansible vault, and say the vault file is `secrets.yaml`, then, we can add `-e @secrets.yaml` in the end of the commands above.

You might also need `--ask-vault-pass` or update your `ansible.cfg` for password settings, to know more about ansible vault, please check the link [here](https://docs.ansible.com/ansible/latest/user_guide/vault.html).

## Uninstall / Reset

If we need to uninstall k0s from a node, we can use the reset role.

### Step 1: Add reset roles to your playbook

Same as install, we can create a playbook, say `reset-k0s.yaml`, and add the following role:

```yaml
---
- hosts: all
  become: true
  gather_facts: true
  roles:
    - k0s/common
    - k0s/reset
```

### Step 2: Run

Then we can uninstall k0s fom our machines by:

```yaml
ansible-playbook playbooks/reset-k0s.yaml --limit workers
ansible-playbook playbooks/reset-k0s.yaml --limit masters
ansible-playbook playbooks/reset-k0s.yaml --limit initial-master
```

## License

Apache-2.0: <https://www.apache.org/licenses/LICENSE-2.0>

## Others

If you don't really need ansible integration for your cluster or looking for one-shot solution, feel free to check these alternative solutions:

- [k0sctl](https://github.com/k0sproject/k0sctl): This is a really nice tool for setup a k0s clusters.
- [k0s-ansible](https://github.com/movd/k0s-ansible): A standard alone playbook that allow you setup your cluster in fully automated way in one shot.
