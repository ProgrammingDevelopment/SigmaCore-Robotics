# Security Policy

## Supported Versions

The following versions of SigmaPrompt Robotic OS are currently supported with security updates:

| Version | Supported |
|---------|----------|
| 1.x     | ✅        |
| < 1.0   | ❌        |

---

## Reporting a Vulnerability

If you discover a security vulnerability, please report it responsibly.

Do NOT open public issues for security-related problems.

Instead, please:

- Contact the maintainer directly
- Provide a detailed technical description
- Include reproduction steps
- Include impact assessment if possible

We aim to respond within 72 hours.

---

## Security Architecture Overview

SigmaPrompt Robotic OS implements:

- TLS-encrypted communication
- Row-Level Security (RLS) in PostgreSQL
- Role-Based Access Control (RBAC)
- Encrypted environment variables
- Secure device token authentication
- Fail-safe actuator control logic

---

## Responsible Disclosure

Security reports will be handled confidentially. Coordinated disclosure is encouraged to protect users and infrastructure.
