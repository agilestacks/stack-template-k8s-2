# Development with Stack Template

## Toolbox

Infrastructure as Code relies on quite a bit of tools available locally, such as: Terraform, Helm, kubectl, AWS CLI, jq, Direnv, Halyard for Spinnaker. We provide Docker image - a **Toolbox** `agilestacks/toolbox` with the tools already installed.

To lunch the Toolbox call `hub toolbox`. It will set the container up to mount HOME and current working directory so you could start deploying right away.

## Hub CLI

To download Hub CLI binaries for deployments outside of Toolbox run:

    curl -O https://controlplane.agilestacks.io/dist/hub-cli/hub.darwin_amd64
    curl -O https://controlplane.agilestacks.io/dist/hub-cli/hub.linux_amd64

for MacOS and Linux respectively.

Rename to `hub`, `chmod +x`, move into `$PATH`. Install extensions:

    hub extensions install

## Makefile

There are a number of targets in Makefile:

- `deploy` to deploy complete stack. You may want to add `-o` to `hub` command to start re-deployment with specific component, e.g. `HUB_OPTS='-o traefik'`.
- `deploy COMPONENT=` to re-deploy specific stack component(s) by name, e.g. `make deploy COMPONENT=postgresql,redis`.
- `kubeconfig` to generate and save Kubeconfig context to access Kubernetes stack instance.
- `explain` and `explain-oplog` to review deployed stack parameters and outputs, operations history.
- `sync` to synchronize stack state from Hub CLI state file to SuperHub to keep UI information up-to-date.

## Environment

Each stack deployment via SuperHub gets a new file under `envrc/`. Use [direnv] to setup OS environment the Makefile and automation scripts refers to.

Add to `~/.bashrc`:

    eval "$(direnv hook bash)"

Add to `.envrc`:

    export AWS_PROFILE=<profile to access cloud account>
    source envrc/stack-name.cloud-account-domain.superhub.io

Secret parameter values are not stored in plain-text but are sourced from OS environment instead. When stack instance is deployed via SuperHub the values are populated by backend automation task. You should inspect the Environment and set Okta application secret:

    export DEX_OKTA_APP_CLIENT_SECRET=<randomn string>

Optionally, obtain and add `HUB_TOKEN`:

    $ hub login -u email@domain.tld
    export HUB_TOKEN=<long random string>

Hub CLI can fetch missing parameters from SuperHub API:

    export HUB_ENVIRONMENT=Dev1
    export HUB_STACK_INSTANCE=stack-name.cloud-account-domain.superhub.io

Set the environment:

    direnv allow

To see Environment secrets:

    hub api environment get --secrets

## Components

`components/` directory is a Git subtree of AgileStacks _components_ repository distribution branch. To track updates, add Components template remote as upstream:

    git remote add components-upstream \
        https://github.com/agilestacks/components.git

Then you'll be able to fetch and merge updates as they became available:

    git fetch components-upstream distribution
    git subtree pull --prefix components components-upstream distribution

Alternativelly, use `hub pull`.

Atlassian published a good introduction to [Git subtree].

## Template update

When Template is created via SuperHub it's content is initialized from a public Git repository hosted on GitHub. You can update template Makefile and support files by adding upstream remote, then fetching and merging changes as they became available:

    git remote add template-upstream \
        https://github.com/agilestacks/stack-template-eks.git
    git fetch template-upstream master

Inspect changes, merge:

    git merge template-upstream/master

## Stack development

Start by adding components to `hub.yaml` and parameters to `params/user.yaml`. Run `make elaborate`, then deploy with `make deploy COMPONENT=component-name`.

Or push the code into SuperHub Git repo and deploy complete stack template from there.


[direnv]: https://direnv.net
[Git subtree]: https://www.atlassian.com/blog/git/alternatives-to-git-submodule-git-subtree
