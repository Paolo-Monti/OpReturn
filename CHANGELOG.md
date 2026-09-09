# Changelog

All notable changes to OP_RETURN Mailbox Reader are documented in this file.

## 1.1 - 2026-09-09

### Added

- API-key authentication for custom HTTPS explorer endpoints.
- Secure API-key loading from an environment variable.
- Configurable authentication header names and value prefixes.
- HTTP and HTTPS proxy support.
- Authenticated proxy support with passwords loaded from environment variables.
- Block-height filtering for single heights, comma-separated lists, inclusive
  ranges, and combined specifications.

### Security

- API authentication is restricted to explicitly selected custom APIs, which
  prevents a key from being sent to the default public providers.
- API keys and proxy passwords are excluded from normal and verbose output.
- The distributed executable is compressed with PEPack LZMA and includes
  integrity and import address table checks.

## 1.0 - 2026-09-08

- Initial public release.
