# opencode-epfl-rcp

[Docker Sandbox mixin kit](https://docs.docker.com/ai/sandboxes/customize/kits/)
that pre-configures [opencode](https://opencode.ai) to use the
[EPFL RCP AI inference-as-a-service](https://www.epfl.ch/research/facilities/rcp/ai-inference-as-a-service/)
endpoint (`https://inference-rcp.epfl.ch/v1`, OpenAI-compatible).

## What it does

- Installs a global opencode config at `/home/agent/.config/opencode/opencode.json`
  defining the `epfl-rcp` provider with a curated set of tool-capable models.
- Routes the `RCP_TOKEN` credential through the sandbox proxy: the proxy
  injects `Authorization: Bearer <token>` on requests to
  `inference-rcp.epfl.ch`, so the real token never enters the sandbox VM.

## Usage

Export the token on the host and load the kit at sandbox creation:

```bash
export RCP_TOKEN=<your-token>

# from a local checkout of this repository
sbx run opencode --kit ./config/sbx/opencode-epfl-rcp/

# or directly from git
sbx run opencode --kit "git+https://github.com/sdsc-ordes/agents#subdir=config/sbx/opencode-epfl-rcp"
```
