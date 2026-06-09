# Security and Safety Policy

RectaBot controls CNC machinery. A security or safety issue can mean property damage or injury, not just a software bug. Please report responsibly.

## What counts as a security/safety issue

### Hardware safety
- Designs that could damage connected equipment (over-voltage, mis-wired isolation, missing protection)
- Failure modes during MCU reset that could cause uncommanded motion or spindle activation
- Power tree issues that bypass the isolation barrier
- Mis-documented voltage levels that could lead a user to wire the board destructively

### Firmware vulnerabilities
- Buffer overflows in grblHAL handling that could cause undefined behavior during a job
- Issues that could cause unintended motion, spindle activation, or loss of E-Stop response
- Default credentials or hardcoded secrets

### Configurator/web tools
- XSS, injection, or other vulnerabilities in the web configurator
- Generated configuration files that could brick a board or produce unsafe motion settings

## How to report

**Email:** `security@rectabot.org`

Include:
- A clear description of the issue
- Steps to reproduce (if applicable)
- Affected hardware revision, firmware version, configurator version
- Your assessment of severity

Please **DO NOT** open a public GitHub issue for security or safety issues until we have agreed on disclosure.

## What to expect

- **Acknowledgment:** within 7 days (this is a solo-founder project; if you don't hear back, please re-send or contact `hello@rectabot.org`)
- **Initial assessment:** within 14 days of acknowledgment
- **Fix or mitigation:** depends on severity; we will keep you informed
- **Public disclosure:** coordinated with you; credit given unless you prefer otherwise

## Scope

This policy covers:
- The hardware design files in this repository
- The web configurator in `Docs/configurator/`
- The documentation in `Docs/`

Separate repositories have their own security contacts via the same `security@rectabot.org` address:
- [RectaBot-firmware](https://github.com/rectabot/RectaBot-firmware) — grblHAL fork
- [RectaPad](https://github.com/rectabot/RectaPad) — pendant firmware

## Out of scope

- Issues in upstream grblHAL — please report directly to https://github.com/grblHAL
- Issues in third-party stepper drivers, VFDs, or other equipment used with RectaBot
- General CNC machine safety issues unrelated to the RectaBot controller specifically

## Acknowledgments

Security researchers and users who report issues responsibly will be credited (with permission) in release notes and the project AUTHORS file.

---

For non-security contact, see `CONTRIBUTING.md` or email **hello@rectabot.org**.
