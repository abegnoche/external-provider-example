<!--
SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0
-->

# Contributing

Contributions to `isvctl-provider-example` are welcome. Read the [README](README.md)
and follow our [Code of Conduct](CODE_OF_CONDUCT.md). Report suspected
vulnerabilities privately using the [security policy](SECURITY.md).

## What belongs here

This repository demonstrates externally maintained providers for the
[AI Cloud Validation Suite](https://github.com/NVIDIA/ai-cloud-validation).
It contains the `example` and `showcase` provider configurations and scripts,
plus CI that checks their compatibility with the latest suite release.

Improvements to these examples, their documentation, and compatibility checks
belong here. Changes to the CLI, validation contracts, or provider registry belong
in `ai-cloud-validation`. Keep real provider implementations in their own
repositories; these examples must remain usable without cloud credentials for
the demo suites.

## Issues and questions

Use this repository's **Issues** tab for bugs, documentation requests, and
suggestions. Search existing issues first. Include the provider name, this
repository's commit, the AI Cloud Validation Suite version or commit, the command
you ran, and the expected and actual results. Remove credentials and sensitive
infrastructure details from logs before sharing them.

For general suite questions, use the
[upstream discussions](https://github.com/NVIDIA/ai-cloud-validation/discussions).

## Development and validation

Fork this repository and create a branch from `main`. Follow the
[README setup instructions](README.md#reproducing-the-results) to place it beside
an `ai-cloud-validation` checkout and install that checkout's dependencies with
`uv sync`. Run commands from the **validation checkout root**; this repository
does not have its own Python package, Makefile, or pre-commit configuration.

For example, check changes to the VM wiring with:

```bash
ISVCTL_DEMO_MODE=1 uv run isvctl test run --no-upload \
  -f ../isvctl-provider-example/example/config/vm.yaml
ISVCTL_DEMO_MODE=1 uv run isvctl test run --no-upload \
  -f ../isvctl-provider-example/showcase/config/vm.yaml
```

Use the local config paths above when testing edits; fetching a registry entry
tests its pinned revision instead of your working tree. For other suites, use
the capability flags listed in the [README](README.md#reproducing-the-results).

The [validation workflow](.github/workflows/validate.yml) defines the full demo
matrix and runs on pull requests, pushes to `main`, and a weekly schedule. It
tests both providers against the newest `vX.Y.Z` tag. Kubernetes and Slurm need
real clusters; storage needs a manifest that these examples do not ship. They
are excluded from the demo matrix. Do not use real credentials for demo checks.

When editing scripts or configurations:

- Preserve the suite's JSON field names, step names, and capability gates.
- Keep demo data clearly synthetic and preserve the default failure for
  unimplemented provider operations.
- Check both providers when changing a shared pattern, including the default
  failure and demo success paths of affected stubs.
- Keep NVIDIA copyright and Apache-2.0 SPDX headers on source files.
- Update the relevant documentation when behavior or usage changes.

A passing demo checks wiring and output contracts; it does not validate a real
platform or demonstrate that a security control works.

## Pull requests

1. Keep the change focused and explain the problem and resulting behavior.
2. Run the affected checks and include commands, versions, and results. Say what
   you could not test; documentation-only changes can state that no runtime tests
   were needed.
3. Sign off every commit as described below.
4. Open a pull request against this repository's `main` branch and respond to
   review feedback. Applicable CI checks must pass before merging.

Maintainers review contributions on a best-effort basis.

### AI-assisted contributions

AI-assisted contributions are welcome when a person chooses the problem, reviews
the changes, and can explain them. State which tool you used and how much of the
change it produced. Autonomous agents should not open issues or pull requests.
Report what was actually tested, including whether any real cluster was used.

## Developer Certificate of Origin (DCO)

All contributions require sign-off under the
[Developer Certificate of Origin 1.1](https://developercertificate.org/).
Signing off certifies that you have the right to submit the contribution under
the project's license and that you understand the contribution and sign-off
become part of the public record.

Use `git commit -s` to add a `Signed-off-by: Your Name <your@email.com>` line to
each commit. Sign-off is separate from cryptographic commit signing. Maintainers
check sign-offs during review.

## License

By contributing, you agree that your contributions are licensed under the
[Apache License 2.0](LICENSE).
