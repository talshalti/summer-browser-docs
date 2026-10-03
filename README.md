# Summer Browser Docs

This repository is the public documentation and verified-download surface for
Summer Browser. Summer Browser itself is developed in a private repository;
publishing documentation here does not publish or license the browser source.

The hosted Summer Docs site includes the product overview, user guidance,
Summer App documentation, website integration guidance, and release notes. The
`docs/` directory contains selected reference documents that have been approved
for public distribution.

Release binaries are attached to GitHub Releases only after their platform
release checks pass. Always verify the published SHA-256 checksums and the
platform signature before running a direct-download build.

Windows, macOS, and Android direct downloads use independent platform-version
tags such as `windows-v2.5.0`; they may publish different versions at different
times. iOS publication remains in Apple's App Store/TestFlight channel and is
linked from the documentation rather than attached as a public IPA.

## Documentation source

Files in this repository are exported from the exact private Summer Browser
commit recorded in `export-manifest.json`. Do not treat GitHub's automatically
generated source archives as Summer Browser source code; they contain only this
documentation repository.

## Rights and contributions

See [RIGHTS.md](RIGHTS.md). Public issues may report documentation mistakes,
but changes to the authoritative documentation are reviewed and exported from
the private product repository.
