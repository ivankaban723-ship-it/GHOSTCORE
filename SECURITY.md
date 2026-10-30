# Security Policy

## Security Model

- **Process Secrets Protection:** Command line arguments (`/proc/PID/cmdline`) are treated as local-only data and never transmitted across network/IPC channels or saved in persistent DB logs.
- **IPC Access Control:** Unix socket created by GHOSTCORE uses strict file system permissions `0600`.

## Reporting a Vulnerability

If you discover a security issue, please do not open a public issue. Contact the repository maintainer directly via GitHub (@Mr-Ivan).
