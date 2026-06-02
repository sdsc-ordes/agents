# SDSC Justfile Module Conventions

These conventions apply to repositories that follow the SDSC stack layout.
Apply a section only when the corresponding module is declared in the root
justfile (look for the matching `mod` line).

Every recipe listed below is destructive. State the exact command and wait
for user confirmation before running it, unless the user has already
requested the operation explicitly.

## `compose` Module

Wraps Docker Compose operations. Typical recipes:

- `compose::up [path]` — start a stack (defaults to `compose/`).
- `compose::down [path]` — stop a stack.
- `compose::all-up` / `compose::all-down` — operate on every stack in the
  repository.

## `secrets` Module

Wraps [SOPS](https://github.com/getsops/sops)-based secret management with age
keys. Destructive recipes: `secrets::encrypt`, `secrets::decrypt`,
`secrets::generate-key`.

### Key File

Recipes export `SOPS_AGE_KEY_FILE="./.sops.agekey"`. This key file must exist
in the project root before any encrypt or decrypt recipe will succeed. When it
is missing, run `just secrets::generate-key` to create it.

### Encrypted File Naming

Encrypted files use the `sops-` prefix, for example `sops-.env.secrets`. The
`encrypt` recipe adds the prefix on output; the `decrypt` recipe strips it.
Keep the prefix intact so the recipes continue to find the right files.
