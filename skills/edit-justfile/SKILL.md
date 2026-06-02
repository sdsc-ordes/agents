---
name: edit-justfile
description: >-
  Creates and edits just recipe to document and automate standard operations in the repository.
  Includes recommended default settings and conventions for structure and naming.
---

# Top-level settings

* use `dotenv-load` if project needs a .env file
* use `set shell := ["bash", "-cue"]`
* enable `postitional-arguments`
* project variables


# Recipe creation

* Use shebang recipes (bash, nushell or python, depending on project preference) except for very simple recipes.
* forward *args when it makes sense.
* Ensure safe quoting.
* Use `[confirm("prompt")]` for destructive recipes.

# Secrets management

If the project requires secrets, aim for keeping secrets outside of the repository unless otherwise specified.

The standard practice should be to store path to external secret files in the .env file, for example:

```sh
#.env.tpl
KEYCLOAK_PASSWORD_FILE=
```

```sh
#.env
KEYCLOAK_PASSWORD_FILE=~/.config/keycloak/secret.txt
```

* no secrets in .env.
* justfile sources the .env
* gitignore `.env`; commit only `.env.tpl`.
* in recipes touching secrets: keep shell tracing off (no `set -x` / `bash -x`), pass values via stdin or process substitution (`--password-stdin`, `< "$FILE"`) rather than CLI arguments, and avoid `echo` of secret-bearing variables — all three surface secrets in CI logs or `ps` output.

NEVER read, write, or modify files outside the project — including secret files and their permissions. Let the user manage them. If you do read a secret, inform the user explicitly and recommend they revoke it.

# Conventions

## Structure

* justfile in repository root
* modules in tools/just/${module}.just
* default is to list recipes.
* put modules in [group('modules')]
* use private recipes to modularize logic when relevant.

# Naming

* Keep simple names for public recipes (e.g. just setup)
* Private recipes can have more specific names (e.g. just setup-commit-hooks)
* Prefer naming modules after functionality (e.g. just image::build, just secrets::edit)
