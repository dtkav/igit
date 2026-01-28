igit — inverse git
==================

igit is a patched build of git that inverts `.gitignore` semantics. It can
**only** track files that are ignored by a regular git repo in the same
directory. This lets you version-control private config, secrets, and
deployment files alongside a public project without leaking them.

## The problem

You maintain an open-source project but also have private files — deployment
configs, `.env` files, `fly.toml`, `docker-compose.override.yml`, etc. These
are gitignored so they don't end up in the public repo. But they contain
important configuration you don't want to lose, and you want version history
on them too.

## How it works

igit is a standalone git binary with three small changes:

1. **Default directory is `.igit/`** instead of `.git/`, so the two repos
   coexist without interfering.
2. **`core.invertExclude` defaults to `true`**, which flips the result of
   git's `is_excluded()` function. Files matched by `.gitignore` rules become
   visible; everything else is ignored.
3. **`igit init` recognizes `.igit/`** as a non-bare repository directory,
   so it auto-configures the worktree (just like git does with `.git/`).

Because the inversion happens inside git's own exclude engine, every command
works correctly — `status`, `add`, `diff`, `log`, `stash`, all of it. No
wrapper scripts, no generated ignore files, no drift.

## Usage

```bash
# Build from source
make -j$(nproc) NO_CURL=1 NO_EXPAT=1
cp git ~/bin/igit

# In any project with a .gitignore
cd my-project
igit init
igit add .
igit commit -m "track private configs"

# Add .igit to your regular .gitignore
echo .igit >> .gitignore
```

igit and git operate on completely separate repos (`.igit/` vs `.git/`) with
separate histories, branches, and remotes. You can push your igit repo to a
private remote independently.

## Patch summary

This is a minimal patch (4 files, ~15 lines changed) against git v2.53.0-rc2:

| File | Change |
|---|---|
| `environment.h` | Default dir `.git` → `.igit`; declare `core_invert_exclude` |
| `environment.c` | `core_invert_exclude = 1` default; config parser for `core.invertexclude` |
| `dir.c` | `is_excluded()` flips result when `core_invert_exclude` is set |
| `builtin/init-db.c` | `guess_repository_type()` recognizes `.igit` as non-bare |

## Configuration

The inversion can be toggled per-repo:

```bash
# Disable inversion (makes igit behave like regular git with a .igit dir)
igit config core.invertExclude false
```

## License

Same as git — GPLv2. See [COPYING](COPYING).

---

# Original git README

[![Build status](https://github.com/git/git/workflows/CI/badge.svg)](https://github.com/git/git/actions?query=branch%3Amaster+event%3Apush)

Git - fast, scalable, distributed revision control system
=========================================================

Git is a fast, scalable, distributed revision control system with an
unusually rich command set that provides both high-level operations
and full access to internals.

Git is an Open Source project covered by the GNU General Public
License version 2 (some parts of it are under different licenses,
compatible with the GPLv2). It was originally written by Linus
Torvalds with help of a group of hackers around the net.

Please read the file [INSTALL][] for installation instructions.

Many Git online resources are accessible from <https://git-scm.com/>
including full documentation and Git related tools.

See [Documentation/gittutorial.adoc][] to get started, then see
[Documentation/giteveryday.adoc][] for a useful minimum set of commands, and
`Documentation/git-<commandname>.adoc` for documentation of each command.
If git has been correctly installed, then the tutorial can also be
read with `man gittutorial` or `git help tutorial`, and the
documentation of each command with `man git-<commandname>` or `git help
<commandname>`.

CVS users may also want to read [Documentation/gitcvs-migration.adoc][]
(`man gitcvs-migration` or `git help cvs-migration` if git is
installed).

The user discussion and development of Git take place on the Git
mailing list -- everyone is welcome to post bug reports, feature
requests, comments and patches to git@vger.kernel.org (read
[Documentation/SubmittingPatches][] for instructions on patch submission
and [Documentation/CodingGuidelines][]).

Those wishing to help with error message, usage and informational message
string translations (localization l10) should see [po/README.md][]
(a `po` file is a Portable Object file that holds the translations).

To subscribe to the list, send an email to <git+subscribe@vger.kernel.org>
(see https://subspace.kernel.org/subscribing.html for details). The mailing
list archives are available at <https://lore.kernel.org/git/>,
<https://marc.info/?l=git> and other archival sites.

Issues which are security relevant should be disclosed privately to
the Git Security mailing list <git-security@googlegroups.com>.

The maintainer frequently sends the "What's cooking" reports that
list the current status of various development topics to the mailing
list.  The discussion following them give a good reference for
project status, development direction and remaining tasks.

The name "git" was given by Linus Torvalds when he wrote the very
first version. He described the tool as "the stupid content tracker"
and the name as (depending on your mood):

 - random three-letter combination that is pronounceable, and not
   actually used by any common UNIX command.  The fact that it is a
   mispronunciation of "get" may or may not be relevant.
 - stupid. contemptible and despicable. simple. Take your pick from the
   dictionary of slang.
 - "global information tracker": you're in a good mood, and it actually
   works for you. Angels sing, and a light suddenly fills the room.
 - "goddamn idiotic truckload of sh*t": when it breaks

[INSTALL]: INSTALL
[Documentation/gittutorial.adoc]: Documentation/gittutorial.adoc
[Documentation/giteveryday.adoc]: Documentation/giteveryday.adoc
[Documentation/gitcvs-migration.adoc]: Documentation/gitcvs-migration.adoc
[Documentation/SubmittingPatches]: Documentation/SubmittingPatches
[Documentation/CodingGuidelines]: Documentation/CodingGuidelines
[po/README.md]: po/README.md
