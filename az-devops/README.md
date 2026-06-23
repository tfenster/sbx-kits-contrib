# az-devops

A mixin that enables Azure DevOps API access from a sandbox by wiring
`dev.azure.com` and `*.visualstudio.com` requests through the sandbox
credential proxy.

Use it with any built-in agent or agent kit when your workflow needs to
call Azure DevOps REST APIs or MCP endpoints while keeping credentials
managed on the host.

## Prerequisites

- An Azure DevOps Personal Access Token (PAT).
- A sandbox secret `az-devops` available on your host before launching the sandbox.

Create the secret once with a base64-encoded `:<PAT>` value.

### Bash (Linux/macOS)

```console
$ printf ':%s' "$AZ_DEVOPS_PAT" | base64 | tr -d '\n' | sbx secret set -g az-devops
```

### PowerShell

```console
$ echo "$([System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes(":<your PAT>")))" | sbx secret set -g az-devops
```

## Usage

Run with any agent:

```console
$ sbx run --kit "git+https://github.com/docker/sbx-kits-contrib.git#dir=az-devops" <agent>
```

Or with a local clone of this repo:

```console
$ sbx run --kit ./az-devops/ <agent>
```

## How auth works

The kit declares Azure DevOps hosts in `network.serviceDomains` and maps
those hosts to the `az-devops` service. `network.serviceAuth.az-devops`
then tells the proxy to inject:

- `Authorization: Basic %s`

The `%s` value comes from a secret with the right name `az-devops`, created
in the way explained above.

The credential is managed as secret on the host so tools
inside the sandbox can make calls to Azure DevOps while the real
credential stays under proxy control for outbound requests.

## Network policy

The outbound allowlist is intentionally narrow and currently includes:

- `**.dev.azure.com`
- `**.visualstudio.com`
- `aka.ms`

If your workflow needs additional Azure hosts, fork this kit and extend
`network.allowedDomains`

## Install behavior

During `commands.install`, the kit checks whether
`/usr/local/share/npm-global/lib` exists and only creates it when missing.
The command is idempotent:

```sh
if [ ! -d /usr/local/share/npm-global/lib ]; then mkdir -p /usr/local/share/npm-global/lib; fi
```

## Cleanup

Remove the stored host secret when you no longer need it:

```console
$ sbx secret rm -g az-devops
```