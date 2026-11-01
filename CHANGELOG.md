# Changelog

All notable changes to these workflows are documented here. The newest release is at the top.

## [1.4.0](https://github.com/cokeSchlumpf/core--devops-workflows/compare/v1.3.0...v1.4.0) (2026-11-01)

### Highlights

A new reusable workflow checks TLS certificates for app custom domains and fails if a certificate is invalid or will expire within the configured threshold (default 30 days). Apps with custom domains can run it weekly so failed renewals produce GitHub notifications.

### Features

* Add workflow app-certificate-check that reads host names from the output `custom_domains` of `deployment/<stage>` (module `sandbox_app`) and fails when a certificate is invalid (expired, wrong host name, untrusted) or expires in less than `min-days` (30) days; intended for weekly runs by apps with custom domains. ([#16](https://github.com/cokeSchlumpf/core--devops-workflows/issues/16)) ([d98f81d](https://github.com/cokeSchlumpf/core--devops-workflows/commit/d98f81daaab100d83c2751b6c279dc29d64a5504))

## [1.3.0](https://github.com/cokeSchlumpf/core--devops-workflows/compare/v1.2.0...v1.3.0) (2026-11-01)


### Features

* destroy app deployments ([#15](https://github.com/cokeSchlumpf/core--devops-workflows/issues/15)) ([35d9140](https://github.com/cokeSchlumpf/core--devops-workflows/commit/35d91404f40b863b48a7743754fd74e8f8dd09b0))


### Documentation

* Move the changelog intro to the top ([#13](https://github.com/cokeSchlumpf/core--devops-workflows/issues/13)) ([78db931](https://github.com/cokeSchlumpf/core--devops-workflows/commit/78db93139f9d217283c26e234a7e105eb6d53a21))

## [1.2.0](https://github.com/cokeSchlumpf/core--devops-workflows/compare/v1.1.1...v1.2.0) (2026-11-01)

### Highlights

The default Python version for the reusable workflows has been raised to 3.14. Callers that explicitly set python-version are unaffected; projects without that input will run on 3.14 from this release. This aligns with downstream projects (commons and pyhx) that require Python 3.14.

### Features

* Default the reusable workflows to Python 3.14; `python-check.yml` and `python-docs.yml` now default `python-version` to "3.14" and the README input table has been updated. Callers that set `python-version` remain unchanged. ([#11](https://github.com/cokeSchlumpf/core--devops-workflows/issues/11)) ([53842a0](https://github.com/cokeSchlumpf/core--devops-workflows/commit/53842a0521eb39046457975c50d2916324f69ac7))

## [1.1.1](https://github.com/cokeSchlumpf/core--devops-workflows/compare/v1.1.0...v1.1.1) (2026-11-01)

### Highlights
This release fixes a problem where polished release notes could include repeated pull request descriptions and extra top-level headings, which introduced invalid links and broke strict docs builds. validate() now rejects polished notes that contain a `#` or `##` heading after the version heading, leaving invalid notes unpolished.

### Bug Fixes

* Reject polished notes that repeat the pull request descriptions ([#9](https://github.com/cokeSchlumpf/core--devops-workflows/issues/9)) ([0b57390](https://github.com/cokeSchlumpf/core--devops-workflows/commit/0b573907f0a74c6910003b9454c0e950b0003fe5))

## [1.1.0](https://github.com/cokeSchlumpf/core--devops-workflows/compare/v1.0.2...v1.1.0) (2026-11-01)


### Features

* deploy apps on release ([#7](https://github.com/cokeSchlumpf/core--devops-workflows/issues/7)) ([fce92a4](https://github.com/cokeSchlumpf/core--devops-workflows/commit/fce92a448605d66af567d4667f309fa53e0b95bb))

## [1.0.2](https://github.com/cokeSchlumpf/core--devops-workflows/compare/v1.0.1...v1.0.2) (2026-10-31)


### Bug Fixes

* Use Poetry 2.5.1 by default, which supports Python 3.14 ([#5](https://github.com/cokeSchlumpf/core--devops-workflows/issues/5)) ([14a4278](https://github.com/cokeSchlumpf/core--devops-workflows/commit/14a4278b3497d296711ab64d4b2e54d5d20aafa1))

## [1.0.1](https://github.com/cokeSchlumpf/core--devops-workflows/compare/v1.0.0...v1.0.1) (2026-10-31)

### Highlights

Workflows now ensure Poetry uses the Python interpreter provided by actions/setup-python before creating the project environment, so projects that require a newer Python version no longer fail during poetry install. This does not change behavior for callers on the default 3.12.

### Bug Fixes

* Run Poetry with the interpreter provided by actions/setup-python before creating the virtual environment so Poetry uses the requested Python version and projects requiring a newer Python no longer fail during `poetry install`. ([#4](https://github.com/cokeSchlumpf/core--devops-workflows/issues/4)) ([9bd2ba8](https://github.com/cokeSchlumpf/core--devops-workflows/commit/9bd2ba8863dc422c5a60a6f6e70fc97d6f1c72c4))

### Documentation

* Added additional repository setup instructions. ([64ec9d5](https://github.com/cokeSchlumpf/core--devops-workflows/commit/64ec9d5f450109afcbb90b30b959974395a4adb2))

## 1.0.0 (2026-09-27)

### Highlights

This release fixes a runtime resolution issue with the "polish" action so the project's first release can run successfully. It is a focused bugfix to ensure the release workflow proceeds.

### Bug Fixes

* Resolve polish action at runtime so the first release can run ([#1](https://github.com/cokeSchlumpf/core--devops-workflows/issues/1)) ([c824a40](https://github.com/cokeSchlumpf/core--devops-workflows/commit/c824a402b346f6ac9e0945d68ea915effd2d1b11))
