<!--
Copyright (c) 2026 Robert August Vincent II <pillarsdotnet@gmail.com>
Co-author: Claude Code.
-->

# vale, packaged for Ubuntu

[`Vale`](https://vale.sh) is a syntax-aware linter for prose. No Ubuntu or
Debian archive carries it, and upstream publishes tarballs only, so this
repository turns the upstream release binary into a Debian package for the
[package registry](https://forgejo.bobbalisa.org/pillarsdotnet/-/packages)
on `forgejo.bobbalisa.org`.

There is no source build here: the upstream Linux release is a static Go
binary, and `debian/rules` downloads the release that `debian/changelog`
names, checks it against the `sha256` recorded in `debian/upstream-sha256`,
and installs it as `/usr/bin/vale`.

## Taking a new upstream release

One commit, two files:

```sh
v=3.26.0   # the new upstream version
curl -fsSL "https://github.com/vale-cli/vale/releases/download/v$v/vale_${v}_checksums.txt" |
  grep Linux_64 > debian/upstream-sha256
dch -v "$v-1" -D UNRELEASED "Repack the upstream $v release binary."
```

Push to `main`, and the Registry workflow builds and uploads the package
for every supported series. The series suffix
(`~ppa1~ubuntu<release>.1`) is added at build time by
`.forgejo/registry-upload` and never committed.

## Licence

`Vale` itself is published under the Expat (MIT) licence, and the packaging
follows it; see [`debian/copyright`](debian/copyright).
