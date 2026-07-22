> ⚠️ this writeup covers retired/free htb content only, per htb's terms of service (academy tier 0 / free module content)

## table of contents
- [1. the 80/20 digest](#1-the-8020-digest)
- [2. structured breakdown (cheat sheet)](#2-structured-breakdown-cheat-sheet)
- [3. feynman checks](#3-feynman-checks)
- [4. omitted chuff](#4-omitted-chuff)

## 1. the 80/20 digest

### what infosec actually is (section 1)
infosec = protecting information and systems from people who shouldn't have access to them. the core "map" of the digital world: client → internet → servers → network → cloud, with blue/red/purple teams sitting around that infrastructure defending, attacking, and collaborating respectively.
*Q: does purple team replace blue and red? A: no, it's a combo of both working together.*

### risk, threat, vulnerability: the triangle
this is the single most important concept in the whole section and it's a chain, not three separate things:
- **vulnerability** = the weak lock on the door (a bug, misconfig, weak password)
- **threat** = the person/event that could exploit that weak lock (hacker, fire, flood)
- **risk** = the actual potential for damage if a threat meets a vulnerability

why this chain matters: a vulnerability with no threat capable of exploiting it isn't dangerous yet. a threat with no vulnerability to exploit can't do anything either. risk is what you get when you multiply the two together (likelihood × impact).
*Q: if a server has a critical bug but is airgapped with zero threat actors able to reach it, is there risk? A: minimal, risk needs both a vulnerability AND a credible threat.*

### roles (section 1)
the module maps 6 infosec roles (CISO, security architect, pentester, incident responder, security analyst, compliance specialist) and explicitly ties each back to how it interacts with pentesting, since that's htb's implied target role for the reader. pentester = actively finds/exploits vulns legally. incident responder = cleans up after/responds to attacks (including simulated ones from pentesters).
*Q: who designs the systems a pentester later tries to break? A: the security architect.*

### the CIA triad + 3 extra pillars (section 2)
core three:
- **confidentiality**: only authorized people see the data (tool: encryption, access control)
- **integrity**: data isn't tampered with (tool: hashing, digital signatures)
- **availability**: data/systems are up when needed (tool: redundancy, disaster recovery)

these three trade off against each other constantly in real system design, that's why it's called a "triad" and not just a checklist.

plus 3 more principles the module bundles in:
- **non-repudiation**: you can't deny you sent/signed something (digital signatures, audit logs)
- **authentication**: proving identity (passwords, biometrics, MFA)
- **privacy**: proper handling of personal data specifically (data minimization, consent)

mental model for why these are separate from CIA: authentication answers "who are you", confidentiality answers "can you see this", integrity answers "was this changed", availability answers "can you reach this", non-repudiation answers "can you prove who did this after the fact". they're answering different questions, which is why you need all of them, not just one.
*Q: does encrypting a file protect its integrity too? A: not by itself, encryption = confidentiality, hashing/signatures = integrity.*

### the 7 infosec processes (section 2)
this is basically the security lifecycle, in order:
risk assessment → security planning → implementing controls → monitoring/detection → incident response → disaster recovery → continuous improvement

it's a loop, not a line: continuous improvement feeds back into risk assessment. think of it like a patch cycle that never actually finishes.
*Q: what comes right after you detect an incident via monitoring? A: incident response (contain/mitigate).*

### tools overview (section 2)
grouped into defensive/general (firewalls, IDS/IPS, SIEM, vuln scanners, encryption tools, access control, awareness training) vs the pentest-specific toolkit called out by name: Nmap (scanning), Wireshark (packet analysis), Metasploit (exploitation), Burp Suite (web app testing), John the Ripper (password cracking). these 5 are the ones you'll actually touch as a beginner pentester.
*Q: which tool would you reach for to find open ports on a target? A: Nmap.*

## 2. structured breakdown (cheat sheet)

| concept | explanation | practical usage / problem solved |
| :--- | :--- | :--- |
| vulnerability | weakness in a system (bug, misconfig, weak pw) | what a pentester actively hunts for |
| threat | entity/event capable of exploiting a vuln | scoping "who are we defending against" |
| risk | likelihood × impact if threat meets vuln | prioritizing what to fix first |
| confidentiality | only authorized access | encryption, access controls |
| integrity | data isn't tampered with | hashing, digital signatures |
| availability | system is up when needed | redundancy, disaster recovery |
| non-repudiation | can't deny you did it | digital signatures, audit logs |
| authentication | proving identity | passwords, biometrics, MFA |
| privacy | proper handling of personal data | data minimization, consent mgmt |
| red team | simulates real attacks | offensive testing |
| blue team | defends the org | monitoring, incident response |
| purple team | red + blue combined | closing the feedback loop between attack and defense |
| security process loop | risk assess → plan → implement → monitor → respond → recover → improve | the lifecycle every org cycles through continuously |
| Nmap | network scanning/discovery | finding live hosts and open ports |
| Wireshark | packet/protocol analysis | inspecting traffic on the wire |
| Metasploit | exploitation framework | weaponizing known vulns |
| Burp Suite | web app security testing | intercepting/manipulating http traffic |
| John the Ripper | password cracking | testing password strength offline |

## 3. feynman checks

**Q1: a company has a critical unpatched vulnerability on a server, but that server sits on an isolated network with no external access and no internal user has malicious intent. is the risk high or low, and why?**
A: low. risk needs both a vulnerability and an active/credible threat capable of reaching it. no threat vector = minimal risk even with a severe vulnerability present.

**Q2: you're designing a system where users need to prove a financial transaction really came from them and can't later deny sending it. which CIA-triad-adjacent principle covers this, and what's a real mechanism for it?**
A: non-repudiation, covered via digital signatures and audit logs, not confidentiality or integrity, since the goal isn't hiding or protecting the data, it's proving authorship after the fact.

**Q3: why does the security process loop end in "continuous improvement" instead of just stopping after disaster recovery?**
A: because threats and tech evolve constantly, so the loop feeds improvement findings back into a fresh round of risk assessment, it's cyclical, not a one time checklist.

## 4. omitted chuff

- the "castle and treasure" analogy (walls = firewalls, guards = access control, knights = pentesters) was skipped since it's illustrative fluff restating concepts already covered directly
- the fighter jet analogy in the author's note (why this module is theory only, no labs yet) is just framing, not testable content
- the list of "areas of infosec" (network sec, app sec, cloud sec, IoT sec, etc) was condensed since it's just a category list with no mechanics behind it yet
- the "purpose of infosec" bullet list (protect data, ensure continuity, compliance, reputation, IP, digital transformation) is largely restating confidentiality/integrity/availability from a business angle rather than new technical content
