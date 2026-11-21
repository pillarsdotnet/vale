<!--
Copyright (c) 2026 Robert August Vincent II <pillarsdotnet@gmail.com>
Co-author: Claude Code.
-->

# vale, packaged for Ubuntu

[`Vale`](https://vale.sh) is a syntax-aware linter for prose. No Ubuntu or
Debian archive carries it, and upstream publishes tarballs only, so this
repository builds it into a Debian package for the
[package registry](https://forgejo.bobbalisa.org/pillarsdotnet/-/packages)
on `forgejo.bobbalisa.org`.

`debian/rules` clones the release tag that `debian/changelog` names, checks
that `HEAD` is the commit in `debian/upstream-commit`, bumps the modules
listed in `GO_MODULE_PINS`, builds with the Go toolchain `GOTOOLCHAIN`
names, and installs the result as `/usr/bin/vale`.

## Why it is not a repack

Until 3.28.1 this package downloaded the upstream Linux release binary and
repacked it, which is less code and less to go wrong. It was given up
because a release binary cannot be remediated.

A static Go binary carries the toolchain and the module versions whoever
built it happened to have, and `trivy` reads both out of the binary. So the
advisory count against a pinned release only ever goes up, without the
binary changing at all:

| binary | findings |
| --- | --- |
| 3.26.0 as released, scanned 4 November | 0 |
| 3.26.0 as released, scanned 20 November | 58, 23 of them high |
| 3.28.1 as released | 2 |
| 3.28.1 built here | 0 |

The 58 are almost all in the standard library of Go 1.25.7. Taking a newer
upstream release clears most of what accumulates -- which is the ordinary
reason to bump -- but not all: 3.28.1 is built with Go 1.27.3, which carries
`CVE-2026-97054`, and against `x/crypto` 0.58.0, which still ships the
`openpgp` package that `GO-2026-5932` says is unmaintained and unsafe by
design. Only a rebuild clears those two.

So the three pins in `debian/rules` are the package: the upstream tag, the
Go toolchain, and the modules whose released versions carry advisories.
Scan after every bump -- a new toolchain or a new upstream release can add
one as easily as clear one.

## Taking a new upstream release

Two files, and then a scan:

```sh
v=3.28.2   # the new upstream version
git ls-remote https://github.com/vale-cli/vale.git "refs/tags/v$v^{}"
# put that commit in debian/upstream-commit
dch -v "$v-1" -D UNRELEASED "Build the upstream $v release."
dpkg-buildpackage -b -us -uc
trivy rootfs --scanners vuln debian/vale/usr/bin
```

A finding at that point is a module or toolchain pin to raise, not a reason
to stop: raise it in `debian/rules`, rebuild, scan again. Check
<https://go.dev/dl/> for the current Go patch release at the same time --
most of what a Go binary ever scans with lives in the standard library.

Push to `main`, and the Registry workflow builds and uploads the package
for every supported series. The series suffix
(`~ppa1~ubuntu<release>.1`) is added at build time by
`.forgejo/registry-upload` and never committed.

## Licence

`Vale` itself is published under the Expat (MIT) licence, and the packaging
follows it; see [`debian/copyright`](debian/copyright).
