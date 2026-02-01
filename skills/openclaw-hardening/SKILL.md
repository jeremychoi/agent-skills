---
name: openclaw-hardening 
description: A comprehensive security auditor for OpenClaw/Moltbot/Clawdbot instances. It checks for common misconfigurations, insecure network bindings, and exposed secrets. Use this skill when a user asks to "secure my agent," "audit openclaw," "check for vulnerabilities," or "harden my configuration."
---

# OpenClaw Hardening & Security Auditor

This skill provides a structured workflow to analyze the security posture of an OpenClaw deployment. It focuses on identifying the "Lethal Trifecta" of risks: Privileged Access, Network Exposure, and Untrusted Input.

## Instructions

### Phase 1: Reconnaissance & Configuration Analysis

1. Locate Configuration:
    - Search for the primary configuration file. Check standard locations in order: `~/.openclaw/openclaw.json`, `~/.moltbot/config.json`, or the directory specified in the `OPENCLAW_CONFIG` environment variable.
    - Load the file and identify the `gateway` configuration block.
2. Check Network Bindings (CRITICAL):
    - Inspect `gateway.bind`.
        - PASS: If set to `"loopback"` or `127.0.0.1`.
        - FAIL: If set to `"0.0.0.0"`, `"lan"`, or a public IP address.
    - Inspect `gateway.port`. Note the port (default `18789`).
    - Proxy Vulnerability Check: Check if the environment suggests a reverse proxy usage (e.g., presence of Nginx/Caddy config files). If a proxy is likely, verify if `gateway.trustedProxies` is configured in `openclaw.json`.
        - RISK: If `trustedProxies` is missing or empty, flag as HIGH RISK (Localhost Trust Bypass Vulnerability).
3. Authentication Verification:
    - Check for `gateway.auth.token` or `gateway.auth.password`.
    - FAIL: If these are null, empty, or set to default strings (e.g., "changeme", "password").
    - CRITICAL: Check for dangerous flags: `gateway.controlUi.allowInsecureAuth` or `dangerouslyDisableDeviceAuth`. If either is `true`, flag as CRITICAL.
4. Sandbox Status:
    - Inspect `agents.defaults.sandbox`.
    - PASS: If `mode` is `"non-main"` or `"all"`.
    - WARN: If `mode` is `"off"`.
    - FAIL: If the process is running as `root` user without strict sandboxing.

### Phase 2: Skill Auditing

1. Review Installed Skills:
    - List contents of `~/.openclaw/skills/` or `~/.openclaw/extensions/`.
    - Malware Scan: Search skill directories for suspicious keywords in `manifest.json` or code files. Look for: `curl`, `wget`, `eval`, `exec`, `process.env`, `child_process`.
    - Provenance Check: Flag any skills with "unknown" or "community" provenance that haven't been pinned or verified.
    - Specific Threat: Check for a skill named "What Would Elon Do?" or similar variants known to be malicious.

### Phase 3: Filesystem & Secret Hygiene

1. Permission Check:
    - Verify permissions of the `.openclaw` directory. It should be `700` (user read/write/execute only).
    - Verify permissions of `openclaw.json`. It should be `600` (user read/write only).
2. Secret Exposure:
    - Scan the configuration file for plaintext secrets (API keys starting with `sk-`, OAuth tokens).
    - Recommendation: If found, advise the user to move these to environment variables or use the Auth0 Token Vault integration.

### Phase 4: Remediation & Reporting

1. Generate Report:
    - Summarize findings into a Markdown table with columns: `Component`, `Status` (PASS/WARN/FAIL), `Risk Level`, `Remediation`.
2. Actionable Fixes:
    - For Network Fails: "Set `gateway.bind` to `127.0.0.1` immediately to prevent public exposure."
    - For Auth Fails: "Generate a strong, random token and set `gateway.auth.token`."
    - For Proxy Issues: "Configure `gateway.trustedProxies` with your Nginx IP or set `LOCALHOST_TRUST_ENABLED=false`."

## Examples

User: "Is my OpenClaw secure?" Claude: "I will run the hardening audit. First, I'm checking your network config...

- Analysis: I found `gateway.bind` is set to `0.0.0.0`.
- Result: FAIL (Critical Risk). Your agent is listening on all network interfaces.
- Remediation: Change this to `127.0.0.1` in `openclaw.json`."

User: "Check my skills for malware." Claude: "Scanning `~/.openclaw/skills`...

- Analysis: I detected a skill named 'weather-pro' that contains an obfuscated `curl` command to an external IP.
- Result: FAIL (High Risk). This resembles a known malware pattern.
- Remediation: Delete the `weather-pro` directory immediately and rotate your API keys."

## Execution Context

- When executing shell commands to check permissions (e.g., `ls -l`), use the `bash` tool.
- When reading config files, use the `fs.readFile` tool.
- Safety First: Do not display full API keys or tokens in the output. Mask them (e.g., `sk-ant...`).