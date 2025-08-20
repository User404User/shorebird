# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Shorebird is a code push platform for Flutter applications that enables over-the-air (OTA) updates without going through app stores. This is a Dart monorepo using workspaces, containing multiple packages that together form the Shorebird ecosystem.

## Development Commands

### Initial Setup
```bash
# Bootstrap the repository (requires Dart in PATH)
./scripts/bootstrap.sh
```

### Running Tests
```bash
# Run all tests from the packages directory (recommended)
cd packages && very_good test -r

# Run tests for a specific package
cd packages/<package_name> && dart test

# Run tests with coverage
dart test --coverage=coverage
dart pub global run coverage:format_coverage --lcov --in=coverage --out=coverage/lcov.info --packages=.dart_tool/package_config.json --report-on=lib --check-ignore
```

### Code Quality
```bash
# Format code
dart format .

# Analyze code
dart analyze --fatal-warnings lib test

# Check formatting without applying changes
dart format --set-exit-if-changed .
```

### Common Development Tasks
```bash
# Upgrade all dependencies
./scripts/upgrade_all.sh

# Generate combined coverage report
./scripts/generate_combined_coverage.sh

# Run E2E tests
./scripts/patch_e2e.sh <flutter-version>
./scripts/apple_e2e.sh
```

## Architecture Overview

### Repository Structure
The codebase is organized as a Dart workspace monorepo with these key packages:

- **shorebird_cli**: Main CLI tool that developers use to release and patch Flutter apps
- **shorebird_code_push_client**: Client library for communicating with Shorebird's backend API
- **shorebird_code_push_protocol**: Shared protocol definitions and data models
- **artifact_proxy**: Proxy server for intercepting and serving Flutter artifacts
- **scoped_deps**: Dependency injection framework used throughout the codebase
- **jwt**: JWT token verification for authentication
- **redis_client**: Redis integration for caching
- **stripe_api**: Stripe payment integration

### Key Architectural Patterns

1. **Dependency Injection**: Uses `scoped_deps` for Zone-based DI. All major components are injected as refs (e.g., `shorebirdEnvRef`, `loggerRef`)

2. **Command Structure**: CLI uses hierarchical command pattern with base classes in `src/commands/`

3. **Platform Abstraction**: Each platform (Android, iOS, etc.) has specific implementations for:
   - Releasers (`src/releasers/`)
   - Patchers (`src/patchers/`) 
   - Artifact builders (`src/artifact_builder.dart`)

4. **Flutter Integration**: Maintains a fork of Flutter with modifications for code push. The CLI manages downloading and using specific Flutter versions.

### Code Push Flow

1. **Release**: `shorebird release [platform]` builds the app with Shorebird's Flutter fork and uploads artifacts
2. **Patch**: `shorebird patch [platform]` rebuilds only Dart code and uploads the diff
3. **Runtime**: Apps check for patches on launch and apply them for the next run

### Important Conventions

- All packages use `very_good_analysis` for linting
- Formatter page width is 80 characters
- Tests use `mocktail` for mocking
- Error messages should be user-friendly and actionable
- Platform-specific code is isolated in separate classes/files
- Use dependency injection rather than global state

### Working with the CLI

The main entry point is `packages/shorebird_cli/bin/shorebird.dart`. Commands are in `packages/shorebird_cli/lib/src/commands/`. When adding new commands:

1. Create a new command class extending `ShorebirdCommand`
2. Add it to the command runner in `shorebird_command_runner.dart`
3. Use dependency injection for all external dependencies
4. Add comprehensive tests

### API Integration

The backend API is at `api.shorebird.dev`. All API interactions go through `ShorebirdCodePushClient`. The protocol is defined in the `shorebird_code_push_protocol` package.

### Testing Guidelines

- Unit test all business logic
- Mock external dependencies using `mocktail`
- Integration tests go in `test/integration/`
- E2E tests are in `scripts/` and run in CI
- Aim for high code coverage (tracked via Codecov)