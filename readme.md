<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0a0a0f,50:1a0f00,100:b45309&height=220&section=header&text=WR4TH&fontSize=64&fontColor=ffffff&fontAlignY=38&fontAlign=50&desc=Windows%20Defender%20Evasion%20Research&descSize=16&descAlignY=58&descAlign=50&descColor=fcd34d80&animation=fadeIn" width="100%"/>

</div>

<br/>

<div align="center">

*Documented red team research into Windows Defender evasion at the payload delivery layer.*

</div>

<br/>

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=rect&color=0:b45309,100:1a0f00&height=1&section=header" width="40%"/>
</div>

<br/>

<div align="center">

### — Overview —

</div>

<br/>

WR4TH is a research project built to understand how attackers operate at the payload delivery and AV evasion layer — specifically, how Windows Defender's detection mechanisms work and where they fail.

The project implements a full cross-layer delivery chain from a Linux controller to a Windows target on the same network, using PowerShell stagers, Batch droppers, and an out-of-band SMTP exfiltration channel. The goal is not the tool itself — it's the documented understanding of why each technique works and how defenders can catch it.

<br/>

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=rect&color=0:b45309,100:1a0f00&height=1&section=header" width="40%"/>
</div>

<br/>

<div align="center">

### — Architecture —

</div>

<br/>

| Component | Layer | Function |
|:---|:---|:---|
| `main.py` | Controller | Python-based C2 interface running on Linux |
| `initial.bat` | Dropper | Entry point — fetches and executes the PowerShell stager |
| `java.bat` | Stager | Intermediate loader, pulls the primary payload |
| `system.ps1` | Payload | PowerShell script — installs SSH, establishes persistence, exfiltrates credentials |
| Mailtrap SMTP | Exfiltration | Out-of-band channel — delivers connection file to attacker inbox |
| SSH | Access | Persistent, full shell access to the compromised Windows host |

<br/>

**Delivery chain**

```
Attacker (Linux)
    └─ Victim executes initial.bat
            └─ Fetches java.bat → Fetches system.ps1
                    └─ Installs SSH · Establishes persistence
                            └─ Exfiltrates .rat file via SMTP
                                    └─ Attacker runs: python main.py target.rat
```

<br/>

**Defender Evasion — how it works**

The bypass targets Windows Defender's static and behavioural detection at the PowerShell execution layer. Key techniques:

- Execution policy manipulation to allow unsigned PS script execution
- Payload hosted remotely and fetched at runtime — nothing malicious lands on disk initially
- SMTP exfiltration uses a legitimate mail relay (Mailtrap) — no suspicious outbound C2 traffic pattern
- SSH is a legitimate Windows feature — persistence via a trusted binary, not injected code

> This is precisely why defenders need to monitor PowerShell download cradles, outbound SMTP from workstations, and unexpected SSH service installations — not just signature-based AV.

<br/>

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=rect&color=0:b45309,100:1a0f00&height=1&section=header" width="40%"/>
</div>

<br/>

<div align="center">

### — Capabilities —

</div>

<br/>

| Feature | Detail |
|:---|:---|
| AV Evasion | Windows Defender bypass — tested 4/10/2024 |
| Remote Access | Full SSH shell on target Windows host |
| File Operations | Upload, download, and remote execution |
| Persistence | Survives reboots via SSH service installation |
| Exfiltration | Out-of-band SMTP credential delivery |
| Interface | Terminal CLI — Linux operator side |

<br/>

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=rect&color=0:b45309,100:1a0f00&height=1&section=header" width="40%"/>
</div>

<br/>

<div align="center">

### — Demo —

</div>

<br/>

<div align="center">

[![Demo](http://i.ytimg.com/vi/2BFzcmkbUMM/hqdefault.jpg)](https://www.youtube.com/watch?v=2BFzcmkbUMM)

</div>

<br/>

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=rect&color=0:b45309,100:1a0f00&height=1&section=header" width="40%"/>
</div>

<br/>

<div align="center">

### — Setup —

</div>

<br/>

**Requirements**

- Arch / Debian Linux
- `sshpass` — `apt install sshpass`
- Python 3 + colorama — `pip install colorama`
- A [Mailtrap](https://mailtrap.io/) account (free tier)

<br/>

**Configuration**

```bash
# 1. Get Mailtrap SMTP credentials from your inbox → Email Testing
#    Copy Username and Password

# 2. In client/system.ps1 — replace on line 67:
USERNAME = "your_mailtrap_username"
PASSWORD = "your_mailtrap_password"

# 3. Upload system.ps1 to GitHub, copy the raw URL

# 4. In client/java.bat line 48 — replace with raw URL
# 5. In client/initial.bat line 6 — replace with raw URL
```

<br/>

**Running**

```bash
# After initial.bat executes on the target, wait for the SMTP email
# Download the .rat attachment into the WR4TH directory

python main.py filename.rat
```

<br/>

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=rect&color=0:b45309,100:1a0f00&height=1&section=header" width="40%"/>
</div>

<br/>

<div align="center">

### — Stack —

</div>

<br/>

**Languages** &nbsp;&nbsp; Python · PowerShell · Batch

**Techniques** &nbsp;&nbsp; AMSI Bypass · PS Execution Policy Manipulation · Out-of-Band Exfiltration · SSH Persistence

**Dependencies** &nbsp;&nbsp; colorama · sshpass · Mailtrap SMTP

<br/>

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=rect&color=0:b45309,100:1a0f00&height=1&section=header" width="40%"/>
</div>

<br/>

<div align="center">

### — Disclaimer —

</div>

<br/>

<div align="center">

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║   This tool is for educational and authorized research only.         ║
║                                                                      ║
║   Use on systems without explicit written permission is illegal.     ║
║                                                                      ║
║   The author is not responsible for any misuse.                      ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

</div>

<br/>

<div align="center">

*Built by [Adithya Poojary](https://github.com/NullSecurity07)*
&nbsp;·&nbsp;
[LinkedIn](https://www.linkedin.com/in/adithyapoojary/)
&nbsp;·&nbsp;
[nullsec0001@proton.me](mailto:nullsec0001@proton.me)

<br/>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:b45309,50:1a0f00,100:0a0a0f&height=120&section=footer&animation=fadeIn" width="100%"/>

</div>
