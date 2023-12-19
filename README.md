# github-actions
to provide reusable github actions
actions can only be shared from public repositories
or from private repositories if you are on the Github Enterprise plan

## Table of Contents

- [Actions](#actions)
  - [Create Build Info](#create-build-info)
    - [Inputs](#inputs)
    - [How it works](#how-it-works)
    - [Example Usage](#example-usage)
  - [Checkout Repository on Deployment Server](#checkout-repository-on-deployment-server)
    - [Inputs](#inputs-1)
    - [How it works](#how-it-works-1)
    - [Example Usage](#example-usage-1)
  - [Setup SSH](#setup-ssh)
    - [Inputs](#inputs-2)
    - [How it works](#how-it-works-2)
    - [Example Usage](#example-usage-2)

## Actions
Yo need to define some environment variables and secrets for the action :

```
env:
      REPO_NAME: ${{ github.event.repository.name }}
      REPO_PARENT: /home/${{ secrets.DEPLOYMENT_SERVER_USER }}
      REPO_PATH: /home/${{ secrets.DEPLOYMENT_SERVER_USER }}/${{ github.event.repository.name }}
```
* DEPLOYMENT_SERVER_IP
* * the IP-address of the derver to deploy to
* DEPLOYMENT_SERVER_PUBLIC_KEY
* DEPLOYMENT_SERVER_SSH_PRIVATE_KEY
* DEPLOYMENT_SERVER_USER



### Checkout repo on deployment server

Located in [checkout-repository/action.yml](checkout-repository/action.yml)

This action checks out the repository on the deployment server.

#### Inputs

- `deployment-server-user`: Deployment server username. This is a required input.
- `deployment-server-ip`: Deployment server IP address. This is a required input.
- `github-token`: GitHub token. This is a required input.
- `repo-parent`: Parent directory of the repository on the deployment server. This is a required input.
- `repo-path`: Path to the repository on the deployment server. This is a required input.
- `repo-name`: Name of the repository. This is a required input.
- `github-ref`: GitHub reference (branch or tag). This is a required input.

#### Example Usage

Here's an example of how you can use the `Checkout repo on deployment server` action in a workflow:

```yaml
...
- name: Checkout repo on deployment server
  uses: stefan-isele/github-actions/checkout-repository@main
  with:
    deployment-server-user: ${{ secrets.DEPLOYMENT_SERVER_USER }}
    deployment-server-ip: ${{ secrets.DEPLOYMENT_SERVER_IP }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repo-parent: '/path/to/repo/parent'
    repo-path: '/path/to/repo'
    repo-name: 'repo-name'
    github-ref: 'main'
```

### Create Build Info

Located in [checkout-repository/action.yml](checkout-repository/action.yml)

This action creates a `build.info` file with data of the current build. You need to have a file `build-info.template` with placeholders for the actual data :

```json
{
  "repository": {
    "value": "${REPO_NAME}",
    "link": "${REPO_URL}"
  },
  "commit": {
    "value": "${COMMIT_NUMBER}",
    "link": "${COMMIT_URL}"
  },
  "time": "${BUILD_TIME}"
}
```

#### Inputs

- `deployment-server-user`: Deployment server username. This is a required input.
- `deployment-server-ip`: Deployment server IP address. This is a required input.
- `repo-path`: Path to the repository on the server. This is a required input.

#### How it works

1. It gets the current date and time.
2. It replaces placeholders on the server by copying the `build-info.template.json` to `build-info.json` on the server.

#### Example Usage

Here's an example of how you can use the `Create Build Info` action in a workflow:

```yaml
...
    - name: Create Build Info
      uses: stefan-isele/github-actions/create-build-info@main
      with:
        deployment-server-user: ${{ secrets.DEPLOYMENT_SERVER_USER }}
        deployment-server-ip: ${{ secrets.DEPLOYMENT_SERVER_IP }}
        repo-path: /path/to/repo/on/server
```

### Setup SSH

Located in [setup-ssh/action.yml](setup-ssh/action.yml)

This action sets up SSH for the runner.

#### Inputs

- `deployment-server-ip`: Deployment server IP address. This is a required input.
- `deployment-server-ssh-private-key`: Private SSH key for the deployment server. This is a required input.

#### How it works

1. It installs `ssh-keyscan` on the runner.
2. It creates a new directory for SSH if it doesn't exist.
3. It creates a new SSH key using the provided private key and sets the permissions for the SSH key to be read-only.
4. It adds the deployment server IP to the known hosts file.

#### Example Usage

Here's an example of how you can use the `Setup SSH` action in a workflow:

```yaml
steps:
- name: Setup SSH
  uses: stefan-isele/github-actions/setup-ssh@main
  with:
    deployment-server-ip: ${{ secrets.DEPLOYMENT_SERVER_IP }}
    deployment-server-ssh-private-key: ${{ secrets.DEPLOYMENT_SERVER_SSH_PRIVATE_KEY }}
```