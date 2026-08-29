# layer-omarchy-base

The Omarchy foundation layer — the package sources every other `layer-omarchy-*`
composes, the Omarchy runtime itself, and the `/etc/skel` seeding a charly image needs.

[Omarchy](https://github.com/omacom/omarchy) is DHH's opinionated Linux distribution:
vanilla Arch + Hyprland, with its own package repository and its own pinned snapshot of
the Arch repositories.

## Using it

Pin **only the meta**, at its sub-path:

```yaml
candy:
    - '@github.com/opencharly/layer-omarchy-base/candy/omarchy-base:v<CalVer>'
```

The meta names its members by bare sibling name, and `QualifyRemoteSiblingDeps` rewrites
those to `.../candy/<member>` at this repo's own tag — so one pin pulls all three at a
matching version. Members are never pinned from outside; they are always installed
together.

| Member | Installs | Effect |
|---|---|---|
| `omarchy-repo` | nothing | Repoints the mirror, adds `[omarchy]` + `[multilib]`, imports the key, masks the limine hooks |
| `omarchy-runtime` | `omarchy`, `omarchy-settings` + the CLI tools they call | The Omarchy package tree and the ~200 `omarchy-*` commands |
| `omarchy-skel` | nothing | Copies `/etc/skel` into the image user's home |

## Layout

Members live in `candy/<name>/` subdirectories, **not** inline in one `charly.yml`. That
is not a style choice: a candy's identity **is** its directory. `ParseCandyManifest`
returns the *first* `candy:` node in a manifest and drops the rest, so three candies in
one file collapse to one named after the directory and the other two resolve as
`unknown candy`.

## Three things worth knowing

**The pacman configuration is a separate, package-less candy.** A candy's `plan:` steps
are emitted *after* its `distro:` packages, so a single candy that both configured pacman
and installed from it would configure too late. `omarchy-runtime` `require:`s
`omarchy-repo` instead.

**The mirror pin is load-bearing, and switching to it needs `-Syy`.** Omarchy publishes
its own snapshot of the Arch repositories and installs against that. Measured 2026-08-29,
`stable-mirror.omarchy.org` served `linux 7.1.9.arch1-2` while `geo.mirror.pkgbuild.com`
was on `7.1.11.arch1-1`. 125 of the 148 base packages come from `core`/`extra`/`multilib`,
so leaving the base image's Arch mirrorlist in place produces a *mixed* snapshot.

The refresh must be forced. The base image ships a sync DB already populated from upstream
Arch, and the snapshot's databases are deliberately *older* — `pacman -Sy` compares
freshness, keeps the newer local DB, and the repoint silently has no effect. The build
then resolves upstream versions and 404s fetching them from the snapshot. Observed
exactly that: the DB offered `qt6-base 6.11.2-3` while the mirror serves `6.11.2-2`.

Channel → mirror (upstream's own mapping; `edge` uses the *unprefixed* host, and there is
no `dev` channel):

| `OMARCHY_CHANNEL` | Arch repos | `[omarchy]` |
|---|---|---|
| `stable` (default) | `stable-mirror.omarchy.org` | `pkgs.omarchy.org/stable` |
| `edge` | `mirror.omarchy.org` | `pkgs.omarchy.org/edge` |
| `rc` | `rc-mirror.omarchy.org` | `pkgs.omarchy.org/rc` |

**An Omarchy image installs a bootloader it never uses.** `omarchy` hard-depends on
`limine`, `limine-mkinitcpio-hook`, `limine-snapper-sync`, `snapper` and `sddm`, so a
container gets the whole boot stack regardless. It is inert — the hooks are masked and no
unit is ever enabled.

`pacman -S --noconfirm --needed omarchy` **exits 0 anyway**, masked or not: pacman treats
a failed post-transaction hook as non-fatal. The four masks exist so an *expected*
`error: command failed to execute correctly` does not train everyone to ignore that line
in build logs.

**Never mask `90-mkinitcpio-install.hook`** — `limine-mkinitcpio-hook` *owns*
`/etc/pacman.d/hooks/90-mkinitcpio-install.hook`, so pre-creating it aborts the
transaction with `failed to commit transaction (conflicting files)`.

## charly vendors no Omarchy configuration

`omarchy` and `omarchy-settings` ship everything —
`/usr/share/omarchy/{bin,shell,themes,default,install,migrations}` and all of `/etc/skel`.
There is no copy of `config/hypr/*.lua`, no `themes/` tree and no `default/themed/*.tpl`
in this repo. `omarchy-skel` exists only because a charly image creates its user *before*
candies run, so `useradd`'s `/etc/skel` copy has already happened.

## License

MIT — see [LICENSE](LICENSE).
