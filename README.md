# Koidra driver registry

Where Koidra hubs fetch their drivers. Releases only, no source: each driver is its own program,
published once per version and target, and a hub downloads the newest build that runs on its
machine and speaks its driver contract, then checks it against the sha256 listed here.

## Use it from a site

In the hub's `config.yaml`:

```yaml
driver_registry: https://github.com/Koidra/driver-registry
```

or for one run, `--driver-registry https://github.com/Koidra/driver-registry`. A section in `data_driver` names
the driver kind (`opcua`, `kplc`, `modbus_tcp`, …); a section can pin a version with
`driver_version: "0.1"` — without one the newest build is used.

The hub needs to reach `github.com` and `release-assets.githubusercontent.com` (downloads
redirect there). No token: this repository is public.

## Published drivers

| Kind | Version | Contract | Targets |
|---|---|---|---|
| `kplc` | 0.1.0 | 2 | `aarch64-apple-darwin`, `x86_64-pc-windows-gnu`, `x86_64-unknown-linux-gnu` |
| `mock` | 0.1.0 | 2 | `aarch64-apple-darwin`, `x86_64-pc-windows-gnu`, `x86_64-unknown-linux-gnu` |

## Layout

```text
releases/
├── index/index.json                                    read first by every hub
└── <kind>-v<version>/koidra-<kind>-<version>-<target>[.exe]
```

`index.json` maps kind → version → `contract_version` and, per target, the build's `sha256` and
`size_bytes`. A hub refuses a build whose digest does not match.

## Publishing

Not by hand. From the monorepo:

```bash
python3 iothub/tools/driver-registry/publish.py                # every driver, every target
python3 iothub/tools/driver-registry/publish.py --only opcua   # one driver
python3 iothub/tools/driver-registry/publish.py --dry-run      # build and show the index only
```

It builds each driver with its setup page, adds the builds to `index.json`, creates the
`<kind>-v<version>` releases, rewrites this README, and then downloads every listed build from the
URLs a hub computes to check them. A version that is already published is never replaced: a
different build under the same version stops the run, and the fix is to bump the driver's version.
