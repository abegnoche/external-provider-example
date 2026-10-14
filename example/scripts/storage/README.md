<!--
SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0
-->

# example storage shim

Storage Provider Shim - one Python file per backend storage
provider implementing the [`StorageApi`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvtest/src/isvtest/core/storage_provider/api.py)
ABC. Drives the storage acceptance subsets that
the [`StorageProviderApiCheck`](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvtest/src/isvtest/validations/storage_provider.py)
validation runs.

## The two pieces

```text
config/storage-provider-manifest.yaml   <- declares the provider(s)
                                            (see the JSON schema)
                  │
                  ▼ providers[].shim.module points at
scripts/storage/api.py                  <- StorageApi subclass +
                                            build_api() factory
```

The manifest is the discovery surface. `StorageProviderApiCheck` reads
the manifest from `manifest_path`, iterates each `providers[]` entry
with a `shim:` block, calls `build_api()`, and runs N-019/N-020/N-021
against the returned `StorageApi`. CSI-only providers (no `shim:` block)
are skipped at the shim layer and exercised by the existing `K8sCsi*`
checks instead.

## Manifest structure (schema v1alpha2)

The manifest describes the storage provider's shim contract — it drives
`StorageProviderApiCheck` only. At the package level it carries `namespace`
(the DNS domain you control) and optional `vendor` metadata; with each
provider's `id` the namespace forms the registration key `<namespace>/<id>`.
Per provider it carries:

- `provider` — implementor identity block (`name`, `description`, `type`,
  `protocols`, `version`) mirroring `ProviderProperties` / `VersionMetadata`.
  `type` + `protocols` + `version` are cross-checked against the shim's
  `properties()`. Top-level `name`/`type` are derived from it when omitted.
- `backend` — optional metadata for the storage system fronted (its `version`
  is an opaque vendor passthrough).
- `shim` — `kind` + `module` (plus production `configmap`/`credentials_secret`).
- `capabilities` — a hierarchical `native | default | none` block
  (`tenantManagement`, `volumeManagement`, `quotaManagement`).

StorageClass names, NFS mount-option expectations, node selectors, and kernel
modules are NOT in the manifest: the `K8sCsi*` / `K8sFilesystem*` checks read
them from the suite/provider config or the `K8S_CSI_*` env vars.

### The manifest is a contract

Declared identity (the `provider` block) and `capabilities` are not
documentation: for providers
with a `shim:` block, `StorageProviderApiCheck` cross-checks each declared
value and **fails on a mismatch** (a `manifest-consistency[<name>]` subtest,
plus `volumeManagement.create` enforced in `volume-provisioning` and
`tenant_id` in `tenant-quota`). Identity fields are checked against the shim's
`properties()`; each capability declared **supported** is **probed at
runtime** — the shim must not raise `NotSupportedError` / `NotImplementedError`
for it. So you can't declare `provider.protocols: [lustre]` or
`quotaManagement.directory: native` unless the shim actually backs it. Omit a
field to leave it unchecked; declare a surface `none` to leave it unprobed.

This repository does **not** ship a storage provider manifest, so its storage
config cannot run as-is and is excluded from demo CI. To implement storage,
start from the upstream templates, place your manifest in this provider's
`config/` directory, and adapt its identity and shim path:

- [Blank manifest template](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/providers/my-isv/config/storage-provider-manifest.yaml)
- [Populated manifest example](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvctl/configs/providers/my-isv/config/storage-provider-manifest.example.yaml)

The schema is `isvctl/schemas/storage-provider-manifest.schema.json`.
`v1alpha1` manifests still load.

## Driving the tests

The manifest drives `StorageProviderApiCheck` ONLY. `storage_manifest_to_steps.py`
in the validation checkout (`isvctl/configs/providers/shared/`) resolves the
manifest path in a `storage_manifest` step and emits
`steps.storage_manifest.storage.manifest_path`, which the check loads in-process.

The CSI / NFS / POSIX filesystem checks are config-driven like every other suite
check: set their StorageClass names via the `K8S_CSI_*` env vars (or literal
overrides in your config). Resolution is **explicit YAML → `K8S_CSI_*` env var →
skip**. [`../../config/storage.yaml`](../../config/storage.yaml) wires both:

After adding and configuring the manifest, run from an AI Cloud Validation
Suite checkout beside this repository:

- Core run (shim check, no cluster):

  ```bash
  ISVCTL_DEMO_MODE=1 \
    uv run isvctl test run --no-upload -f ../isvctl-provider-example/example/config/storage.yaml
  ```

- Kubernetes run (StorageProviderApiCheck from the manifest; CSI/filesystem
  checks against an existing cluster, StorageClasses from `K8S_CSI_*` env vars):

  ```bash
  export K8S_CSI_SHARED_FS_SC=example-rwx
  uv run isvctl test run \
      --no-upload -f ../isvctl-provider-example/example/config/storage.yaml \
      --capability kubernetes \
      -- -k "K8sCsi or K8sFile or K8sNfs or StorageProviderApi"
  ```

## Authoring checklist

1. **Backend connection.** Edit `MyStorageApi.__init__` in
   [`api.py`](api.py) to read your endpoint URL / credentials from
   environment variables (or `/etc/shim/config.yaml` once the per-provider
   ConfigMap topology lands).
2. **`health_check`** - authenticated round-trip to the management API.
   Raise `AuthenticationError` on 401/403; return `None` on success.
   Drives N-019.
3. **`get_tenant_quota`** - return `TenantQuota(tenant_id, hard_limit_bytes,
   used_bytes, name)`. Drives N-021. The shim's default tenant comes from the
   `STORAGE_TENANT_ID` env var (the knob); the manifest's `tenant_id` is a
   cross-check assertion that must match what `get_tenant_quota` returns.
4. **`list_volumes`** - enumerate volumes in the selected tenant, honoring
   the optional `ids` / `tag_filters` arguments. Never leak foreign-tenant
   volumes regardless of filters.
5. **`create_volume` / `delete_volume`** (optional, off by default) - the
   scaffold ships these raising `NotSupportedError` with
   `volumeManagement.create` / `.delete` declared `none`: the managed-K8s
   default where the CSI driver owns provisioning, so the suite falls back to
   inventorying existing volumes via `list_volumes` (N-020). Implement both and
   flip those two entries to `native` only if your shim provisions volumes
   itself.
6. **Directory / user quota methods** (optional) - the scaffold ships
   these as stubs that raise `NotSupportedError`. Implement the ones your
   backend backs and declare the matching surface `native` in the
   manifest's `quotaManagement` block; leave the rest as raising stubs and
   declare them `none`. Validation probes each surface declared supported
   and fails if it raises the sentinel. Attach L2 qualifiers via the
   `capability_qualifiers()` hook.

Every method has a `TODO` block marking the spot to replace.

## Demo mode

The skeleton ships with a `DEMO_MODE = os.environ.get("ISVCTL_DEMO_MODE")
== "1"` gate on the implemented surfaces (`health_check`, `get_tenant_quota`,
`list_volumes`): real runs raise `NotImplementedError` /
`AuthenticationError` to make it obvious the backend isn't wired up; demo runs
return dummy data so the validation passes end-to-end. The optional
`create_volume` / `delete_volume` stay raising `NotSupportedError` either way
(volume-provisioning then reports a clean skip). After adding and configuring
the missing manifest, exercise the shim in demo mode:

```bash
ISVCTL_DEMO_MODE=1 \
  uv run isvctl test run --no-upload -f ../isvctl-provider-example/example/config/storage.yaml
```

## Delivering a real provider

Keep the implemented shim and its configured manifest together in your provider
repository. Supply credentials through your deployment's secret management;
do not commit credentials or include them in example output. See the upstream
[Storage Provider API guide](https://github.com/NVIDIA/ai-cloud-validation/blob/main/isvtest/src/isvtest/core/storage_provider/README.md)
for the contract and integration details.
