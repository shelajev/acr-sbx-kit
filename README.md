# ACR Docker Sandboxes kit

This mixin installs [Agentic Context Registry (ACR)](https://github.com/jbaruch/agentic-context-registry), a package manager for AI coding-agent instructions. ACR can materialize GitHub-hosted rules, skills, scripts, and hooks for Claude Code, Codex, and Cursor into the sandbox workspace.

The default ACR version is pinned and its release archive is checked against the publisher's `checksums.txt` before installation. A GitHub credential is optional: public repositories work without one, while a token enables private repositories, higher API limits, and package publishing. The real token remains in Docker Sandboxes' host-side credential proxy.

## Usage

Once published as an OCI kit:

```bash
sbx run codex --kit docker.io/sbx/acr-kit:latest .
```

Directly from this Git repository (pin `ref` to a tag or full commit for reproducible use):

```bash
sbx run codex --kit "git+https://github.com/shelajev/acr-sbx-kit.git#ref=main" .
```

From a local checkout:

```bash
sbx run codex --kit ./acr-sbx-kit .
```

The mixin is agent-independent, so `codex` can be replaced by another Docker Sandboxes agent.

To install a different ACR release, pass the version without a leading `v`:

```bash
sbx run codex --kit ./acr-sbx-kit --kit-arg acr.version=0.2.1 .
```

## Using ACR

Inside the sandbox, initialize a project and install a package:

```bash
acr install github:OWNER/REPO \
  --agent codex \
  --freshness outdated \
  --non-interactive
acr realize
```

Use `--agent claude-code`, `--agent codex`, and/or `--agent cursor` to select the layouts ACR should maintain. `acr install` updates dependency state; `acr realize` writes the selected agents' files. Run `acr help COMMAND` for all options.

## GitHub authentication

For public packages, no credential is required. For private packages, increased GitHub API limits, or `acr publish`, bind a GitHub token to the kit's optional `acr-github` service using Docker Sandboxes' credential setup when prompted. The distinct service name avoids colliding with an agent's own GitHub credential declaration. The kit exposes only the `proxy-managed` sentinel as `GH_TOKEN`; the proxy injects the real token into requests to `api.github.com` and `uploads.github.com`.

## Network policy

The kit allows only the GitHub hosts needed to download and verify ACR releases and for ACR to resolve, download, and publish packages:

- `github.com`
- `release-assets.githubusercontent.com`
- `api.github.com`
- `codeload.github.com`
- `objects.githubusercontent.com`
- `uploads.github.com`

## Verify the kit

```bash
sbx kit validate .
sbx kit inspect . --json
```

For a smoke test after sandbox creation:

```bash
acr version
acr help install
```

## License

Apache-2.0. See [LICENSE](LICENSE).
