# OstojaOS module registry

Official catalog: **https://ostojaos.github.io/module-registry/**

This repository publishes a signed catalog and versioned ARM64 module packages. It does not contain the OstojaOS core source or private signing keys. The current panel supports installation from a downloaded `.ostojaos` archive; an in-panel internet catalog client is a separate integration.

## Available modules

- **Files 0.2.1** — file manager, folder trees, removable devices and file operations.
- **Terminal 0.2.1** — interactive system-user terminal with administrator permission checks.

Both require core `>=0.2.0,<0.3.0`, module API 1 and ARM64. OS package requirements are included in each signed manifest. Version 0.2.1 adds distribution notices and explicit native runtime dependencies; it does not require a core upgrade from 0.2.0.

## Install

Download the appropriate release archive from the catalog, then upload it in **Modules** in OstojaOS. The core checks module signatures, payload hashes, API/core compatibility, architecture and dependencies. Never install packages from an untrusted signer.

## Catalog protocol

- `catalog.json`: deterministic UTF-8 JSON with a trailing newline; `schemaVersion: 1`.
- `catalog.sig`: Ed25519 envelope with `algorithm`, `signer` and base64 `signature`. The signature covers the exact bytes of `catalog.json`.
- `modules[].releases[]`: signed module manifest, base64 manifest signature, immutable archive URL, SHA256, size and channel. All published versions remain available; releases are sorted newest first.
- Manifest signatures cover compact, sorted-key UTF-8 JSON **without** a trailing newline, matching the core package format.
- Clients must use a separately trusted, pinned public key; downloading a key from this site does not establish trust. `ostojaos-local` is the existing key identifier trusted by the initial OstojaOS deployment, retained for compatibility.
- A client must verify catalog signature, compatibility, archive length and SHA256, manifest signature, and every payload hash before installation. Registry metadata alone must never authorize installation or execution. Freshness and rollback policy must be handled by the future client.

## Publish a new release

Build and sign the module using the module packaging tools in the source workspace. Keep the private Ed25519 key offline; GitHub Actions receives no signing secrets.

```sh
python3 scripts/registry.py import-release /path/to/module-0.3.0-arm64.ostojaos
python3 scripts/registry.py sign --key /secure/path/module-signing-key.pem
python3 -m unittest discover -s tests -v
python3 scripts/registry.py verify
```

Create a draft GitHub Release tagged `<module-id>-v<version>`, upload the exact archive referenced in the new entry, then publish the release. Immutable releases are enabled. Only then merge the signed catalog update. Existing version entries and release assets must never be replaced; publish a new version instead. A future per-module repository can be selected with `import-release --repository OstojaOS/<repository>`.

```sh
python3 scripts/registry.py verify --online
python3 scripts/registry.py build --online
```

CI checks signatures, payload hashes, actual public downloads, catalog consistency and tests before deploying GitHub Pages. Action versions are pinned to commit SHAs. Pull requests cannot deploy; normal publication uses the main branch. Module compilation is separate from registry publication.

## License

Original registry software, catalog metadata and modules use **PolyForm Noncommercial 1.0.0**. This is a source-available noncommercial license, not an OSI-approved open-source license. Third-party components retain their licenses; module archives include LICENSE and NOTICE. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
