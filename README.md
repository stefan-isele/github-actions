# github-actions
to provide reusable github actions

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



### Checkout Repository on Deployment Server

Located in [checkout-repository/action.yml](checkout-repository/action.yml)

This action checks out the repository on the deployment server.

#### Inputs

- `ssh-private-key`: Private key for SSH. This is a required input.
- `server-ip`: Server IP address. This is a required input.
- `repo-name`: Name of the repository to checkout. This is a required input.

#### How it works

1. It creates a new SSH key using the provided private key.
2. It sets the permissions for the SSH key to be read-only.
3. It uses SSH to clone the repository from GitHub to the server.

#### Example Usage

Here's an example of how you can use the `Checkout repo on deployment server` action in a workflow:

```yaml
name: Deploy
on: [push]

jobs:
    build:
        runs-on: ubuntu-latest
        steps:
        - name: Checkout code
            uses: actions/checkout@v2

        - name: Checkout repo on deployment server
            uses: stefan-isele/github-actions/.github/actions/checkout-repository
            with:
                ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
                server-ip: ${{ secrets.SERVER_IP }}
                repo-name: 'my-repo'
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
name: Deploy
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Create Build Info
      uses: stefan-isele/github-actions/.github/actions/create-build-info
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
  uses: stefan-isele/github-actions/.github/actions/setup-ssh
  with:
    deployment-server-ip: ${{ secrets.DEPLOYMENT_SERVER_IP }}
    deployment-server-ssh-private-key: ${{ secrets.DEPLOYMENT_SERVER_SSH_PRIVATE_KEY }}
```