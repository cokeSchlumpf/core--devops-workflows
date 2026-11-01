# Changelog

## [1.1.1](https://github.com/cokeSchlumpf/core--devops-workflows/compare/v1.1.0...v1.1.1) (2026-11-01)


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

## Changelog

All notable changes to these workflows are documented here. The newest release is at the top.
