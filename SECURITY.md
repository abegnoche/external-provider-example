<!--
SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0
-->

# Security

`external-provider-example` contains example provider scripts and configurations
for the NVIDIA AI Cloud Validation Suite. It is not an NVIDIA-supported product.
Demo results are synthetic and provide no assurance about a real platform's
security or compliance.

## Reporting a vulnerability

**Do not disclose vulnerabilities in public issues, pull requests, or discussions.**
This project follows the NVIDIA reporting channels used by
[AI Cloud Validation Suite](https://github.com/NVIDIA/ai-cloud-validation/blob/main/SECURITY.md):

- **Web (preferred):** [NVIDIA Product Security](https://www.nvidia.com/en-us/security/).
- **Email:** <psirt@nvidia.com>. Use the
  [NVIDIA public PGP key](https://www.nvidia.com/en-us/security/pgp-key) for
  sensitive information.
- **GitHub private reporting:** If this repository's **Security** tab offers
  **Report a vulnerability**, you may use it. If the option is unavailable,
  use the NVIDIA web or email channel above.

Include the repository name and affected commit, the AI Cloud Validation Suite
version or commit, the affected provider and files, the vulnerability type,
reproduction steps, a proof of concept if available, and the potential impact.
Do not include live credentials or unrelated private data.

See NVIDIA's [PSIRT policies](https://www.nvidia.com/en-us/security/psirt-policies/)
for its coordinated disclosure and response process.

## Security architecture and context

This repository supplies YAML configurations, Python provider stubs, a storage
shim scaffold, and Bash cluster inventory scripts. The separately installed
AI Cloud Validation Suite loads the configurations and executes their commands.
The repository does not implement a hosted API, authentication service, or
credential store.

**Repository exposure:** Public; GitHub reports this repository as public.
**Software exposure:** Externally distributed example scripts. These are local
tools, with no hosted service or production support commitment.

Most Python stubs emit dummy JSON success only with `ISVCTL_DEMO_MODE=1` and fail
otherwise. The storage shim also has unimplemented operations. Kubernetes and
Slurm inventory scripts query existing clusters through local tools such as
`kubectl`, `sinfo`, and `scontrol`; demo mode does not isolate those tools.

The [CI workflow](.github/workflows/validate.yml) installs the latest tagged
AI Cloud Validation Suite and exercises a limited demo matrix with `--no-upload`.
It excludes Kubernetes, Slurm, and storage. The configs and scripts execute with
the permissions and environment of the invoking user or CI runner.

## Threat model

- **Untrusted executable content:** Changes to provider YAML commands, Python
  scripts, or imported validation code can execute with the caller's privileges.
  Review the repository revision and upstream dependencies before running them.
- **Accidental cluster access:** `scripts/k8s/` and `scripts/slurm/` under each
  provider use the caller's cluster context and local commands. A run can query
  an unintended cluster and expose its inventory; confirm the target environment
  before using these configurations.
- **Misleading evidence:** `scripts/common/stub.py` and the storage shim can emit
  fabricated success in demo mode. Treating those results as real validation
  would conceal missing platform functionality and security controls.
- **Disclosure through results:** Cluster inventory, logs, and command output
  can reveal infrastructure details. Review output before sharing it and use
  `--no-upload` for local example runs to prevent result submission by the suite.
- **CI dependency changes:** The workflow resolves the newest upstream release
  and executes its dependencies and these provider scripts. Keep the runner
  disposable, its token permissions minimal, and real cloud credentials out of
  demo jobs.

## Critical security assumptions

- The operator trusts the selected provider code, validation checkout, Python
  environment, and executables on `PATH`. Provider execution is not sandboxed by
  this repository.
- Cluster access controls and credential protection are supplied by the host,
  cluster, and invoking tools. These examples do not add an authorization layer.
- Demo runs use synthetic data without real cloud credentials and do not upload
  results. `ISVCTL_DEMO_MODE=1` changes stub behavior; it is not a security boundary.
- Anyone adapting a stub into a real provider must implement authentication,
  input validation, secret handling, resource cleanup, and real verification for
  that platform. Dummy security-check output does not implement those controls.
