# 00 — Introduction: Windows Internals and Privilege Escalation for Real World Red Team Operators

---

## 1. What This Course Is

This course teaches you Windows Internals and Privilege Escalation from the ground up, with one clear goal: to get you skilled enough to work as a real world red team operator.

Not a CTF player. Not a certification holder. A person that companies actually hire to break into their own infrastructure and tell them what a real attacker could have done.

The course assumes you have basic pentesting knowledge. You know what a reverse shell is, you have used tools like Nmap, Burp Suite, or Metasploit at some point, and you understand what a vulnerability is. That is all you need coming in.

You do not need to know anything about Windows internals. You do not need to know anything about red teaming. And you do not need to know any C programming. This course starts from zero on all of those things and builds up steadily.

Each file in this course covers one topic. Every topic is explained the way you would explain it to a friend sitting next to you — in plain language, with real examples, and with enough depth that you actually understand what is happening instead of just memorizing steps.

---

## 2. What a Red Team Operator Actually Is

### The simple version

A red team operator is someone a company pays to pretend to be a real attacker, break into their systems using the same methods real attackers use, and then tell the company exactly what happened, how far in they got, and what it means.

The key word is "pretend to be a real attacker." That is not what a normal pentester does, and the difference matters a lot.

### How a red team operator is different from a normal pentester

A regular pentester is given a scope. They are told "test these 20 IP addresses" or "test this web application." They find vulnerabilities, confirm they are real, write a report, and that is the job. The security team usually knows a test is happening. The goal is to find and document as many vulnerabilities as possible.

A red team operator works completely differently. They are given an objective — something like "get to the payroll database" or "prove you can access the CEO's email" — and no one in the security team except the most senior people knows the test is happening. The red team has no restrictions on how they get there. If they need to call someone and pretend to be IT support, they can. If they need to send a phishing email, they can. If they need to break into the building, they can. The goal is not to find every vulnerability. The goal is to achieve the objective while staying hidden, exactly like a real attacker would.

Here is what that means day to day for a red team operator:

- Planning a full attack campaign before it starts, mapping out likely paths from the internet to the target
- Setting up command and control (C2) infrastructure — the servers the malware calls home to
- Getting initial access through phishing, exploiting a public-facing application, or compromising credentials
- Moving through the network without triggering alarms, one machine at a time
- Escalating privileges on Windows machines to get from a normal user account to SYSTEM or Domain Admin
- Staying persistent so the access survives reboots and security scans
- Stealing the data or reaching the objective the engagement was designed to test
- Writing detailed reports that explain what happened technically and what it means for the business
- Sometimes running "purple team" exercises where the red and blue teams work together to improve detection

That last escalation step — getting from a normal user to SYSTEM — is exactly what this course focuses on. It is one of the most important skills a red team operator needs, and it requires deep Windows Internals knowledge to do properly.

### Why Windows Internals is what separates a real red team operator from a tool runner

There are a lot of people who can run WinPEAS, copy the output, Google the findings, and run a privilege escalation exploit. Companies do not pay well for that. What they pay well for is someone who can look at a machine, understand what is actually happening under the hood, spot something that WinPEAS missed, write a custom payload that gets past the EDR, and explain the whole thing in a way that makes the client understand why it is dangerous.

Windows Internals is the knowledge that lets you do all of that. When you understand how Windows loads DLLs, you can find DLL hijacking opportunities that no automated tool will ever flag. When you understand how access tokens work, you know exactly when and why a token impersonation attack will work, not just which tool to run. When you understand how the NT API differs from the Win32 API, you understand why direct syscalls bypass EDR hooks and how to write code that does it.

Every technique in this course makes much more sense once you understand the internals behind it. That is why the course is structured this way — internals first, then attacks.

---

## 3. Why This Course Focuses on Misconfigurations

Real enterprise Windows machines are full of misconfigurations. Not because the people who set them up were careless, but because Windows has a lot of moving parts, organizations are large, configurations change over time, software gets installed by vendors with weak defaults, and no one goes back and audits every service, every registry key, and every DLL load path.

Misconfigurations are interesting for a specific reason: they exist on every machine, patched or not. A CVE exploit works only on systems that have not applied the specific patch for that vulnerability. As soon as the organization patches, the exploit stops working. But a misconfiguration — like a service running as SYSTEM with a binary that any user can replace — exists regardless of patch level. It exists because of how the software was set up, not because of a bug in the code.

This means that skills built around misconfigurations travel with you. Every engagement, every machine, every organization will have some misconfigurations. The attacker who knows what to look for will find something. The attacker who only knows how to run CVE exploits will be stuck the moment the machines are patched.

That said, some CVEs are important enough that every red team operator needs to know them. HiveNightmare (CVE-2021-36934) lets any standard user read the SAM database and extract all local password hashes, and it affects millions of Windows 10 and 11 machines even today. BYOVD (Bring Your Own Vulnerable Driver) is the technique ransomware groups use to kill EDR at the kernel level, and it is one of the most discussed attack techniques of 2024 and 2025. These are covered in full.

---

## 4. What You Will Learn — Every File in This Course

### 00 — Introduction (this file)
What the course is, what a red team operator does, why misconfigs matter, tools used, career information, and how to get started.

### 01 — Lab Setup
**What it is:** How to build a safe Windows lab on your own machine where you can practice everything in this course without touching real systems.

**Why a red team operator needs this:** You cannot practice privilege escalation on other people's machines. You need your own lab where you can break things, reset them, and try again. Building a good lab also teaches you Windows configuration, which helps you spot misconfigs faster.

**Real world use:** Most red team operators maintain a personal lab and a work lab. They test new tools and techniques there before using them on actual engagements.

### 02 — Windows Architecture
**What it is:** How Windows is built from the inside — user mode vs kernel mode, what ring levels are, how the different parts of Windows talk to each other, how Windows boots from power button to desktop.

**Why a red team operator needs this:** Every single attack technique in this course interacts with Windows architecture in some way. Token impersonation touches the kernel. EDR evasion requires understanding what runs in user mode vs kernel mode. BYOVD targets the kernel directly. Without this foundation, the attacks are just magic tricks you memorize.

**Real world use:** When a red teamer writes a payload that uses direct syscalls to avoid EDR detection, they are exploiting the fact that they understand the boundary between user mode and kernel mode and know exactly where the EDR can and cannot hook code.

### 03 — Processes and Threads
**What it is:** What a process is in Windows, what a thread is, how they are created, what the Process Environment Block (PEB) and Thread Environment Block (TEB) are, and how Windows tracks running programs.

**Why a red team operator needs this:** Process injection — putting your code inside another process to hide from security tools — is a core red team technique. To do it correctly you need to understand how processes and threads work. The PEB is also a goldmine of information and is used in many evasion and injection techniques.

**Real world use:** During an engagement, a red team operator injects a payload into a legitimate process like `explorer.exe` or `svchost.exe` so that the C2 traffic looks like it is coming from a trusted Windows process. This requires process and thread manipulation.

### 04 — Windows Memory
**What it is:** How Windows manages memory — virtual memory, physical memory, paging, what the stack and heap are, what memory permissions are, and why processes cannot read each other's memory by default.

**Why a red team operator needs this:** Shellcode runs in memory. Payloads are written to allocated memory regions. EDR detection often focuses on suspicious memory allocations. Understanding memory lets you understand why certain shellcode injection techniques work and why others get caught.

**Real world use:** When a red team operator allocates memory with `VirtualAllocEx` and then uses `WriteProcessMemory` to write shellcode into another process, every single one of those API calls maps directly to memory management concepts explained in this file.

### 05 — Windows Registry
**What it is:** What the Windows registry is, how it is organized into hives and keys, what autorun keys are, and why the registry is so powerful from an attacker's perspective.

**Why a red team operator needs this:** The registry is used in multiple attack techniques — autorun persistence, AlwaysInstallElevated privilege escalation, weak registry ACL attacks. You cannot use these techniques properly if you do not understand how the registry is structured.

**Real world use:** A red team operator on a post-exploitation engagement adds a key to `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` to make their payload survive reboots. This is one of the most common persistence techniques used by real attackers.

### 06 — Windows Services
**What it is:** What Windows services are, how they start and run, what the Service Control Manager (SCM) is, what service accounts are, and why services running as SYSTEM are so interesting to attackers.

**Why a red team operator needs this:** Three of the biggest privilege escalation techniques — weak service permissions, unquoted service paths, and weak binary permissions — all target Windows services. You need to understand how services work before you can attack them.

**Real world use:** During an engagement, a red team operator finds that a third-party backup software installs a service with an unquoted path in `C:\Program Files\Backup Solution\Service\backup.exe`. They drop a malicious `Backup.exe` into `C:\Program Files\` and when the service restarts, it runs the malicious binary as SYSTEM.

### 07 — Tokens and Privileges
**What it is:** What an access token is, what privileges are, what the most important privilege flags are — specifically SeImpersonatePrivilege and SeDebugPrivilege — and why attackers want them so badly.

**Why a red team operator needs this:** Token manipulation is one of the most reliable ways to escalate privileges on Windows, especially from service accounts. IIS worker processes, SQL Server service accounts, and many other services run with SeImpersonatePrivilege. Knowing this tells you immediately that these accounts are vulnerable to impersonation attacks.

**Real world use:** A red team operator gets code execution as the `iis apppool\defaultapppool` account through a web shell. They check the token privileges, see SeImpersonatePrivilege, and immediately know they can use GodPotato or SigmaPotato to get SYSTEM. This is a textbook post-exploitation path.

### 08 — Windows API and NT API
**What it is:** What the Win32 API is, what the NT API (native API) is, how they differ, why `CreateFile` in Win32 eventually becomes `NtCreateFile` in the NT API, and why this chain matters for EDR evasion.

**Why a red team operator needs this:** EDR products hook specific functions in the Win32 API and in NTDLL (which exposes the NT API) to watch what programs are doing. When you understand this chain, you understand exactly where the hooks sit and how to go around them using direct syscalls.

**Real world use:** A red team operator needs to allocate memory in a remote process to inject shellcode. They know that `VirtualAllocEx` from the Win32 API will be hooked by the EDR. So instead of calling `VirtualAllocEx`, they call `NtAllocateVirtualMemory` directly at the syscall level, bypassing the hook entirely.

### 09 — DLL Basics
**What it is:** What a DLL (Dynamic Link Library) is, how Windows finds and loads DLLs when a program starts, what the DLL search order is, and why this order creates attack opportunities.

**Why a red team operator needs this:** DLL hijacking works because Windows searches for DLLs in a specific order of folders. If you can write a malicious DLL into a folder that Windows checks before the real DLL's location, Windows will load your DLL instead. This is one of the most commonly used persistence and privilege escalation techniques.

**Real world use:** A red team operator finds that a software application running as SYSTEM tries to load a DLL called `version.dll` and looks in the application's own folder first. That folder has weak permissions. They drop a malicious `version.dll` there, restart the service, and get SYSTEM.

### 10 — Manual Enumeration
**What it is:** How to find privilege escalation opportunities on a Windows machine using only built-in Windows commands, without running any tools that might be detected.

**Why a red team operator needs this:** On a real engagement, running WinPEAS will trigger almost every modern EDR. A skilled operator needs to be able to do manual recon using `wmic`, `sc`, `icacls`, `reg query`, `whoami`, and other built-in commands. This is what separates an operator from someone who just runs scripts.

**Real world use:** During a red team engagement where the client has Crowdstrike deployed, an operator gains a low-privilege shell and manually queries service configurations, checks folder ACLs, and reviews token privileges using nothing but Windows built-in commands. No alerts fire.

### 11 — Automated Enumeration
**What it is:** How to use tools like WinPEAS, Seatbelt, and PrivescCheck to quickly find privilege escalation opportunities, and when it is safe to use them vs when manual is better.

**Why a red team operator needs this:** On engagements where EDR coverage is light or where speed matters, automated tools find misconfigs in seconds that would take minutes manually. Knowing how these tools work also helps you understand what they miss.

**Real world use:** During an internal engagement against a company with no EDR deployed, an operator runs WinPEAS and finds three privilege escalation paths in under a minute: an unquoted service path, an AlwaysInstallElevated registry key, and a writable service binary. They pick the cleanest one and escalate.

### 12 — Service Misconfiguration Attacks
**What it is:** Three specific service-based privilege escalation attacks in full detail — weak service permissions (you can change what a service runs), unquoted service paths (Windows tries to run your binary by mistake), and weak binary permissions (you can replace the file the service runs).

**Why a red team operator needs this:** Service misconfigurations are the most common privilege escalation finding on real world internal engagements. Almost every network has at least one, because third-party software installs services with bad defaults constantly.

**Real world use:** A red team operator is on a machine where a vendor-installed monitoring agent runs as SYSTEM with an unquoted path of `C:\Program Files\Monitoring Agent\Service\agent.exe`. They write a malicious exe to `C:\Program Files\Monitoring.exe`, wait for the next reboot or trigger a service restart, and get a SYSTEM shell.

### 13 — Token Impersonation
**What it is:** How token impersonation works technically, what SeImpersonatePrivilege enables, and how tools like GodPotato, PrintSpoofer, and SigmaPotato use this to escalate to SYSTEM. Which tool to use on which Windows version.

**Why a red team operator needs this:** Token impersonation is the go-to escalation technique when you land on a machine as a service account. IIS, MSSQL, and many other service accounts have SeImpersonatePrivilege by default. This technique works on all Windows versions including the latest ones.

**Real world use:** A red team operator exploits an MSSQL injection vulnerability and gets `xp_cmdshell` execution as the `NT Service\MSSQLSERVER` account. This account has SeImpersonatePrivilege. They upload SigmaPotato, run it, and get SYSTEM in under 30 seconds.

### 14 — DLL Hijacking
**What it is:** All the modern variants of DLL hijacking — classic DLL search order hijacking, DLL sideloading, phantom DLL hijacking. How to find hijackable DLLs using Process Monitor. How to write a malicious DLL in C, taught from zero.

**Why a red team operator needs this:** DLL hijacking is one of the most used techniques for both privilege escalation and persistence. It is also heavily used by real malware. Understanding it makes you better at both writing attacks and detecting them.

**Real world use:** A red team operator uses Process Monitor to watch what DLLs a privileged application tries to load. They see it tries to load `wbemcomn.dll` from a user-writable directory and fails to find it there. They compile a malicious DLL with that name, drop it in that directory, and when the application runs, it loads and runs their code.

### 15 — Registry Attacks
**What it is:** How to use registry weaknesses for privilege escalation — autorun abuse for persistence, AlwaysInstallElevated to install MSI packages as SYSTEM, and weak registry ACLs that let you change what a service runs.

**Why a red team operator needs this:** Registry-based attacks are common on older enterprise machines and are often missed by IT teams. AlwaysInstallElevated in particular is still found regularly on enterprise machines that have been poorly configured.

**Real world use:** A red team operator queries the registry and finds both `HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated` and `HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated` set to 1. They craft a malicious MSI file that adds them to the Administrators group and install it. Windows installs it as SYSTEM.

### 16 — UAC Bypass
**What it is:** What User Account Control (UAC) is, why it is not a true security boundary, and how to bypass it using the fodhelper technique and the eventvwr technique — without needing a password.

**Why a red team operator needs this:** On Windows machines, even a user in the Administrators group is running at medium integrity until they accept a UAC prompt. If you get code execution as an admin user but cannot trigger a UAC popup, these bypass techniques let you elevate to high integrity silently.

**Real world use:** A red team operator is running as a local administrator but their process is medium integrity (UAC is blocking full admin access). They use the fodhelper bypass to launch a high-integrity process without triggering a UAC dialog. No prompt appears. They now have full admin access.

### 17 — Scheduled Task Abuse
**What it is:** How Windows scheduled tasks work, how to find tasks with weak permissions, how to replace the binary a privileged task runs, and how to create your own persistence through tasks.

**Why a red team operator needs this:** Scheduled tasks are a very common persistence mechanism used by both red teams and real malware. They also create privilege escalation opportunities when the task runs as SYSTEM but the binary it calls can be replaced by a low-privilege user.

**Real world use:** A red team operator finds a scheduled task that runs daily as SYSTEM, calling `C:\Temp\cleanup.bat`. The `C:\Temp` folder has write permissions for all users. They replace `cleanup.bat` with a command that adds their user to the local Administrators group. The next morning, the task runs and they have admin access.

### 18 — BYOVD (Bring Your Own Vulnerable Driver)
**What it is:** What BYOVD is, why kernel-level access makes EDRs completely blind, how ransomware groups like Qilin, BlackByte, and Kasseika have used this in real attacks, what the LOLDrivers database is, and how attackers use signed vulnerable drivers to kill security tools.

**Why a red team operator needs this:** BYOVD is one of the most discussed attack techniques in 2024 and 2025. Real ransomware groups use it to blind EDR tools before deploying ransomware. Red team operators who understand this technique can simulate the exact TTPs of advanced threat groups.

**Real world use:** During a red team engagement simulating a ransomware group, an operator loads a signed but vulnerable driver from the LOLDrivers database into the target machine. The driver has a vulnerability that allows kernel-level process termination. They use it to kill the client's CrowdStrike sensor, then operate freely with no EDR coverage for the rest of the engagement.

### 19 — Stored Credentials
**What it is:** How to find stored credentials on Windows machines — saved credentials in cmdkey, passwords in unattended installation files, credentials in the SAM database, and HiveNightmare (CVE-2021-36934) which lets any user read the SAM on affected Windows 10 and 11 machines.

**Why a red team operator needs this:** Credentials on disk are one of the easiest wins on a post-exploitation engagement. Organizations often leave sensitive credentials in configuration files, installation scripts, and the Windows credential manager. Finding one set of credentials can chain into lateral movement across the entire network.

**Real world use:** During an engagement, a red team operator finds `C:\Windows\Panther\unattend.xml` — a file left over from an automated Windows installation — containing a local Administrator password in plaintext. They use it to authenticate as Administrator on five other machines on the same network. The whole internal network is compromised from one forgotten file.

### 20 — AMSI Bypass
**What it is:** What the Antimalware Scan Interface (AMSI) is, how it inspects PowerShell and other scripting runtimes before they execute code, and how to disable it so your payloads run without being scanned.

**Why a red team operator needs this:** AMSI is the reason that running popular offensive PowerShell scripts like PowerView or PowerUp gets you caught immediately. Every red team operator needs to know how to get past AMSI before they can use these tools effectively.

**Real world use:** A red team operator tries to run PowerView on a target machine and it gets blocked by Windows Defender instantly, because AMSI scanned the script and flagged it. They apply an AMSI bypass in memory first — which patches the scanning function to always return "clean" — and then run PowerView without any issues.

### 21 — ETW Patching
**What it is:** What Event Tracing for Windows (ETW) is, how EDR products use ETW to collect telemetry about what processes are doing, and how to patch the ETW write function in memory to stop it from sending data.

**Why a red team operator needs this:** EDR tools get a lot of their detection capability from ETW. When you patch ETW in your process, the EDR becomes blind to what that process is doing. It is one of the standard techniques used in modern loaders and red team tools.

**Real world use:** A red team operator's C2 implant patches ETW inside its own process by writing a `RET` instruction over the start of the `EtwEventWrite` function. The EDR stops receiving telemetry from that process. The operator can now run sensitive operations from that process without triggering behavioral detections.

### 22 — Direct Syscalls
**What it is:** What system calls are, how EDR hooks work by sitting in NTDLL and watching every API call, and how direct syscalls let your code talk straight to the Windows kernel without going through NTDLL — completely bypassing every EDR hook.

**Why a red team operator needs this:** Modern EDR evasion has largely moved to syscall-level techniques. Direct syscalls are now standard in professionally written red team tooling. Understanding them is what puts you at the level of a real tool developer, not just a tool user.

**Real world use:** A red team operator writes a custom payload that allocates memory and injects shellcode using direct syscalls instead of the standard Win32 API. The EDR, which hooks the NTDLL functions, never sees these calls happen. The payload runs without triggering any EDR alert.

---

## 5. C Programming in This Course

You do not need to know any C before starting this course.

C comes up in two files: DLL hijacking (file 14) and direct syscalls (file 22). In both of those files, C will be taught from scratch — starting with what a variable is, how to compile a file, and working up step by step to the specific technique being covered. You will write working code by the end of each of those files even if you have never written a line of C before.

The reason this course uses C and not Python or PowerShell for those topics is that DLLs have to be compiled from C (or C++), and direct syscalls require writing raw assembly inline in C code. There is no shortcut around this. But the good news is that you do not need deep C expertise — you only need to understand specific patterns, and those will be explained in full.

---

## 6. Tools Used in This Course

### WinPEAS
**What it is:** A script that automatically checks a Windows machine for hundreds of known privilege escalation paths — misconfigs, weak permissions, stored credentials, and more.

**What you use it for:** Fast automated enumeration at the start of post-exploitation. Gives you a prioritized list of things to investigate further.

### Seatbelt
**What it is:** A C# tool that does deep reconnaissance of a Windows machine — installed software, running processes, network connections, credential stores, PowerShell history, and much more.

**What you use it for:** Thorough host reconnaissance after gaining access. Better than WinPEAS for operational security checks and detailed host profiling.

### PrivescCheck
**What it is:** A PowerShell script focused specifically on finding privilege escalation paths. Produces clean, structured output that is easy to read.

**What you use it for:** A second opinion alongside WinPEAS, especially good at service and task misconfigurations. Easier to read than WinPEAS output.

### GodPotato
**What it is:** A token impersonation tool that works by abusing the DCOM activation service.

**What you use it for:** Escalating from a service account with SeImpersonatePrivilege to SYSTEM. Works on Windows Server 2012 through Windows 11 and Windows Server 2022.

### PrintSpoofer
**What it is:** A tool that uses named pipes and the Print Spooler service to impersonate SYSTEM.

**What you use it for:** Same as GodPotato — escalating from SeImpersonatePrivilege to SYSTEM. Specifically good for Windows 10 and Windows Server 2016/2019 when the DCOM path used by GodPotato is blocked.

### SigmaPotato
**What it is:** A newer fork of GodPotato with extended OS support and the ability to run entirely in memory using .NET reflection — meaning the binary never has to touch disk.

**What you use it for:** Token impersonation on engagements where you need to avoid writing files to disk. The in-memory execution makes it much harder for EDR to catch. Works on Windows 8 through Windows 11 and Windows Server 2012 through 2022.

### Process Hacker
**What it is:** A free, open source tool that shows you everything about running processes — their tokens, handles, memory regions, loaded DLLs, and privileges — in a clear graphical interface.

**What you use it for:** Understanding what is happening inside Windows processes during lab work. When learning token impersonation or DLL hijacking, Process Hacker lets you see exactly what the OS is doing under the hood.

### x64dbg
**What it is:** A free debugger for 64-bit Windows applications. It lets you step through program execution one instruction at a time and see exactly what the program is doing in memory.

**What you use it for:** Analyzing how tools and payloads work internally, understanding assembly, and learning how EDR hooks are placed in NTDLL. Useful when writing custom offensive tools.

### WinDbg
**What it is:** Microsoft's official kernel debugger. It can attach to the Windows kernel itself and let you inspect kernel structures, set kernel breakpoints, and understand how the OS works at the deepest level.

**What you use it for:** Advanced Windows internals research. When you need to understand how a kernel structure like the EPROCESS block or the token object looks in memory, WinDbg is the tool you use.

### Sysinternals Suite
**What it is:** A collection of free tools made by Microsoft (originally by Mark Russinovich) that give you deep visibility into what Windows is doing. The most important ones are Process Monitor (Procmon), Process Explorer, Autoruns, and AccessChk.

**What you use it for:**
- **Process Monitor:** Watching every file, registry, and network operation a program makes in real time. Essential for finding DLL hijacking opportunities.
- **Process Explorer:** A better version of Task Manager that shows process trees, loaded DLLs, and handles.
- **Autoruns:** Shows every single place where code can auto-start on Windows boot. Critical for finding persistence.
- **AccessChk:** Checks permissions on services, files, registry keys, and objects. Essential for service misconfiguration attacks.

### LOLDrivers
**What it is:** A community-maintained database of legitimate, signed Windows drivers that contain known vulnerabilities. The name stands for "Living Off the Land Drivers."

**What you use it for:** Finding vulnerable drivers for BYOVD attacks. The database includes the vulnerability details, the CVE number, and sometimes the exact code needed to exploit it.

### Visual Studio
**What it is:** Microsoft's official IDE (Integrated Development Environment) for writing and compiling C and C++ code on Windows.

**What you use it for:** Writing and compiling malicious DLLs, custom payloads, and any other Windows code in this course. It is the standard tool for Windows C development.

### mingw-gcc
**What it is:** A free compiler that lets you compile C code on Linux into Windows executables and DLLs. The name stands for "Minimalist GNU for Windows."

**What you use it for:** Cross-compiling Windows DLLs and executables from a Linux machine. This is how most red teamers compile their Windows tools — they work on Kali Linux and use mingw-gcc to produce the final Windows binary. You do not need a Windows machine just to compile code.

---

## 7. Career — What This Gets You

### Jobs you can apply for after this course

The knowledge in this course is directly applicable to these roles:

- **Red Team Operator** — The target job. You plan and run adversary simulation engagements against real company infrastructure.
- **Penetration Tester (Windows/Internal)** — Internal network pentesting with a focus on Windows Active Directory environments.
- **Vulnerability Researcher (Windows)** — Finding new vulnerabilities in Windows software and components.
- **Offensive Security Engineer** — Building internal red team tools and infrastructure at large companies.
- **Threat Intelligence Analyst (Technical)** — Understanding how real attackers work to help defenders build better detection.
- **Malware Analyst** — Understanding Windows internals well enough to reverse engineer what malware is doing.

The Windows Internals knowledge from this course is also a prerequisite for malware development roles and for advanced courses like writing your own C2 from scratch or kernel exploitation.

### Companies that hire red team operators in India

In India, the companies that regularly have red team operator positions include:

- **Consulting and professional services:** KPMG, Deloitte, PwC, Ernst & Young (EY), Accenture, IBM Security, Wipro, TCS, Infosys (they all have dedicated red team practices)
- **Financial sector:** Citi, Barclays India, JPMorgan India, HDFC Bank, ICICI Bank (banks invest heavily in red teaming because of RBI requirements)
- **Technology companies:** Microsoft India, Paytm, Razorpay, Flipkart, Swiggy
- **Security-focused companies:** Secureworks, Mandiant (now part of Google), CrowdStrike India

Cities where most positions are: Bengaluru, Hyderabad, Pune, Mumbai, Noida/Gurgaon (NCR).

### Companies that hire red team operators globally

- **United States:** NSA, CISA, MITRE (government and defense), Mandiant, CrowdStrike, Rapid7, NCC Group, Bishop Fox (security firms), Microsoft, Google, Amazon, Meta (tech giants all have internal red teams)
- **United Kingdom:** NCSC, BAE Systems, GCHQ (government), NCC Group, Pen Test Partners
- **Europe:** Darktrace, Truesec, NVISO
- **Global:** Big Four consulting firms (Deloitte, PwC, KPMG, EY) have red team practices in almost every country

### Realistic salary

**India:**
- Entry level (0–2 years experience): ₹6 lakh – ₹12 lakh per year
- Mid level (2–5 years, real red team skills): ₹15 lakh – ₹30 lakh per year
- Senior (5+ years, team lead, tool development): ₹30 lakh – ₹60 lakh+ per year

The wide range reflects the difference between someone who just has certifications vs someone who has real hands-on Windows Internals knowledge and can build custom tooling. This course is designed to get you toward the higher end of those ranges.

**Globally (USD):**
- Entry level: $80,000 – $120,000
- Mid level: $120,000 – $180,000
- Senior / team lead: $180,000 – $250,000+

US Government and defense contractor red team roles often come with security clearances and can pay significantly higher.

### What hands-on knowledge gives you that certifications alone do not

Certifications like OSCP, CRTO, and CRTE prove that you passed a timed exam in a lab environment. Interviewers at serious companies know this. They test deeper.

When you interview for a red team operator role at a company like KPMG, Mandiant, or CrowdStrike, interviewers ask things like: "Walk me through how you would escalate privileges on a fully patched Windows 11 machine with CrowdStrike deployed." A certification holder says "I would run WinPEAS." A real operator walks through the thought process — checking token privileges first, looking for service misconfigurations manually to avoid EDR, checking DLL load paths for high-privilege processes, and explaining exactly why each step is chosen.

This course builds that kind of knowledge. The internals, the reasoning, and the hands-on technique together. That combination is what gets you hired.

---

## 8. What You Need Before Starting

- Basic Kali Linux usage — you know how to open a terminal, run commands, and navigate the file system
- Basic pentesting knowledge — you have some idea of what a reverse shell is and have used at least one tool like Nmap or Metasploit
- A machine with at least 16GB RAM — you will be running two virtual machines at the same time in your lab (a Windows machine and a Kali machine)
- Nothing else — no C knowledge, no Windows internals knowledge, no red team experience needed

If your machine has less than 16GB RAM, you can still follow along with the concepts. The lab files will tell you the minimum specs to run each exercise.

---

## 9. Course Folder Structure

Here is what each file covers:

| File | Topic |
|------|-------|
| 00_intro.md | This file — course overview, career info, tools |
| 01_lab_setup.md | Building your Windows + Kali practice lab |
| 02_windows_architecture.md | User mode, kernel mode, ring levels, Windows boot |
| 03_processes_and_threads.md | Processes, threads, PEB, TEB |
| 04_windows_memory.md | Virtual memory, paging, stack, heap |
| 05_windows_registry.md | Registry structure, hives, keys |
| 06_windows_services.md | Services, SCM, service accounts |
| 07_tokens_and_privileges.md | Access tokens, privilege flags, SeImpersonate |
| 08_windows_api_and_ntapi.md | Win32 API, NT API, syscall chain |
| 09_dll_basics.md | DLLs, load order, search path |
| 10_manual_enumeration.md | Manual recon with built-in Windows commands |
| 11_automated_enumeration.md | WinPEAS, Seatbelt, PrivescCheck usage |
| 12_service_misconfigs.md | Weak service perms, unquoted paths, binary replacement |
| 13_token_impersonation.md | GodPotato, PrintSpoofer, SigmaPotato |
| 14_dll_hijacking.md | DLL search order abuse, sideloading, C from scratch |
| 15_registry_attacks.md | AlwaysInstallElevated, autorun, registry ACLs |
| 16_uac_bypass.md | UAC explained, fodhelper, eventvwr bypass |
| 17_scheduled_tasks.md | Task abuse, weak task permissions, persistence |
| 18_byovd.md | Vulnerable driver abuse, kernel-level EDR killing |
| 19_stored_credentials.md | cmdkey, unattend files, SAM, HiveNightmare |
| 20_amsi_bypass.md | AMSI internals, bypass techniques |
| 21_etw_patching.md | ETW internals, patching EtwEventWrite |
| 22_direct_syscalls.md | Syscall mechanics, bypassing EDR hooks |

**Your next step: open `01_lab_setup.md`.**

---

## 10. What You Learned in This File

- A red team operator is not a pentester. They simulate real attackers with no scope restrictions, operating covertly, and targeting specific objectives.
- The difference between a tool runner and a real operator is Windows Internals knowledge — understanding what is happening under the hood, not just which tool to run.
- This course focuses on misconfigurations rather than CVE exploits because misconfigs exist everywhere, even on fully patched machines. CVE exploits only work until a patch is applied.
- The two CVEs covered in full (HiveNightmare and BYOVD) are important because they are still highly relevant in 2024 and 2025 and are used by real threat groups.
- C programming is needed for two files in this course and will be taught from scratch in those files. No prior C knowledge needed.
- The tools in this course cover both manual and automated work — the skill is knowing when to use which one and understanding why each one does what it does.
- Red team operator is a well-paid job in India (₹15–60L mid to senior range) and globally ($120K–$250K+), and this knowledge differentiates you from certification-only candidates.

---

## 11. Interview Questions

These are the types of questions you will face in real red team operator and senior pentester interviews. Read the question, cover the answer, try to answer it yourself first, then check below.

---

### Theory Questions

**Question 1: What is privilege escalation and why does it matter in a real engagement?**

**Full answer:** Privilege escalation is the act of going from a lower level of access to a higher level of access on a system without being explicitly given that higher access. On Windows, this usually means going from a normal user account to SYSTEM (the highest local account) or from a standard domain user to Domain Admin (the highest Active Directory account).

It matters in a real engagement because initial access almost never gives you the level of access you need to reach the engagement's objective. A web shell on an IIS server gives you the permissions of the IIS service account, which cannot read the payroll database or access the Domain Controller. Privilege escalation is the step that closes that gap. Without it, you have a foothold but no impact. With it, you can demonstrate the full realistic impact of a breach.

**How to say this confidently in an interview:** Speak slowly and in full sentences. Start with the definition, then immediately explain why it matters in context. Avoid saying "it allows you to get higher permissions" — that is too vague. Say "it takes you from, for example, an IIS service account with no real access to SYSTEM, and from there you can dump credentials, move laterally, and reach the actual objectives the engagement was testing."

---

**Question 2: What is the difference between user mode and kernel mode in Windows, and why does it matter for offensive security?**

**Full answer:** Windows runs programs at different levels of privilege to protect the operating system from crashes and malicious code. User mode is where regular applications run — your browser, Word, Notepad. Code running in user mode cannot directly access hardware, cannot read another process's memory, and cannot modify OS data structures. If a user-mode program crashes, only that program dies. The OS keeps running.

Kernel mode is where the Windows kernel, device drivers, and critical OS code runs. Code in kernel mode has direct access to all memory, all hardware, and all system structures. If kernel-mode code crashes, the whole system crashes — that is what a Blue Screen of Death is.

For offensive security this matters enormously. EDR products run their detection hooks in user mode (specifically in a library called NTDLL). This means they can be patched out or bypassed by a program running in the same user mode. BYOVD attacks take this further — by getting code running in kernel mode, an attacker is above the EDR, which cannot protect itself from kernel-level code. The attacker can simply terminate EDR processes from kernel mode with no resistance.

**How to say this confidently in an interview:** Draw the mental picture for the interviewer. Say "Think of it as two floors — user mode is the ground floor where applications run, kernel mode is the basement where the OS itself runs. EDR hooks live on the ground floor, so an attacker who can go to the basement — either through BYOVD or a kernel exploit — is completely out of reach of those hooks."

---

**Question 3: What is SeImpersonatePrivilege and why do attackers want it?**

**Full answer:** SeImpersonatePrivilege is a Windows privilege that allows a process to impersonate another user's security context. Specifically it means the process can take a token that represents another user and use it to run code as that user.

Attackers want it because it is the key to tools like GodPotato, PrintSpoofer, and SigmaPotato. These tools force the SYSTEM account (which runs at the highest privilege on a local machine) to authenticate to a local service controlled by the attacker. When SYSTEM authenticates, it hands over a SYSTEM token. Because the attacker's process has SeImpersonatePrivilege, it can take that SYSTEM token and use it to spawn a new process running as SYSTEM.

The reason this is so practical is that many service accounts have SeImpersonatePrivilege by default. IIS application pool accounts, MSSQL service accounts, and many other service accounts are set up this way. So getting code execution as any of these service accounts — through a web shell, SQL injection, or another technique — immediately gives you a path to SYSTEM.

**How to say this confidently in an interview:** Explain it as a chain: "SeImpersonatePrivilege lets you take someone else's token and run code as them. Tools like GodPotato force SYSTEM to authenticate to you, which gives you a SYSTEM token, and then you use that privilege to run code as SYSTEM. The reason this is so common in real engagements is that IIS and MSSQL service accounts have this privilege by default."

---

**Question 4: What is BYOVD and why has it become so common in ransomware attacks?**

**Full answer:** BYOVD stands for Bring Your Own Vulnerable Driver. It is a technique where an attacker loads a legitimate, Microsoft-signed driver onto a target machine and then exploits a vulnerability in that driver to gain kernel-level code execution.

The reason it has become so common in ransomware attacks is the specific problem it solves. Modern EDR tools — CrowdStrike, SentinelOne, Microsoft Defender for Endpoint — are difficult to kill from user mode because they are protected processes. You cannot simply terminate them with `taskkill`. But from kernel mode, there is nothing stopping you. A driver running in kernel mode can terminate any process, including protected EDR processes.

This is exactly what groups like Qilin, BlackByte, and Kasseika have done in real ransomware attacks. They load a vulnerable driver, exploit it to get kernel access, use that access to kill the EDR, and then deploy the ransomware with no security tools running to stop it. The LOLDrivers database is a publicly available list of drivers with known exploitable vulnerabilities, which has made it easier for attackers to find suitable drivers.

**How to say this confidently in an interview:** Emphasize why it is relevant right now. Say "BYOVD is the technique behind most major ransomware attacks in 2024 and 2025 because it solves the fundamental problem of modern EDR — you cannot kill it from user mode, but you can kill anything from kernel mode. The Qilin and BlackByte ransomware groups have both used this on real attacks."

---

**Question 5: What is DLL hijacking and what makes it different from a normal exploit?**

**Full answer:** DLL hijacking takes advantage of the way Windows searches for DLLs when an application starts up. When a Windows program needs to load a DLL, it looks in several locations in a specific order — the application's own folder, then Windows system folders, then directories in the system PATH. If an attacker can place a malicious DLL with the right name in a folder that Windows searches before the folder containing the real DLL, Windows will load the malicious DLL instead of the real one.

What makes this different from a normal exploit is that it does not require a bug in the application's code. The application is behaving exactly as designed — it is just asking Windows to find a DLL by name, and an attacker abused the search path. This means there is no CVE number, no patch to apply, and no vulnerability scanner will flag it. The fix is a configuration change (fixing the folder permissions or the search path), not a security update.

This also means the technique is persistent — as long as the attacker's DLL stays in that folder, it will be loaded every time the application runs. If the application runs as SYSTEM, every time it loads the DLL, the attacker's code runs as SYSTEM.

**How to say this confidently in an interview:** Contrast it with traditional exploits. "Most exploits target a bug in code. DLL hijacking targets a behavior that is working exactly as intended — it just abuses the fact that Windows searches for DLLs in a specific order. There is no CVE, no patch, no scanner will find it. You find it with Process Monitor."

---

### Scenario-Based Questions

**Question 6: You land a shell on a Windows server as the IIS service account. What is the very first thing you check and why?**

**Full answer:** The first thing to check is the current user's token privileges using `whoami /priv`. Before doing anything else, before running any tools or checking any other information, you want to know what privileges the IIS service account has.

The reason is that IIS application pool accounts almost always have SeImpersonatePrivilege. If that privilege is listed as "Enabled" in the output, you immediately know that escalating to SYSTEM is a straightforward one-step process using GodPotato or SigmaPotato. You do not need to spend time doing a full enumeration of the machine. The escalation path is right in front of you.

If SeImpersonatePrivilege is there, you run SigmaPotato to get a SYSTEM shell and only then do you start the rest of your post-exploitation work — credential dumping, lateral movement, and working toward the engagement objective.

**How to say this confidently in an interview:** Show methodical thinking. Say "The very first command I run after getting a shell is `whoami /priv`. Not WinPEAS, not a service check — token privileges. Because IIS accounts almost always have SeImpersonatePrivilege, and if that is present, I can get SYSTEM in one command and then do everything else from a SYSTEM context."

---

**Question 7: You are on a fully patched Windows 11 machine with CrowdStrike Falcon deployed. You are a local admin but running at medium integrity. How do you get SYSTEM without triggering an alert?**

**Full answer:** This is a two-step problem. First, you need to get to high integrity (bypass UAC) without triggering an alert. Second, you need to escalate from high-integrity admin to SYSTEM.

For the UAC bypass, the fodhelper technique works on modern Windows and does not trigger CrowdStrike in most configurations because it uses a trusted Windows binary (`fodhelper.exe`) and a registry key write in the current user hive — both of which are normal operations on their own. You write a command to `HKCU\Software\Classes\ms-settings\Shell\Open\command` and then run `fodhelper.exe`. It executes your command at high integrity.

Once at high integrity, to get SYSTEM you can either use a kernel driver if BYOVD is in scope, or look for a scheduled task or service running as SYSTEM with a binary or script you can replace. If this is a real machine in an organization, there is almost always a third-party agent, backup tool, or monitoring software running as SYSTEM with some misconfiguration. You do not need an exploit. You need a misconfiguration.

For the CrowdStrike concern specifically: manual techniques using built-in Windows functionality (registry writes, WMI, scheduled task creation) are less likely to trigger behavioral detections than running known offensive tools. The combination of a UAC bypass using system binaries plus a manual service or task replacement is much quieter than running WinPEAS or Mimikatz.

**How to say this confidently in an interview:** Walk through it step by step without rushing. Show that you know the two separate problems (UAC and SYSTEM escalation) and have a concrete answer for each. Mention CrowdStrike specifically to show you understand EDR-aware operations.

---

**Question 8: During an engagement, WinPEAS finds no privilege escalation paths. What do you do next?**

**Full answer:** WinPEAS missing something does not mean there is nothing there. It means the automated checks did not find anything obvious. The next step is manual enumeration.

Start with the things WinPEAS is weakest at. Check every running service manually with `sc qc` and look at the binary paths — not just for unquoted paths but for whether the binaries are in writable locations. Check the ACLs on service binaries using `icacls`. Look at scheduled tasks in detail using `schtasks /query /fo LIST /v` and check whether the executables they call are in locations you can write to. Look at what DLLs high-privilege processes are trying to load using Process Monitor, and check whether any of those DLL paths are writable.

Also look at things that are not strictly "privilege escalation" but can become it with a bit of creativity: stored credentials, readable backup files, old configuration files, PowerShell history, browser-saved passwords. Lateral movement to a higher-privilege machine is also a valid path — you do not always need to escalate on the current machine.

**How to say this confidently in an interview:** Show you are not dependent on tools. Say "WinPEAS finding nothing is where the real work starts. I go manual — checking service ACLs with icacls, looking at task binaries, using Procmon to watch DLL loads from privileged processes. Tools automate the common checks. Manual work finds the uncommon ones."

---

**Question 9: A client asks you why they should care about an unquoted service path you found. How do you explain the risk in business terms?**

**Full answer:** The risk is that any user on the machine can become SYSTEM — the most powerful account on the computer — without needing a password or special access.

In practical terms: if an attacker gets onto this machine through any means — a phishing email, a weak password on a shared folder, a compromised employee laptop — and they have any kind of shell or command execution as a regular user, they can use this vulnerability to immediately take full control of the machine. Full control means they can install software, read every file, dump all stored passwords, and use the machine as a launching point to attack the rest of the network.

The vulnerability requires only write access to a specific folder, which on many systems is available to all users. The attacker does not need any special tools — a single executable file dropped in the right location is all it takes. It will work on any Windows version, patched or unpatched, because this is a configuration problem, not a code bug. Fixing it requires changing the service path in the registry to add quotes, which takes about two minutes.

**How to say this confidently in an interview:** Always translate technical findings into business impact. Interviewers at consulting firms specifically test for this. "The risk is complete machine compromise for any user who gets a foothold. The fix is a two-minute registry change. The cost of not fixing it is potentially losing the whole network."

---

**Question 10: You need to dump credentials from LSASS on a machine with Microsoft Defender running. How do you approach this?**

**Full answer:** Directly calling `sekurlsa::logonpasswords` in Mimikatz will get caught immediately. The approach needs to account for AMSI (which scans PowerShell and .NET code) and Defender's file and behavior signatures.

There are a few approaches depending on what is available:

First, if you have SYSTEM privileges and can patch AMSI and ETW first, you can run an in-memory version of Mimikatz through a reflective loader — the binary never touches disk and the scan never fires because AMSI has been disabled in that process.

Second, if process injection is available, you can inject into a process that Defender trusts, and do the LSASS dump from within that trusted process context.

Third, if those options are too risky in terms of detection, use `comsvcs.dll` for a legitimate MiniDump of the LSASS process — this is a LOLBin (Living Off the Land Binary) approach using a Windows component that ships with every Windows install. The command is `rundll32.exe C:\windows\system32\comsvcs.dll MiniDump <LSASS PID> C:\Temp\lsass.dmp full`. Defender catches this less reliably than Mimikatz because it is using a signed Microsoft DLL. Then move the dump off the machine and parse it with Mimikatz locally on your attack machine.

**How to say this confidently in an interview:** Show that you know multiple paths and can reason about which one fits the situation. "The direct Mimikatz approach is too noisy. I'd start with a AMSI+ETW bypass in memory, then try reflective loading. If that's still too risky, the comsvcs.dll MiniDump approach is much quieter — it's a Microsoft DLL doing the dump, not a known offensive tool."
