# Ansible Execution Environment Definition: Getting Started Guide Example

Congratulations on Your New Execution Environment Definition file!

This file tells **Ansible Builder** (the tool used to build EEs) how to create the build instruction file (`Containerfile` for **Podman**, `Dockerfile` for **Docker**) and the build context for your Execution Environment image.

An EE is a container image (like a "workspace") that bundles all the tools and collections your automation needs to run consistently everywhere.
This guide will walk you through the entire process, from reviewing your new files to using your final image in Ansible Automation Platform (AAP).

## Review What Was Generated

First, let us review the files that were just created for you:

- `ee12.yaml` - This is the main definition file that `ansible-builder` will use to construct your EE image.

- `template.yaml` - This is the Ansible portal template file that generated this. You can use it as a base to create new templates for your portal.
- `catalog-info.yaml` - This is the Ansible portal file that registers this Execution Environment definition as a "component" in your portal's catalog.


## Configuration

### Base Image
- **Base Image**: `registry.access.redhat.com/ubi9/python-311:latest`

### Ansible Collections (3)
```yaml
- name: ansible.posix
- name: ansible.mcp_builder
- name: ansible.mcp
```

### MCP Servers (1)
- `github_mcp`

## Get the Execution Environment definition file
#### Clone the repository and navigate to this directory.

```bash
git clone <repository URL>
cd ee-repo-1dfsd/ee12
```

Open `ee12.yaml` in your favorite text editor and review the configuration.

## Build Your Execution Environment (EE)

### Step 1: Install Required Tools

With your configuration ready, you'll need the following tools on your local machine to build the image:

- ansible-builder (The tool that builds the EE).
- A container engine: Podman (recommended) or Docker.

If you don't have them, you can find installation guides here:

1. [Installing ansible-builder](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/creating_and_using_execution_environments/assembly-using-builder)
2. [Installing Podman](https://podman.io/getting-started/installation) (recommended) OR [Installing Docker](https://docs.docker.com/engine/install/)

### Step 2 (Optional): Add additional configuration to the Execution Environment definition file

Before you build the EE, you must ensure `ansible-builder` can access all the collections you listed in your `ee12.yaml` file.
If the EE definition only uses collections from [Ansible Galaxy](https://galaxy.ansible.com/), you can safely **skip this step**.

If one or more collections specified in your EE definition are to be pulled from Automation Hub (or a custom Galaxy server),
please ensure that those servers are configured in an `ansible.cfg` file and included in the EE build.
You can refer to this documentation for more details: [Configure Red Hat Automation Hub as the primary content source](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html/getting_started_with_automation_hub/configure-hub-primary#proc-configure-automation-hub-server-cli)

For reference, here is an example of an `ansible.cfg` file that includes the Red Hat Automation Hub server:

```yaml
[galaxy]
server_list = automation_hub

[galaxy_server.automation_hub]
url=https://console.redhat.com/api/automation-hub/content/published/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
token=<SuperSecretToken>
```

Next, include the `ansible.cfg` file in your Execution Environment build by adding the following sections to the generated EE definition file:

```yaml
additional_build_files:
  - src: /path-to/ansible.cfg
    dest: configs

additional_build_steps:
  prepend_galaxy:
    - COPY _build/configs/ansible.cfg /etc/ansible/ansible.cfg
```

### Step 3: Build Your Execution Environment

Now you're ready to build! Open a terminal in the directory where the EE definition file exists and run the build command:

```bash
ansible-builder build --file ee12.yaml --tag ee12:latest --container-runtime podman
```

This command uses your `ee12.yaml` file to build an image and tags it as `ee12:latest`

- You can update the tag (specified after the `--tag` flag) to the desired tag for the built image.
- If you are using `Docker`, change the runtime with `--container-runtime docker`.

The `ansible-builder` CLI supports passing build-time arguments to the container runtime,
use the `--build-arg` flag to pass these arguments.

For the full list of supported flags, refer to the
[ansible-builder reference for `build arguments`](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/creating_and_using_execution_environments/assembly-using-builder)

### Step 4 (Recommended): Test Your Execution Environment Locally

This is the best way to verify your EE works before you share it. To do this, you can use `ansible-navigator`.

1. Install [`ansible-navigator`](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/using_content_navigator/assembly-intro-navigator_ansible-navigator):

`ansible-navigator` is a part of Ansible development tools. The can be installed on a container inside VS Code or from a package on RHEL.

Please refer to the following documentations for more details:

[Installing Ansible development tools on a container inside VS Code](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/using_content_navigator/installing-devtools#devtools-install-container_installing-devtools)

[Installing Ansible development tools from a package on RHEL](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/using_content_navigator/installing-devtools#devtools-install_installing-devtools)

2. Run your playbook with the built Execution Environment:
```bash
ansible-navigator run playbook.yml --execution-environment-image ee12:latest
```

If it runs successfully, your EE is working!

### Step 5: Push Your Execution Environment to a Container Registry

To use this EE in Ansible Automation Platform (AAP), it must exist in a container registry (like quay.io, or [private Automation Hub](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/managing_automation_content/managing-containers-hub)).

1. Tag your local image for the target registry (e.g., `quay.io/your-username/ee12:latest`).

```bash
podman tag ee12:latest quay.io/your-username/ee12:latest
```

2. Push the tagged image to the target registry:

```bash
podman push quay.io/your-username/ee12:latest
```

### Step 6: Use Your Execution Environment in Ansible Automation Platform (AAP)

You're all set! Now just tell AAP about your new EE.

1. In your AAP dashboard, navigate to Administration -> Execution Environments.
2. Click the *Add* button.
3. Fill in the fields:
  - **Name**: A friendly name (e.g., "My New Custom EE")
  - **Image**: The full path from Step 4 (e.g., quay.io/your-username/ee-new:latest)
  - **Pull**: Choose your pull policy (e.g., "Always pull image before running")
4. Click the *Save* button.

Your new EE can now be selected in any Job Template.

See the [Add a execution environment to a job template](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/using_automation_execution/assembly-controller-execution-environments) documentation for more details.
