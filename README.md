# omarchy-base

The Omarchy foundation layer for charly images — the package sources every other
`layer-omarchy-*` composes, the Omarchy runtime itself, and the `/etc/skel`
seeding a charly image needs.

[Omarchy](https://github.com/omacom/omarchy) is DHH's opinionated Linux
distribution: vanilla Arch + Hyprland, with its own package repository and its
own pinned snapshot of the Arch repositories. This layer makes that distribution
available inside a charly-built image.

## Using it

Pin **only the meta**, at its sub-path:

```yaml
candy:
    - '@github.com/opencharly/layer-omarchy-base/candy/omarchy-base:v2026.242.0701'
```

The meta names its members by bare sibling name, and `QualifyRemoteSiblingDeps`
rewrites those to `.../candy/<member>` at this repo's own tag — so one pin pulls
all three at a matching version. Members are never pinned from outside; they are
always installed together.

| Member | Installs | Effect |
|---|---|---|
| `omarchy-repo` | nothing | Repoints pacman at Omarchy's pinned mirror, adds `[omarchy]` + `[multilib]`, imports the key, keeps the limine hooks off the image |
| `omarchy-runtime` | `omarchy`, `omarchy-settings` + the CLI tools they call | The Omarchy package tree and the ~200 `omarchy-*` commands |
| `omarchy-skel` | nothing | Copies `/etc/skel` into the image user's home |

## Choosing a channel

`OMARCHY_CHANNEL` selects which Omarchy snapshot the image installs against:
`stable` (the default), `edge`, or `rc`. There is no `dev` channel.

| `OMARCHY_CHANNEL` | Arch repos | `[omarchy]` |
|---|---|---|
| `stable` (default) | `stable-mirror.omarchy.org` | `pkgs.omarchy.org/stable` |
| `edge` | `mirror.omarchy.org` | `pkgs.omarchy.org/edge` |
| `rc` | `rc-mirror.omarchy.org` | `pkgs.omarchy.org/rc` |

## Things worth knowing

**The mirror pin is load-bearing, and switching to it needs `-Syy`.** Omarchy
publishes its own snapshot of the Arch repositories and installs against that.
Measured 2026-08-29, `stable-mirror.omarchy.org` served `linux 7.1.9.arch1-2`
while `geo.mirror.pkgbuild.com` was on `7.1.11.arch1-1`. Because 125 of the 148
base packages come from `core`/`extra`/`multilib`, leaving the base image's Arch
mirrorlist in place produces a *mixed* snapshot. The refresh must be forced
(`-Syy`): the base image ships a sync DB already populated from upstream Arch,
and the snapshot's databases are deliberately older, so a plain `-Sy` keeps the
newer local DB and the repoint silently has no effect.

**An Omarchy image installs a bootloader it never uses.** `omarchy` hard-depends
on `limine`, `limine-mkinitcpio-hook`, `limine-snapper-sync`, `snapper` and
`sddm`, so a container gets the whole boot stack regardless. It is inert — the
alpm hooks that would drive `limine-entry-tool` are never extracted, and no unit
is enabled. `omarchy-repo` adds five `NoExtract` rules to `/etc/pacman.conf` so
the hook files are never written, because `limine-entry-tool` requires a
genuinely mounted FAT32 ESP that a container cannot have.

**charly vendors no Omarchy configuration.** `omarchy` and `omarchy-settings`
ship everything — `/usr/share/omarchy/{bin,shell,themes,default,install,migrations}`
and all of `/etc/skel`. There is no copy of `config/hypr/*.lua`, no `themes/`
tree and no `default/themed/*.tpl` in this repo. `omarchy-skel` exists only
because a charly image creates its user *before* candies run, so `useradd`'s
`/etc/skel` copy has already happened.

## Layout

- `charly.yml` — repo shape: the `discover:` rule that finds the member candies.
- `candy/omarchy-base/` — the meta candy + its `skill:` entity.
- `candy/omarchy-repo/` — the pacman configuration candy.
- `candy/omarchy-runtime/` — the `omarchy` + `omarchy-settings` candy.
- `candy/omarchy-skel/` — the `/etc/skel` seeding candy.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-distros:omarchy-base` — the package sources, the
  mirror-snapshot mechanics, the limine-hook `NoExtract` rules, and the
  `/etc/skel` seeding.
- Base image: `/charly-distros:omarchy`.
- Derived layers: the sibling `opencharly/layer-omarchy-*` repos.
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella.
