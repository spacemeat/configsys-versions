# configsys-versions

The auto-derived **version-floor** data channel for [configsys](https://github.com/spacemeat/configsys).

A source-built component needs a minimum toolchain — ripgrep-from-source needs a Rust that clears
its MSRV, lazygit-from-source a Go that clears its `go.mod`. Those minimums drift as upstreams bump
them, faster than configsys or a recipe plugin releases. This plugin carries just those **floors**
(a `version-floors:` section), tracked on `main` and regenerated daily, so they stay current with
no release.

It ships **no code** and no recipes — only floor data — so there's no trust step.

## What it does

`version-floors:` *tightens* a component's existing cargo/go requirement (it never creates one), so
each floor lets configsys advise or select a toolchain that meets it:

```
$ configsys inspect         # (with a source binding pinned)
ripgrep … source            build needs cargo ≥1.85 — pin rust to rustup (rustup meets it)
```

A floor is **inert unless you build that component from source** — if you never pin a `via: source`
method, the floors just sit unused. That's why it's safe to load broadly and to pin to `main`.

## Use it

Two ways, both keeping it a visible, declared layer (never an auto-loaded surprise):

**Transitively** — if you use [configsys-source](https://github.com/spacemeat/configsys-source), it
already pulls this in (pinned to `main`), so you get fresh floors for free. Nothing to do.

**Directly** — if you pin a *core* source binding (fzf, btop, …) without configsys-source, add it to
`~/.config/configsys/configsys.hu`:

```
plugins: [
    { source: "github:spacemeat/configsys-versions"  ref: main }
]
```

then `configsys plugin sync`. Because it tracks `main`, re-sync periodically (a scheduled
`configsys plugin sync`, like a signature update) to pick up upstream MSRV/go.mod bumps.

## Regenerating (maintainer)

```
# with configsys-source loaded so its recipes' source bindings are visible:
tools/versionsweep.py --derive > floors.hu   # (wrap in the { } section; see the file header)
```

The derivation reads each source-buildable component's **built-tag** manifest (`Cargo.toml`
`rust-version`, `go.mod` `go`), so the floor matches the version you'd actually compile — not HEAD.

## Why separate from configsys-source

Recipes want **stability** (pin a tested tag); floors want **freshness** (track `main`). One plugin
at one ref can't give both. Splitting them lets you pin configsys-source to a release while floors
keep flowing on `main`.

## License

MIT © Trevor Schrock
