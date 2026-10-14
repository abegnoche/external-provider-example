# example scaffold

Copy-and-fill-in scripts for adding your own platform to the validation suite.

The Python provider stubs ship with TODO blocks and two behaviors:

- **Default run** - exits with `"Not implemented - ..."`, making it obvious where to fill in your platform's API calls.
- **Demo mode** (`ISVCTL_DEMO_MODE=1`) - returns dummy-success JSON so the whole pipeline runs end-to-end without any cloud. Used by this repository's demo CI matrix.

## The three pieces that make this work

```text
suites/*.yaml                    <- contract   (what to validate; platform-agnostic)
                  │
                  ▼ imported by
example/config/*.yaml            <- wiring     (which scripts implement each step)
                  │
                  ▼ invokes
example/scripts/<domain>/*.py     <- scaffold   (generated for you; fill in TODO blocks)
```

The `suites/` layer lives in the AI Cloud Validation Suite checkout and defines
the validation contract. Import it from your provider config, then implement the
provider scripts against that contract.

## Domains

| Domain | Scripts | Contract | Provider YAML | AWS reference |
|--------|---------|----------|---------------|---------------|
| `iam/` | 3 | [`suites/iam.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/iam.yaml) | [`config/iam.yaml`](../config/iam.yaml) | [`providers/aws/scripts/iam/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/iam) |
| `control-plane/` | 11 | [`suites/control-plane.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/control-plane.yaml) | [`config/control-plane.yaml`](../config/control-plane.yaml) | [`providers/aws/scripts/control-plane/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/control-plane) |
| `vm/` | 12 | [`suites/vm.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/vm.yaml) | [`config/vm.yaml`](../config/vm.yaml) | [`providers/aws/scripts/vm/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/vm) |
| `bare_metal/` | 14 | [`suites/bare_metal.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/bare_metal.yaml) | [`config/bare_metal.yaml`](../config/bare_metal.yaml) | [`providers/aws/scripts/bare_metal/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/bare_metal) |
| `storage/` | 21 | [`suites/storage.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/storage.yaml) | [`config/storage.yaml`](../config/storage.yaml) | [`providers/aws/scripts/storage/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/storage) |
| `network/` | 24 | [`suites/network.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/network.yaml) | [`config/network.yaml`](../config/network.yaml) | [`providers/aws/scripts/network/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/network) |
| `observability/` | 5 | [`suites/observability.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/observability.yaml) | [`config/observability.yaml`](../config/observability.yaml) | [`providers/aws/scripts/observability/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/observability) |
| `image-registry/` | 7 | [`suites/image-registry.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/image-registry.yaml) | [`config/image-registry.yaml`](../config/image-registry.yaml) | [`providers/aws/scripts/image-registry/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/image-registry) |
| `security/` | 17 | [`suites/security.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/security.yaml) | [`config/security.yaml`](../config/security.yaml) | [`providers/aws/scripts/security/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/security), [`providers/aws/scripts/capacity/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/capacity) |
| `k8s/` | 9 shell | [`suites/k8s.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/k8s.yaml) | [`config/k8s.yaml`](../config/k8s.yaml) | [`providers/aws/scripts/eks/`](https://github.com/NVIDIA/ai-cloud-validation/tree/main/isvctl/configs/providers/aws/scripts/eks) |
| `slurm/` | 2 shell | [`suites/slurm.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/slurm.yaml) | [`config/slurm.yaml`](../config/slurm.yaml) | - |
| `breakfix/` | 14 | [`suites/bare_metal.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/bare_metal.yaml), [`suites/k8s.yaml`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/k8s.yaml) | [`config/bare_metal.yaml`](../config/bare_metal.yaml), [`config/k8s.yaml`](../config/k8s.yaml) | - |

The `k8s/` and `slurm/` examples drive a **real** cluster (validations shell out
to `kubectl` / `sinfo`), so they are excluded from the demo CI matrix.

> The `storage/` domain is the first to use the Storage
> Provider Shim (the [`StorageApi`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvtest/src/isvtest/core/storage_provider/api.py)
> ABC) instead of
> per-test-case scripts: one [`api.py`](storage/api.py) per backend
> subclasses `StorageApi` and a [provider manifest](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/providers/my-isv/config/storage-provider-manifest.yaml)
> declares which `api.py` to load. This repository does not ship that manifest;
> the link points to the upstream template, which must be adapted before storage
> runs. See [`storage/README.md`](storage/README.md) for the authoring guide.

See [`suites/README.md`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/README.md) for the per-step / per-field breakdown.

## Usage

**1. Preview the VM pipeline with no cloud:**

Run from an AI Cloud Validation Suite checkout beside this repository:

```bash
ISVCTL_DEMO_MODE=1 uv run isvctl test run --no-upload \
  -f ../external-provider-example/example/config/vm.yaml
```

See the [repository README](../../README.md#reproducing-the-results) for the
other demo suites and their capability flags. The upstream `make demo-test`
target tests upstream scaffolds, not your edits in this repository.

**2. Generate the scaffold and wiring under your provider name:**

```bash
uv run isvctl provider scaffold acme --output-dir ../external-provider-acme
```

Use `--dry-run` to preview the destination and next commands. Scaffolding
requires a validation-suite checkout that supports `isvctl provider scaffold`.

**3. Implement each script** - each has a `TODO:` block with pseudocode and a link to the AWS reference implementation.

**4. Run your implemented provider (no demo flag):**

From the validation checkout root, point at the generated provider config:

```bash
uv run isvctl test run --no-upload -f ../external-provider-acme/config/vm.yaml
uv run isvctl test run --no-upload -f ../external-provider-acme/config/storage.yaml --capability vm
```

Storage additionally needs a populated manifest; the examples here omit it.

Add `--dry-run` to any of these to list what would run and what would be
skipped, without executing a thing.

### Which checks run

Checks in a plain suite declare what they presuppose. `requires: []` (core) runs
in every context; `requires: [kubernetes]` runs only under
`--capability kubernetes`; `requires: [vm, bare_metal]` is any-match — either
context satisfies it. A plain suite with no `--capability` runs its core checks.

The same applies to your steps: if a step builds or destroys a fixture only some
contexts need, gate it so a core run neither provisions nor leaks it. Both halves
of a fixture take the same gate — see `config/storage.yaml`, where
`setup_cluster` and `teardown_cluster` are both `requires: [kubernetes]`.

Nothing is mandatory. A check is in scope only if you declared the suite holding
it, so 100% is always relative to what you declared.

## Private provider repositories

Keep your provider scripts in your own repository. The AI Cloud Validation
Suite supplies the CLI, validation suites, and validation code; your repository
supplies the provider `config/` and `scripts/` files. Run by config path during
development, or register a revision using the upstream
[provider registry guide](https://github.com/NVIDIA/ai-cloud-validation/blob/main/docs/guides/provider-registry.md).

Generate the scaffold directly into the private provider repository path you
intend to keep:

```bash
git clone <ai-cloud-validation>
cd ai-cloud-validation
uv sync

uv run isvctl provider scaffold acme --output-dir ../external-provider-acme
```

Then initialize or connect that generated directory to your private Git repo
and implement the scripts there:

```bash
cd ../external-provider-acme
git init
# Remove the scaffold overwrite marker if present before maintaining this repository.
rm -f .scaffold-meta
git remote add origin git@github.com:acme/external-provider-acme.git
# implement scripts/*
```

Run the private provider from the validation suite checkout by passing its
config path:

```bash
cd ../ai-cloud-validation
uv run isvctl test run --no-upload -f ../external-provider-acme/config/vm.yaml
```

Current limitation: out-of-tree provider YAML assumes you run `isvctl` from
the validation suite checkout root. Suite imports and shared scripts use paths
relative to that checkout; provider-owned scripts stay relative to the generated
provider `config/` directory.

## Anatomy of a script

The per-step Python stubs follow this general shape - this is what you're
copying:

```python
DEMO_MODE = os.environ.get("ISVCTL_DEMO_MODE") == "1"

def main() -> int:
    args = parser.parse_args()
    result = {"success": False, "platform": "<domain>", ...}

    # ╔═══════════════════════════════════════════════════════╗
    # ║  TODO: Replace with your platform's API calls         ║
    # ║  Example (pseudocode):                                ║
    # ║    client = MyCloudClient(region=args.region)         ║
    # ║    ...                                                ║
    # ╚═══════════════════════════════════════════════════════╝

    if DEMO_MODE:
        # dummy-success values for demo contract checks
        result["success"] = True
        result[...] = ...
    else:
        result["error"] = "Not implemented - replace with your platform's ... logic"

    print(json.dumps(result, indent=2))
    return 0 if result["success"] else 1
```

Keep the output field names in the documented contract - the validations
read specific keys (`instance_id`, `state`, `public_ip`, etc.). The AWS
reference implementation is the source of truth for what "correct" output
looks like.

For bare-metal serial console validation, the scaffold must also prove that
historical console logs are queryable for the required retention window. Emit
`console_log_queryable`, `retention_days_configured`,
`oldest_queryable_log_age_days`, `query_result_count`, and `retention_evidence`
from a real serial console log archive or retention policy query.

## See also

- [`config/`](../config/) - the YAML wiring that invokes these scripts
- [`suites/README.md`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/suites/README.md) - per-step breakdown and JSON field reference
- [AWS reference](https://github.com/NVIDIA/ai-cloud-validation/blob/main/docs/references/aws.md) - working implementation of every script in this tree
- [External Validation Guide](https://github.com/NVIDIA/ai-cloud-validation/blob/main/docs/guides/external-validation-guide.md) - writing scripts, JSON output format
