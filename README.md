⚔️ Attacker vs Defender — Windows Kernel Internals Hub
<div align="center">
```
   ╔═══════════════════════════════════════════════════════╗
   ║                                                       ║
   ║   🛡️  DEFENDER          ⚔️          🗡️  ATTACKER      ║
   ║                                                       ║
   ║   Handle Table  ←──── clash ────→  DKOM / BYOVD      ║
   ║   HVCI / KDP    ←──── clash ────→  Kernel Write Prim ║
   ║   PPL / WDAC    ←──── clash ────→  LPE Chains        ║
   ║                                                       ║
   ╚═══════════════════════════════════════════════════════╝
```
🔗 Live Site → attvdef.mintlify.site
![Live Site](https://img.shields.io/badge/Live%20Site-attvdef.mintlify.site-18c99c?style=for-the-badge&logo=firefoxbrowser&logoColor=white)
![Topics](https://img.shields.io/badge/Topic-Windows%20Kernel-3b82f6?style=for-the-badge&logo=windows&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active%20Research-brightgreen?style=for-the-badge)
</div>
---
🎯 What is this?
A deep-dive documentation hub covering the eternal arms race between
Windows kernel defenders and offensive researchers — from the
ground up.
Each note walks a single structural or conceptual battle:
how the abstraction was built, how it was attacked, how Microsoft
responded, and what the next attacker move looks like. The result is a
living map of the Windows kernel security model — not a list of CVEs,
but the why behind each layer.
> 💡 **This README is just the map.** The territory lives at
> **[attvdef.mintlify.site](https://attvdef.mintlify.site/)** — open it
> in a new tab and follow along.
---
🧭 Start Here
#	Note	Why it matters
1	Windows Handle Table Internals	The foundation — every user-mode ↔ kernel-object interaction goes through here.
2	_EPROCESS Anatomy (soon)	The structure attackers love to patch. Token, ACLs, links, PPL flags.
3	The Object Manager (soon)	How Windows centralizes naming, lifetime, and security.
4	Kernel Pool Allocator (soon)	Where UAFs live, and why QUAD zones changed everything.
---
🛡️ The Defender's Stack
The site walks through every layer the kernel throws in front of an
attacker. Read top-to-bottom to understand the defense-in-depth model.
```
   ┌─────────────────────────────────────────────┐
   │  User-mode attacker                         │
   │  NtOpenProcess / NtCreateFile / NtDuplicate │
   └────────────────────┬────────────────────────┘
                        ▼
   ┌─────────────────────────────────────────────┐
   │  1. Handle Table Abstraction                │ ← hides layout
   │  _EPROCESS.ObjectTable → _HANDLE_TABLE      │
   └────────────────────┬────────────────────────┘
                        ▼
   ┌─────────────────────────────────────────────┐
   │  2. GrantedAccessBits (cached, immutable)   │ ← enforces ACL
   │  25-bit mask in _HANDLE_TABLE_ENTRY         │
   └────────────────────┬────────────────────────┘
                        ▼
   ┌─────────────────────────────────────────────┐
   │  3. ObRegisterCallbacks (EDR hook surface)  │ ← EDR strips bits
   │  Pre/post-operation callbacks               │
   └────────────────────┬────────────────────────┘
                        ▼
   ┌─────────────────────────────────────────────┐
   │  4. Protected Process Light (PPL)           │ ← anti-admin
   │  Protection-level comparison at open time   │
   └────────────────────┬────────────────────────┘
                        ▼
   ┌─────────────────────────────────────────────┐
   │  5. AppContainer / Sandbox boundary         │ ← isolates apps
   │  Capabilities-based access tokens           │
   └────────────────────┬────────────────────────┘
                        ▼
   ┌─────────────────────────────────────────────┐
   │  6. PatchGuard (KPP)                        │ ← kills hooks
   │  Periodic integrity scan of nt! / hal!      │
   └────────────────────┬────────────────────────┘
                        ▼
   ┌─────────────────────────────────────────────┐
   │  7. DSE / WHQL / Vulnerable Driver Blocklist│ ← gates drivers
   │  Code Integrity at image-load time          │
   └────────────────────┬────────────────────────┘
                        ▼
   ┌─────────────────────────────────────────────┐
   │  8. HVCI (Hypervisor-Protected CI)          │ ← no W^X bypass
   │  VTL 1 enforces kernel code page RO         │
   └────────────────────┬────────────────────────┘
                        ▼
   ┌─────────────────────────────────────────────┐
   │  9. KDP (Kernel Data Protection)            │ ← DKOM blocker
   │  VTL 1 marks _EPROCESS / token as RO        │
   └────────────────────┬────────────────────────┘
                        ▼
   ┌─────────────────────────────────────────────┐
   │ 10. Hardware: CET / IBT / MBEC              │ ← ROP/JOP killer
   │  CPU-enforced control-flow integrity        │
   └────────────────────┬────────────────────────┘
                        ▼
                  🎯 Kernel Object
            _OBJECT_HEADER → _OBJECT_TYPE
```
---
⚔️ The Attacker's Playbook
Every defense has a counter. The site documents both sides —
understanding the bypass is understanding the defense.
Attack class	Targets	Counter-defense on site
DKOM (Direct Kernel Object Manipulation)	`_EPROCESS.Token`, `GrantedAccessBits`, `ActiveProcessLinks`	Handle table abstraction · KDP
BYOVD (Bring Your Own Vulnerable Driver)	`rtcore64.sys`, `gdrv.sys`, `iqvw64e.sys`	WHQL · Vulnerable Driver Blocklist
Handle table UAF races	Recycled `_HANDLE_TABLE_ENTRY` slots	`StrictFIFO` free-list policy
PPL bypass	Anti-cheat drivers exposing physical memory	`RunAsPPL` · Microsoft-signed EDR
LSASS dumping	`lsass.exe` process memory	Credential Guard · LSA Protection
ROP / JOP chains	Return addresses, indirect branches	CET Shadow Stack · IBT
Kernel code injection	`NtAllocateVirtualMemory` in kernel	HVCI · ACG
EDR bypass	Direct syscalls, EDR-killing drivers	`ObRegisterCallbacks` · ETW-TI
---
📚 What you'll find on the site
📖 Long-form notes — not listicles, not blog posts. Real deep dives.
🧬 Annotated struct definitions — every field, every offset, every
security implication. Sources: Windows Internals 7 · Vergilius
Project · Microsoft Learn.
🎨 Mermaid diagrams — defense-in-depth stacks, lookup sequences,
data flows. All in a consistent dark neon theme.
🛠️ WinDbg cheat sheets — copy-paste commands to inspect every
structure live on your own system.
🐍 Python snippets — for folks building forensic tools.
🔗 Tabs for WinDbg / Python / Cheat-sheet on every note.
🌗 Dark mode — because kernel research happens at 3 AM.
---
🚀 How to use the site
Browse the sidebar. Notes are grouped by subsystem (Object
Manager, Memory Manager, Scheduler, …).
Use the right-hand TOC to jump between sections of a note.
Click the `<>` icon on any code block to copy it.
Follow the "Next" links at the bottom of each note — they're
curated to build understanding in order.
Star this repo if a note saved you an hour of reverse-engineering.
---
🗓️ Roadmap
[x] Windows Handle Table Internals
[x] DKOM & the modern mitigation stack
[x] Defense-in-depth diagram (sequence + flow)
[ ] _EPROCESS Anatomy — every field, every offset
[ ] The Object Manager — naming, lifetime, security
[ ] Token & Privilege architecture
[ ] Kernel Pool Allocator — QUAD zones, UAF mitigations
[ ] PPL deep dive — protection levels, signer levels
[ ] HVCI under the hood — VTL 1 walk-through
[ ] ETW Threat Intelligence — what EDRs see, what they don't
[ ] Syscall architecture — Win32k, Linux subsystem, filter mgr
---
🤝 Contributing
Spotted a wrong offset? A missing mitigation? A bypass that's not
listed? PRs welcome.
```bash
git clone https://github.com/<your-username>/attvdef.git
cd attvdef
# notes live in src/*.mdx — Mintlify components are supported
# preview locally:
npm i -g mintlify
mintlify dev
```
See `CONTRIBUTING.md` for the style guide and the
list of wanted notes.
---
📖 Primary Sources
Windows Internals 7, Part 1 & 2 — Russinovich, Ionescu, Solomon
The Vergilius Project — https://www.vergiliusproject.com
Microsoft Learn — Security — https://learn.microsoft.com/en-us/windows/security
Microsoft Security Response Center blog — for vulnerability write-ups
Project Zero blog — for in-the-wild chain analysis
Every note cites its sources inline. If a claim isn't sourced, treat it
as opinion.
---
⚠️ Disclaimer
This documentation is for defensive research, education, and
red-team authorized testing only. The kernel structures and bypasses
described are public knowledge — but using them on systems you don't
own or have explicit written permission to test is a crime in most
jurisdictions. Don't be that person.
---
📝 License
MIT — see `LICENSE`. The written content is yours to learn
from, quote, and build on. The Mintlify site config and theme belong to
their respective owners.
---
<div align="center">
🔗 attvdef.mintlify.site
⭐ Star the repo if it's useful · 🐛 Open an issue if it's not · 🍴 Fork if you want to extend
Built with Mintlify · Powered by curiosity · Maintained on caffeine
</div>