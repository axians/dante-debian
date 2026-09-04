# Dante for Debian 13

This repository builds the Dante SOCKS client and server from the package in
Debian 14 (Forky) for Debian 13 (Trixie).

The current package is `dante 1.4.4+dfsg2-1`. It entered Debian unstable on
3 August 2026 and migrated to Debian testing, currently Forky, on 8 August
2026. The package source is rebuilt in Debian 13 so that the resulting binary
packages use the libraries and ABI available in Trixie.

This is an Axians build mirror, not an official Debian archive. Upstream's
original project introduction remains available in [README](README).

## Source provenance

- Debian package: <https://tracker.debian.org/pkg/dante>
- Debian packaging repository: <https://salsa.debian.org/debian/dante>
- Dante upstream: <https://www.inet.no/dante/>
- Axians releases: <https://github.com/axians/dante-debian/releases>

The Git history, Debian tags, upstream branch, and `pristine-tar` data were
mirrored from Salsa. The current package was imported from Salsa's `mrs/next`
branch after it was accepted into Debian.

The published Forky source and this repository's source were compared after
extraction. The upstream and Debian packaging contents are identical. A
locally regenerated `debian.tar.xz` can have a different compressed checksum
because archive metadata is regenerated, even when all extracted files are
identical.

## Branches

- `debian/master`: Axians build and release branch.
- `mrs/next`: imported Salsa package revision for `1.4.4+dfsg2-1`.
- `upstream`: repacked upstream source history.
- `pristine-tar`: data used to reproduce the upstream source archive.

Builds use only the public Axians GitHub mirror. Salsa availability is not
required while CI is running.

## Build and validation

The GitHub Actions workflow builds in a pinned `debian:13-slim` container on
an Ubuntu 24.04 runner. It performs the following steps:

1. Reconstructs the original source archive with `pristine-tar`.
2. Exports the package source without repository-only CI and backport files.
3. Installs build dependencies from `debian/control`.
4. Runs `dpkg-buildpackage --build=full --no-sign`.
5. Runs Lintian with errors treated as failures.
6. Runs the package's `adequate` autopkgtest.
7. Uploads binary packages, debug packages, and complete source metadata.

Actions and the Debian container image are pinned to immutable digests or
commit hashes. Only the release job receives `contents: write`, and only for
Debian release tags.

## Releases

A tag named `debian/<package-version>` triggers a fresh build and publishes a
GitHub Release after all checks pass. The tag version must exactly match the
version in `debian/changelog`.

Each release contains:

- `dante-server` and `dante-client` packages;
- runtime and development libraries;
- detached debug-symbol packages;
- `.dsc`, `.changes`, and `.buildinfo` metadata;
- original and Debian source archives;
- the autopkgtest summary; and
- `SHA256SUMS` for every uploaded build artifact.

Release packages and source metadata are unsigned. They are CI artifacts, not
a signed Debian APT repository.

## Verify a release

Download all assets from one release into an empty directory, then run:

```sh
sha256sum --check SHA256SUMS
```

Every listed file must report `OK` before installation.

## Install on Debian 13

For the Dante server on an `amd64` host:

```sh
sudo apt install ./dante-server_*_amd64.deb
```

For the `socksify` client, install its matching preload library at the same
time:

```sh
sudo apt install \
  ./libdsocksd0t64_*_amd64.deb \
  ./dante-client_*_all.deb
```

APT resolves the remaining dependencies from the configured Debian 13
repositories. Configuration and operation are documented in the installed
manual pages, including `danted(8)`, `danted.conf(5)`, and `socksify(1)`.

## Creating a release

After updating the package and confirming the branch build succeeds, create an
annotated tag whose value matches `debian/changelog`:

```sh
git tag --annotate 'debian/1.4.4+dfsg2-1' \
  --message 'Dante Debian release 1.4.4+dfsg2-1'
git push origin 'refs/tags/debian/1.4.4+dfsg2-1'
```

The tag build repeats the full package, Lintian, and autopkgtest pipeline
before GitHub publishes the release.

## License

Dante's license and copyright terms are in [LICENSE](LICENSE) and
[debian/copyright](debian/copyright).
