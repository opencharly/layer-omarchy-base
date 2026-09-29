# AGENTS.md — layer-omarchy-base

Standalone candy repo for the Omarchy foundation layer — the package sources
every other `layer-omarchy-*` composes, the Omarchy runtime itself, and the
`/etc/skel` seeding a charly image needs. The repo is multi-candy: the root
`charly.yml` carries only the repo shape (`discover:`), and the member candies
live in `candy/<name>/charly.yml` — a candy's identity is its directory, so
members are never inlined into one manifest.

Canonical files:

- `charly.yml` — the repo shape (`repo:` + `discover:`).
- `candy/omarchy-base/charly.yml` — the meta candy and its
  `omarchy-base-skill:` skill entity (projected as
  `/charly-distros:omarchy-base`).
- `candy/omarchy-repo/charly.yml` — the pacman configuration candy.
- `candy/omarchy-runtime/charly.yml` — the `omarchy` + `omarchy-settings` candy.
- `candy/omarchy-skel/charly.yml` — the `/etc/skel` seeding candy.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-distros:omarchy-base` — the owning skill. The member composition, the
  mirror-snapshot mechanics, the limine alpm hook `NoExtract` rules, and why an
  Omarchy image installs a bootloader it never uses. Load before editing or
  troubleshooting the layer.
- `/charly-distros:omarchy` — the Omarchy base image built from this foundation.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, per-distro `distro:` arms, package/repo
  sections, and service declarations). Load before editing any entity field or
  plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- The members' `plan:` `check:` steps are the functional evidence — the meta's
  check asserts the runtime and seeded home coexist, and `omarchy-repo`'s checks
  assert the mirror, `[omarchy]`, `[multilib]` and the snapshot alignment.

## Modify this repo

- Members live in `candy/<name>/` subdirectories, **not** inline in one
  `charly.yml`: `ParseCandyManifest` returns the first `candy:` node in a
  manifest and drops the rest, so multiple candies in one file collapse to one
  named after the directory.
- Keep the pacman configuration in its own package-less candy (`omarchy-repo`):
  a candy's `plan:` steps are emitted *after* its `distro:` packages, so a candy
  that both configured pacman and installed from it would configure too late.
  Members that install packages `require:` it instead.
- Keep the five `NoExtract` rules for the limine alpm hooks — `/etc/pacman.d/hooks`
  masking cannot cover `90-mkinitcpio-install.hook` and leaves an unsilenceable
  failure behind.
- Edit the meta's `skill:` entity together with its candy entity; the skill is
  the projected usage source.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
