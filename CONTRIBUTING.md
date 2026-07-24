# Contributing to RectaBot

Thanks for your interest in contributing! 🎉

RectaBot is an open-source CNC controller project. This document explains how
to contribute effectively.

---

## 📋 Before You Contribute

### For bug reports
- Use the **Bug Report** issue template
- Include hardware revision (v1.0, v1.1, etc.)
- Include firmware version (`$I` command output)
- Steps to reproduce + expected vs actual behavior

### For feature requests
- Use the **Feature Request** issue template
- Explain the use case (what CNC machine? what problem solved?)
- Check existing issues — your idea might already be tracked

### For hardware design discussions
- Open a **Discussion** (not an issue) in the Hardware category
- Reference specific schematic / layout files

---

## 🛠️ Repository Structure

This is the **main** RectaBot repository. The project spans these repositories:

| Repo | Purpose |
|---|---|
| **RectaBot** (this one) | Hardware design (Gerber, BOM), documentation, brand assets, configurator tool |
| **RectaBot-firmware** | grblHAL fork with RectaBot board map |

---

## 🔧 Development Workflow

1. **Fork** the repository
2. **Clone** your fork locally
3. **Create a branch** for your change: `git checkout -b feature/short-description`
4. **Make changes** following the style of surrounding code
5. **Test** locally before submitting
6. **Commit** with clear messages (see Commit Style below)
7. **Push** to your fork and open a Pull Request

---

## 💬 Commit Style

We use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(configurator): add VFD Modbus model dropdown

- Adds Huanyang HY01/HY02 model selection
- Wires $395 spindle type based on selection
- Updates working area display reactively

Closes #42
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

---

## 🧪 Testing

### Web Configurator
- Open `Docs/configurator/index.html` in a browser
- Click through wizard steps
- Verify generated `config.txt` matches expected format

### Documentation
- Render Markdown locally (`Ctrl+Shift+V` in VS Code)
- Verify all internal links work
- Spellcheck (optional, but appreciated)

---

## 📐 PCB / Hardware Changes

Hardware changes go through these phases:

1. **Design discussion** in Discussions
2. **PR with schematic change**
3. **Review by maintainer**
4. **Prototype order** (if accepted)
5. **Prototype validation**
6. **Merge to main + version bump**

Major hardware changes warrant a new revision number (v1.1, v2.0).

---

## 🛡️ Security

If you discover a security vulnerability, please **DO NOT** open a public
issue. Email: `security@rectabot.org` (forwarded to maintainer).

Security topics:
- Firmware vulnerabilities (e.g., buffer overflows in grblHAL handling)
- Web Configurator XSS / injection issues
- Default credentials / hardcoded secrets

---

## 📜 License

By contributing, you agree your contributions are licensed under:
- **MIT** for code and documentation (this repository) — see `LICENSE`
- **CERN-OHL-S v2** for hardware design files (Gerber, BOM, CPL) — see `LICENSE.hardware`
- **CERN-OHL-S v2** for brand assets (logo SVGs) — see `Docs/Brand_Assets/README.md`
- **GPL-3.0** for grblHAL firmware contributions — separate repo: [RectaBot-firmware](https://github.com/rectabot/RectaBot-firmware)

---

## 💬 Community

- **Discussions:** GitHub Discussions (this repo)
- **Email:** hello@rectabot.org
- **Forum (planned):** community.rectabot.org

---

*Thanks for making RectaBot better! Every issue, idea, and PR moves us forward.* 🚀
