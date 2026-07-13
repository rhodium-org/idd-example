# idd-example

A worked example of a **composed** [throughline](https://github.com/rhodium-org/throughline)
project: an ordinary requirements graph that adopts a published standard **by
reference** and works the combined graph as one, using
[throughline-compose](https://github.com/rhodium-org/throughline-compose).

The scenario is a council-housing resident service. Its own product requirements
live in this repo; its authentication requirements each cite a clause of the OWASP
**ASVS**, pulled in from the [`standard-asvs`](https://github.com/rhodium-org/standard-asvs)
source under the namespace `asvs`.

## What it demonstrates

- **Adopt a standard by reference, not by copy.** The ASVS source is named in
  [`throughline.toml`](throughline.toml) as a git `url` pinned to an edition with
  `ref`. Nothing is vendored; `tl-compose` fetches the pinned source into a per-user
  cache outside this project and composes from it.
- **The composer picks the namespace.** This project binds the source to `asvs`;
  the source does not name itself. A borrowed clause is referenced as `asvs:SR-0003`.
- **Borrowed clauses are cited, never renumbered.** Each product
  `system_requirement` links to an ASVS clause with a `satisfies` link:

  | This project | satisfies | ASVS clause |
  |---|---|---|
  | `SR-0001` minimum password length | → | `asvs:SR-0003` (V2.1.1) |
  | `SR-0002` long passphrases, no truncation | → | `asvs:SR-0004` (V2.1.2) |
  | `SR-0003` reject breached credentials | → | `asvs:SR-0006` (V2.1.7) |

- **One tool, one guarantee.** `tl-compose check` merges the source into a single
  union graph and runs throughline's own validator over it. Bare `tl check` fails
  fast the moment it meets `asvs:SR-0003` — it cannot resolve a cross-source
  reference — and points you at `tl-compose`, so you never get a false clean result.

## Layout

```
throughline.toml            # project config + the [[sources]] declaration
intents/INT-0001.yml        # why this service exists
user-requirements/UR-0001   # residents authenticate securely
system-requirements/        # SR-0001..3, each satisfying an ASVS clause
```

## Running it

```sh
pip install "throughline-compose @ git+https://github.com/rhodium-org/throughline-compose@main"

tl-compose check --strict   # composes the pinned asvs source and validates the union
tl check                    # for contrast: fails fast on the unresolved asvs: reference
```

`tl-compose` fetches the pinned source over the network on first run and caches it;
subsequent runs are offline. CI runs the same `tl-compose check --strict`.

## Pinning

The source is pinned here to a specific commit of the `standard-asvs` thin v4.0.3
slice, because that slice has not yet cut an edition tag. The canonical form pins a
published edition tag instead:

```toml
[[sources]]
namespace = "asvs"
url = "https://github.com/rhodium-org/standard-asvs"
ref = "v4.0.3"
```

## Licence

Apache-2.0. See [LICENSE](LICENSE).
