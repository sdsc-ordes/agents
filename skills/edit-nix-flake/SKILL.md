---
name: edit-nix-flake
description: >-
  Creates and edits nix flakes that provide a project's devshells,
  packages, and tooling wrappers. Triggers when adding, editing, or
  refactoring a `flake.nix` or other `.nix` file. Encodes layout
  and architecture conventions. Skip when only running tooling through
  an existing devshell.
---

# Edit Nix Flake

## When To Use

Apply this skill whenever creating or modifying a `flake.nix`, a
`shells/*.nix` devenv module, or a per-package `.nix` file. For invoking
tooling through an existing devshell, use the `run-nix-devshell` skill.

Local references in this skill:

- `references/example-flake.md` — a complete annotated layout, usable
  as the bootstrap template.
- `references/devenv-shells.md` — devenv module patterns for
  language-specific shells.
- `references/packages.md` — custom package and wrapper patterns
  (treefmt-nix as the worked example).

## Step 1: Choose The Layout

When starting from scratch, prefer placing all nix files under `tools/nix`.
If the project already uses a `flake.nix` at the root, only move it to `tools/nix`
if the complexity grows to warrant multiple files (devenv modules, package derivations, ...).

```
project-root/
└── tools/nix/
    ├── flake.nix
    ├── flake.lock
    ├── shells/<name>.nix    # devenv modules, one per specialized shell
    └── packages/<name>.nix  # per-package derivations and wrappers
```

`flake.lock` is tracked alongside `flake.nix`.

## Step 2: Set Up The Toplevel Structure

A flake.nix follows this skeleton (full annotated form in
`references/example-flake.md`):

```nix
{
  description = "<project> devshell";

  nixConfig = { /* cachix substituters and trusted keys */ };

  inputs = {
    nixpkgs.url     = "github:cachix/devenv-nixpkgs/rolling";
    flake-utils.url = "github:numtide/flake-utils";
    treefmt-nix     = { url = "github:numtide/treefmt-nix";
                        inputs.nixpkgs.follows = "nixpkgs"; };
    devenv          = { url = "github:cachix/devenv";
                        inputs.nixpkgs.follows = "nixpkgs"; };
  };

  outputs = { nixpkgs, flake-utils, devenv, ... }@inputs:
    let
      defineOutput = system: let
        pkgs = nixpkgs.legacyPackages.${system};
      in {
        devShells = { /* Step 3 */ };
      };
    in
    flake-utils.lib.eachDefaultSystem defineOutput;
}
```

- **`flake-utils.lib.eachDefaultSystem`** lifts the output across the
  common systems (Linux/macOS, x86/ARM).
- **`inputs.X.inputs.nixpkgs.follows = "nixpkgs"`** forces sub-inputs to
  share the project's nixpkgs revision, avoiding duplicate downloads
  and version drift.
- Include `devenv` only when relevant (language-specific setups). When not
  using devenv, use the latest nixpkgs instead (e.g. `nixpkgs.url = "github:NixOS/nixpkgs/nixos-26.05`).
- `treefmt-nix` is the recommended formatter. Refer to `references/packages.md` and the [official repository](https://github.com/numtide/treefmt-nix) to configure it.

## Step 3: Define A devShell

Add one shell per environment. Define the shared package list once in
the `let` block, then extend it per shell as needed:

```nix
let
  packagesBasic = with pkgs; [ git just jq ];
in
{
  devShells = {
    default = pkgs.mkShell {
      packages = packagesBasic ++ (with pkgs; [ kubectl kubernetes-helm ]);
    };

    ci = pkgs.mkShell {
      packages = packagesBasic;
      shellHook = "unset TMPDIR";
    };
  };
}
```

- **`default`** is the conventional name for the primary shell.
- **Specialized shells** (`ci`, `docs`, etc.) sit alongside `default`.
- **`shellHook`** runs on interactive shell entry only — not under
  `nix develop --command`. See the `run-nix-devshell` skill for the
  consequences when running.
- Sort packages alphabetically within each list to keep diffs minimal.

## Step 4: Component-Specific Shells With devenv

When a shell needs language-aware setup (Python+uv, Node+pnpm,
Rust+cargo), extract the configuration into `shells/<name>.nix` and
load it via `devenv.lib.mkShell`. See `references/devenv-shells.md` for
the module structure, the available language options, and the rule for
choosing devenv over plain `mkShell`.

## Step 5: Custom Packages And Wrappers

When a tool needs project-specific configuration (formatter rule sets,
patched derivations, CLI wrappers), define it in `packages/<name>.nix`
and import the resulting derivation into a package list. See
`references/packages.md` for the function signature and the treefmt
wrapper example.

## Operational Notes

- **ALWAYS** commit `flake.lock`. Treat it as source.
  Refresh it with `nix flake update` whenever modifying inputs.
- **ALWAYS** run `nix flake check`** after editing nix files.
- `flake-utils.lib.eachDefaultSystem` covers the four
  common targets. Use `flake-utils.lib.eachSystem [ ... ]` when the
  project needs a narrower or wider set.
