# asdf-d2

[asdf-vm](https://asdf-vm.com) plugin for [D2](https://d2lang.com), the text-to-diagram
scripting language from [d2lang/d2](https://github.com/d2lang/d2) (formerly `terrastruct/d2`).

Installs the official prebuilt `d2` release binaries. Supports macOS and Linux,
amd64 and arm64. (D2 also ships Windows binaries, but asdf itself doesn't run
natively on Windows — use WSL, where this plugin works like any other Linux install.)

## Install

```shell
asdf plugin add d2 https://github.com/<your-username>/asdf-d2.git
asdf install d2 latest
asdf global d2 latest
d2 --version
```

While developing locally, before it's pushed anywhere:

```shell
asdf plugin add d2 /path/to/this/checkout
```

## Usage

```shell
asdf list all d2          # every published version
asdf install d2 0.7.1     # a specific version
asdf install d2 latest    # newest published release
asdf global d2 0.7.1      # set as the default
asdf local d2 0.7.1       # pin for the current project (.tool-versions)
```

## How it works

- **`bin/list-all`** reads tags from `git ls-remote` rather than the GitHub
  REST API, to avoid the API's 60 requests/hour unauthenticated rate limit
  (a real issue on shared CI runners).
- **`bin/latest-stable`** resolves GitHub's own `/releases/latest` redirect
  instead of just taking the newest tag. This matters for d2 specifically:
  its upstream sometimes pushes a version tag before that version's release
  binaries finish building, so "newest tag" and "newest downloadable
  release" briefly disagree. The redirect matches whatever GitHub itself
  considers the latest published release.
- **`bin/download`** maps `uname -s`/`uname -m` to the `macos|linux` /
  `amd64|arm64` naming d2's release assets use, downloads
  `d2-v<version>-<os>-<arch>.tar.gz`, and extracts it.
- **`bin/install`** copies the `d2` binary (and man page, if present) into
  `$ASDF_INSTALL_PATH`, then runs `d2 --version` as a sanity check before
  declaring success.

If you ever ask to install a version whose release build hasn't finished
yet, `bin/download` fails with an explanit message rather than a bare curl
error, and points you at `asdf list all d2` to pick another version.

## Testing changes

```shell
asdf plugin remove d2 || true
asdf plugin add d2 /path/to/this/checkout
asdf install d2 latest
asdf global d2 latest
d2 --version
echo 'x -> y' | d2 - out.svg && echo ok
```

## Publishing to the community plugin registry (optional)

To make `asdf plugin add d2` work without a URL, submit this repo to
[asdf-vm/asdf-plugins](https://github.com/asdf-vm/asdf-plugins) — add a
`plugins/d2` file containing `d2 https://github.com/<you>/asdf-d2.git` and
open a PR. Their CI will run this plugin's install/uninstall flow on Linux
and macOS before it's accepted, so make sure the smoke test above passes
cleanly first.
