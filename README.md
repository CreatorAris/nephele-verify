<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/CreatorAris/CreatorAris/dist/github-snake-dark.svg" />
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/CreatorAris/CreatorAris/dist/github-snake.svg" />
  <img alt="github contribution snake animation" src="https://raw.githubusercontent.com/CreatorAris/CreatorAris/dist/github-snake.svg" />
</picture>

# Nephele Verify

Independent verification page for [Nephele Workshop](https://nephele.arisfusion.com) digital evidence (`.nep`). Single-file HTML, no build step, no server, no telemetry.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![No Build](https://img.shields.io/badge/build-none-green.svg)](#self-host)
[![Mirrorable](https://img.shields.io/badge/mirrorable-yes-purple.svg)](#self-host)
[![GitHub stars](https://img.shields.io/github/stars/CreatorAris/nephele-verify.svg)](https://github.com/CreatorAris/nephele-verify/stargazers)
[![GitHub last commit](https://img.shields.io/github/last-commit/CreatorAris/nephele-verify.svg)](https://github.com/CreatorAris/nephele-verify/commits)

[中文文档](README_ZH.md) · [verify.arisfusion.com](https://verify.arisfusion.com)

</div>

## What this is

This repository is the source of [verify.arisfusion.com](https://verify.arisfusion.com), the page that verifies `.nep` evidence packages produced by Nephele Workshop.

It is published — open, MIT, single static file — for three reasons:

1. **Algorithm transparency.** Anyone can read the verification logic and confirm it matches the [Technical Audit Document](https://nephele.arisfusion.com/docs/security/audit).
2. **Mirror tolerance.** The verifier must remain runnable even if `arisfusion.com` ever disappears. Anyone can host their own.
3. **Offline / chain-of-custody use.** Courts, lawyers, and archives can run the verifier offline by opening `index.html` directly from disk.

## What it verifies

A `.nep` package is a ZIP container with a manifest, file hashes, an optional Merkle proof, and an RFC 3161 timestamp token. The page checks all of the following client-side, in the browser:

- **File hashes** — recomputes SHA-256 of every protected file and compares to the manifest.
- **Merkle root** — rebuilds the Merkle tree from leaf hashes and confirms the root.
- **TSA timestamp** — parses the binary RFC 3161 token (ASN.1 / DER), extracts `genTime` and the issuer, and confirms the digest inside the token matches the Merkle root.
- **URL evidence packages** — verifies the screenshot hash, captured HTML hash, TLS certificate snapshot, and the timestamp token over the URL evidence root.

All cryptography runs in the browser via the Web Crypto API. JSZip is loaded from a CDN to read the ZIP container. **No file content is uploaded anywhere.**

## What it does not verify

- The TSA's certificate chain is **not** validated against a root store. Use OpenSSL `ts -verify` against the original `.tsa` token for chain-of-trust validation; the page surfaces enough information (issuer name, `genTime`, hashed digest) to do this offline.
- A valid timestamp does not prove the file content existed *before* the timestamp — only that *some* digest was timestamped at the time the TSA asserts.

## Self-host

There is no build step, no environment variables, no configuration.

```bash
# Static file. Anywhere that serves HTML works.
cp index.html /var/www/your-domain/

# Or open it directly off disk:
open index.html       # macOS
start index.html      # Windows
xdg-open index.html   # Linux
```

The only outbound request is the JSZip CDN load (`cdnjs.cloudflare.com`). For a fully offline build, replace the `<script src=...>` tag with a vendored copy of `jszip.min.js`.

## Architecture

```
.nep ZIP file
    -> JSZip extracts manifest.json + file hashes + (proof.tsa | proof.json)
    -> Web Crypto API recomputes SHA-256 over each file
    -> Merkle tree rebuilt and root compared
    -> ASN.1 / DER parser unpacks the TSA token
    -> Render verdict + per-component status
```

All in one HTML file. Inspect with view-source.

## Reporting issues

Bugs in the verifier — false positives, false negatives, ASN.1 edge cases, i18n typos — file an issue here. PRs welcome; this repo is the source of truth for `verify.arisfusion.com`.

Feature requests for the broader Nephele Workshop product (the desktop client itself) — the client tree is closed-source, so file them via the contact address on the [website](https://nephele.arisfusion.com), not here.

## License

MIT, see [LICENSE](LICENSE). Free to fork, mirror, embed, modify.

## Related repositories

- [nephele-core-audit](https://github.com/CreatorAris/nephele-core-audit) — auditable subset of the Nephele Workshop client (rights / packer / validator)
- [nephele-wisp](https://github.com/CreatorAris/nephele-wisp) — browser-side companion (Chrome / Edge extension)
