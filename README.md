# github-actions
to provide reusable github actions
## Actions
### Setup SSH

Located in [setup-ssh/action.yml](setup-ssh/action.yml)

This action sets up SSH for the runner.

#### Inputs

- `ssh-private-key`: Private key for SSH. This is a required input.
- `server-ip`: Server IP address. This is a required input.
- `known-host`: Known host key. This is a required input.

#### How it works

1. It creates a new directory for SSH if it doesn't exist.
2. It creates a new SSH key using the provided private key and sets the permissions for the SSH key to be read-only.
3. It adds the server IP and the known host key to the known hosts file.

### Example Usage

```yaml
name: Deploy
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Setup SSH
      uses: ./.github/actions/setup-ssh
      with:
        ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
        server-ip: ${{ secrets.SERVER_IP }}
        known-host: ${{ secrets.KNOWN_HOST }}
```

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
            uses: ./.github/actions/checkout-repository
            with:
                ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
                server-ip: ${{ secrets.SERVER_IP }}
                repo-name: 'my-repo'
```                

### Create Build Info

Located in [checkout-repository/action.yml](checkout-repository/action.yml)

This action creates a `build.info` file with data of the current build.

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
      uses: ./.github/actions/create-build-info
      with:
        deployment-server-user: ${{ secrets.DEPLOYMENT_SERVER_USER }}
        deployment-server-ip: ${{ secrets.DEPLOYMENT_SERVER_IP }}
        repo-path: /path/to/repo/on/server
```
