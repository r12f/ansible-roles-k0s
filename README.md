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

That's it! All roles are ready to go!

## Creating a k0s cluster

The high level process for creating a k0s cluster is:

1. Create a initial controller for the k0s cluster
2. Ask initial controller to generate the tokens for initializing the rest of controllers and workers.
3. Use the token to initialize the rest of controllers and workers.

Hence, our roles are also defined in this way.

### Step 1: Update the vars for your machines

```yaml
k0s_host_is_initial_master: true
```

### Step 2: Add roles to your playbook

### Step 3: Play

#### Step 3.1: Initialize initial controller

#### Step 3.2: Update tokens

#### Step 3.3: Update rest machines

## Reset
