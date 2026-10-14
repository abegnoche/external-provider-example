# external-provider-example

> **Example only - not a real provider.** The provider stubs return dummy
> success under `ISVCTL_DEMO_MODE=1` and fail otherwise. Kubernetes and Slurm
> scripts query real clusters and are not part of the demo checks.
> A passing demo run says nothing about any platform.
> This repository is not an NVIDIA-supported product.

A living example of externally maintained providers for the
[NVIDIA AI Cloud Validation Suite](https://github.com/NVIDIA/ai-cloud-validation).
It shows the shape a partner repository takes: the providers' `config/` and
`scripts/` live here, while the validation suite supplies the CLI, the suites,
and the validation code.

It holds two providers, `example` and `showcase`, one per directory, to show
that one repository can hold several (for example one per product or version),
each with its own registry entry. Both were generated with
`isvctl provider scaffold`, and pass against release 0.13.0.

## Reproducing the results

The demo suites exercise unimplemented provider stubs with
`ISVCTL_DEMO_MODE=1`; without it those stubs fail with "Not implemented".

From an ai-cloud-validation checkout with the provider registry
(`isvctl provider fetch`), fetch one by name and run it like any provider. Their
registry entries have `status: demo`, so isvctl turns demo mode on and never
uploads the dummy results:

```bash
uv run isvctl provider fetch example
uv run isvctl test run --provider example --suite vm
```

`showcase` works the same way.

Release 0.13.0 predates the registry: clone this repository next to the
checkout and pass the config path instead, from the checkout root. Here demo
mode must be set by hand, and `--no-upload` keeps the dummy results out of the
ISV Lab Service:

```bash
git clone https://github.com/NVIDIA/ai-cloud-validation.git
cd ai-cloud-validation
git checkout v0.13.0
uv sync
git clone https://github.com/abegnoche/external-provider-example.git ../external-provider-example

ISVCTL_DEMO_MODE=1 uv run isvctl test run --no-upload -f ../external-provider-example/example/config/vm.yaml
```

Some suites need a capability, mirroring the suite's `make demo-test`:

| Suite | Extra flags |
| --- | --- |
| `bare_metal`, `control-plane`, `iam`, `vm` | none |
| `network`, `observability`, `security` | `--capability vm` |
| `image-registry` | once with `--capability vm`, once with `--capability bare_metal` |

The [validation workflow](.github/workflows/validate.yml) checks this matrix for
both providers against the latest suite release. Kubernetes and Slurm require
real clusters. Storage requires `config/storage-provider-manifest.yaml`, which
neither example ships, so these three suites are excluded from the demo matrix.

## Layout

Each provider directory (`example/`, `showcase/`) contains:

- `config/` - YAML wiring: each file imports a suite and points its steps at `scripts/`.
- `scripts/` - the scaffold scripts, one directory per domain. See
  [`example/scripts/README.md`](example/scripts/README.md) for the script contract.

## Community and security

- [Contributing](CONTRIBUTING.md) - development, validation, pull requests, and DCO sign-off.
- [Code of Conduct](CODE_OF_CONDUCT.md) - community standards and reporting conduct issues.
- [Security](SECURITY.md) - private vulnerability reporting and execution assumptions.

This repository is currently hosted under `abegnoche`; a transfer to the NVIDIA
organization is planned. The clone URL above reflects its current location.

## License

[Apache-2.0](LICENSE)
