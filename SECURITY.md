# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| latest  | :white_check_mark: |

As Aura is in early development, only the latest version on `main` receives security updates.

## Reporting a Vulnerability

If you discover a security vulnerability in Aura, **please do not open a public issue.**

Instead, report it privately via **GitHub Security Advisories**: Use [GitHub's private vulnerability reporting](https://github.com/cjvana/aura/security/advisories/new).

### What to include

- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if you have one)

### What to expect

- Acknowledgment within 48 hours
- A plan for remediation within 7 days
- Credit in the release notes (unless you prefer anonymity)

## Scope

The following areas are in scope for security reports:

- **PTY handling**: Escape sequence injection, privilege escalation via terminal
- **Unix socket IPC**: Unauthorized access to the Aura socket, message spoofing
- **Claude Code hooks**: Hook injection, malicious payload in hook stdin/stdout
- **Configuration parsing**: TOML deserialization attacks, path traversal
- **GPU rendering**: Shader exploits, resource exhaustion via malicious input
- **Webview panel**: XSS in the wry-based side panel, IPC message injection
- **Dependencies**: Vulnerabilities in upstream crates

## Security Design

Aura follows these security principles:

- The Unix socket is created with user-only permissions (`0600`)
- Hook communication validates JSON schema before processing
- The webview panel uses a restricted custom protocol (no arbitrary URL loading)
- No network requests are made by Aura itself (Claude Code handles its own networking)
- Configuration file paths are resolved with care to prevent traversal
