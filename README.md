# Nephele Verify

Independent verification page for Nephele Workshop digital evidence (`.nep` files). Single-file HTML, no build step, no server, no telemetry.

This is the source of [verify.arisfusion.com](https://verify.arisfusion.com). It is published here so that:

1. Anyone can read the verification algorithm and confirm it matches the [Technical Audit Document](https://nephele.arisfusion.com/docs/security/audit).
2. Anyone can host their own mirror. The verification of a `.nep` evidence package must remain possible even if `arisfusion.com` ever disappears.
3. Third parties — courts, lawyers, archives — can run the verifier offline by opening `index.html` directly from disk.

## What it verifies

A `.nep` package is a ZIP container with a manifest, file hashes, an optional Merkle proof, and an RFC 3161 timestamp token. This page checks all of the following client-side, in the browser:

- **File hashes** — recomputes SHA-256 of every protected file and compares to the manifest.
- **Merkle root** — rebuilds the Merkle tree from leaf hashes and confirms the root.
- **TSA timestamp** — parses the binary RFC 3161 token (ASN.1 / DER), extracts `genTime` and the issuer, and confirms the digest inside the token matches the Merkle root.
- **URL evidence packages** — verifies the screenshot hash, the captured HTML hash, the TLS certificate snapshot, and the timestamp token over the URL evidence root.

All cryptography runs in the browser via the Web Crypto API. JSZip is loaded from a CDN to read the ZIP container. No file content is uploaded anywhere.

## How to host your own mirror

```
# Static file. Anywhere that serves HTML works.
cp index.html /var/www/your-domain/

# Or open it directly off disk:
open index.html       # macOS
start index.html      # Windows
xdg-open index.html   # Linux
```

That's it. There is no build step, no environment variables, no configuration. The only outbound request is the JSZip CDN load (`cdnjs.cloudflare.com`); if you want a fully offline build, replace that `<script src=...>` with a vendored copy of `jszip.min.js`.

## What this verifies — and what it does not

This page tells you whether a `.nep` file is internally consistent and whether its TSA token is well-formed. It does **not** validate the TSA's certificate chain against a root store, and it does **not** prove that the file content existed before the timestamp — only that *some* digest was timestamped at the time the TSA asserts.

For a chain-of-trust validation against the timestamping authority's root certificates, use OpenSSL's `ts -verify` against the original `.tsa` token; the page surfaces enough information (issuer name, `genTime`, hashed digest) to do this offline.

## License

MIT — see [LICENSE](LICENSE). Free to fork, mirror, embed, modify.

## Reporting issues

Bugs in the verifier — false positives, false negatives, ASN.1 edge cases, i18n typos — file an issue here. Feature requests for the broader Nephele Workshop product belong upstream.
