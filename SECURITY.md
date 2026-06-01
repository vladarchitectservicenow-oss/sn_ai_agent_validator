# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please do **not** open a public issue. Instead, contact the project maintainer directly:

- Email: vladarchitect@github
- GitHub: [vladarchitectservicenow-oss](https://github.com/vladarchitectservicenow-oss)

## Security Design Principles

This application operates entirely within the ServiceNow platform boundary. No scan data, configuration details, or readiness scores ever leave the tenant. The scanner reads from ServiceNow tables via GlideRecord, writes to a scoped custom table, and exposes results through authenticated REST endpoints.

### What We Do NOT Store or Transmit

- Instance credentials or API keys
- BYOK provider authentication secrets
- User session tokens
- Any data external to the ServiceNow instance

### Authentication

All REST endpoints require Basic Auth with a valid ServiceNow session. Unauthenticated requests return HTTP 401.

## Supported Versions

| Version | Supported |
|---------|-----------|
| Australia (2025) | ✅ Full support |
| Zurich (2024) | ✅ Full support |
| Utah (2023) | ✅ Full support |
| Vancouver (2023) | ✅ Limited (AI Agent features require Utah+) |

## Security Updates

Security fixes are released as patch versions. Subscribe to repository releases for notifications.
