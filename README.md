# ⚔️ Attacker vs Defender

> A deep-dive hub covering the eternal arms race between Windows kernel
> defenders and offensive researchers.

**🌐 Live site → [attvdef.mintlify.site](https://attvdef.mintlify.site/)**

---

## What is this?

A collection of long-form notes on **Windows kernel internals**, written
from the perspective of the security arms race: how each kernel
abstraction was built, how it was attacked, and how Microsoft responded.

If you've ever opened WinDbg, stared at `dt nt!_EPROCESS`, and wondered
*why the hell is this field here* — this site is for you.

## Why visit the site?

This README is just the door. The actual content — annotated structs,
WinDbg cheat sheets, Python snippets, and Mermaid diagrams — lives at
**[attvdef.mintlify.site](https://attvdef.mintlify.site/)**.

## What's covered

- 🛡️ **Handle Table Internals** — the foundation. Every user-mode
  reference to a kernel object goes through here.
- 🧬 **_EPROCESS Anatomy** *(soon)* — the structure attackers love to
  patch: token, ACLs, links, PPL flags.
- 🔐 **The Object Manager** *(soon)* — how Windows centralizes naming,
  lifetime, and security.
- 🧠 **Token & Privilege Architecture** *(soon)*
- 🪣 **Kernel Pool Allocator** *(soon)* — where UAFs live.
- ⚡ **HVCI / KDP / VTL 1** *(soon)* — the hypervisor-backed defenses.

## Topics

`windows` `kernel-internals` `object-manager` `handle-table` `dkom`
`hvci` `kdp` `ppl` `edr` `defense-in-depth` `reverse-engineering`

## Sources

- *Windows Internals 7, Part 1 & 2* — Russinovich, Ionescu, Solomon
- [The Vergilius Project](https://www.vergiliusproject.com)
- [Microsoft Learn — Security](https://learn.microsoft.com/en-us/windows/security)
- Project Zero blog — for in-the-wild chain analysis

## Contributing

Spotted a wrong offset? A missing mitigation? PRs welcome.

```bash
git clone https://github.com/<your-username>/attvdef.git
```

## ⚠️ Disclaimer

For **defensive research, education, and authorized red-team testing
only**. The structures and bypasses described are public knowledge —
using them on systems you don't own is a crime in most jurisdictions.

## License

MIT — see [LICENSE](./LICENSE).

---

<div align="center">

**👉 [attvdef.mintlify.site](https://attvdef.mintlify.site/)**

⭐ Star · 🐛 Issue · 🍴 Fork

</div>
