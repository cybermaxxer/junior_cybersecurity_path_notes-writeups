Pre scriptum: this specific md also contains information from third resources that complements the HTB's learning material
## Table of Contents

* [1. The 80/20 Digest](#1-the-8020-digest)
   * [1.1 Foundations: TCP/UDP, protocols, ports](#11-foundations-tcpudp-protocols-ports)
   * [1.2 Addressing: MAC, IPv6, subnetting](#12-addressing-mac-ipv6-subnetting)
   * [1.3 Data flow: how a webpage request actually travels](#13-data-flow-how-a-webpage-request-actually-travels)
   * [1.4 IP packet anatomy + sniffing/traceroute](#14-ip-packet-anatomy--sniffingtraceroute)
   * [1.5 Wireless: WEP → WPA → WPA2 → WPA3 (the important one)](#15-wireless-wep--wpa--wpa2--wpa3-the-important-one)
   * [1.6 VLANs, trunking, VLAN hopping](#16-vlans-trunking-vlan-hopping)
   * [1.7 VPNs, IPsec, key exchange, cryptography basics](#17-vpns-ipsec-key-exchange-cryptography-basics)
   * [1.8 Cisco/vendor stuff: CDP, STP, VXLAN](#18-ciscovendor-stuff-cdp-stp-vxlan)
* [2. Structured Breakdown (Cheat Sheet)](#2-structured-breakdown-cheat-sheet)
* [3. Feynman Checks](#3-feynman-checks)
* [4. Omitted Chuff](#4-omitted-chuff-the-80-noise)

---

## 1. The 80/20 Digest

### 1.1 Foundations: TCP/UDP, protocols, ports

the whole module is basically: internet protocols = agreed on rules (RFCs) so any two devices can talk regardless of vendor. everything downstream (ports, addressing, VLANs) exists to answer "how does data get from A to B reliably and identifiably."

| criteria    | TCP                                                   | UDP                                                   |
| ----------- | ----------------------------------------------------- | ----------------------------------------------------- |
| connection  | connection oriented, does a three way handshake first | connectionless, just fires packets                    |
| reliability | guarantees delivery, retransmits lost packets         | no guarantee, packets can vanish                      |
| speed       | slower (handshake + checks overhead)                  | faster (no overhead)                                  |
| use case    | web, file transfer, anything needing accuracy         | streaming, VoIP, anything needing speed over accuracy |
_why the split exists_: losing one byte of a webpage breaks the html. losing one video frame is invisible but waiting for a handshake before every packet makes streaming laggy. the protocol design matches the actual cost of failure for that job.

Q: would a missing packet crash a live video call? A: no, it just drops a frame, that's exactly why it's UDP.

**ports you should know cold (tier 1):**

| acronym  | port   | usage                                                        |
| -------- | ------ | ------------------------------------------------------------ |
| SSH      | 22     | encrypted remote login/exec                                  |
| HTTP     | 80     | unencrypted web                                              |
| HTTPS    | 443    | encrypted web                                                |
| DNS      | 53     | domain → IP resolution                                       |
| FTP      | 20-21  | file transfer, common anonymous-login misconfig              |
| SMTP     | 25     | sending email                                                |
| SMB      | 445    | windows file sharing, huge AD attack surface                 |
| RDP      | 3389   | graphical remote access to windows                           |
| Kerberos | 88     | AD authentication (named after 3-headed dog, not an acronym) |
| LDAP     | 389    | AD directory queries                                         |
| DHCP     | 67, 68 | auto-assigns IPs                                             |

Q: nmap shows 88 and 389 open, what environment is this? A: windows Active Directory, that combo is a classic signature.

**tier 2/3 (recognize, situational):** Telnet(23), SNMP(161-162), TFTP(69), NTP(123), POP3(110), IMAP(143), NFS(111,2049), BOOTP(67,68), RADIUS(1812-1813), NNTP(119), RPC(135,137-139), ICMP(0-255), SIP(5060), ISAKMP(500), PPTP(1723), OSPF(89), oracle-tns(1521/1526), ingreslock(1524, historically abused as RCE backdoor), SOAP(80,443), Ident(113), IGMP(0-255).

**UDP-specific:** DNS(53), DHCP(67,68), NTP(123) — small quick request/response, doesn't need handshake overhead, cheaper to just re-ask than maintain a TCP connection. also: TFTP(69), SNMP(161), RIP(520), IKE(500), UPnP(1900, notoriously insecure), MySQL(3306), PostgreSQL(5432).

**ICMP — its own protocol, no ports at all:**

|type|what it does|
|---|---|
|echo request/reply|this is literally `ping`|
|destination unreachable|"can't deliver this packet"|
|time exceeded|TTL hit 0, packet dropped|
|redirect|router says "use this other router instead"|

**TTL, why it matters offensively:** every hop through a router decrements TTL by 1. hits 0 → "time exceeded" reply. that's literally how `traceroute` works (increasing TTL 1,2,3...).

**OS fingerprinting via TTL:**

|default TTL|likely OS|
|---|---|
|128|windows|
|64|linux/macOS|
|255|solaris|

TTL 122 in a response ≈ 128 minus 6 hops ≈ windows, 6 routers away. rough heuristic, not gospel (TTL is user-configurable).

Q: you ping a host, TTL comes back 63, why might it still be linux even though 64 is the default? A: each hop -1's the TTL, a linux box 1 hop away shows TTL 63 — factor in hop count before guessing OS.

**SIP/VoIP:** runs on TCP/UDP 5060-5061. `OPTIONS` method is the offensively interesting one — sending it to different usernames enumerates valid users on a SIP server (different response for valid vs invalid), sets up brute forcing.

---

### 1.2 Addressing: MAC, IPv6, subnetting

**MAC address — mental model: it's the "who are you physically" layer.**

- 48-bit (6 octets), hex, e.g. `DE:AD:BE:EF:13:37`
- first 3 bytes (24 bit) = OUI (Organizationally Unique Identifier), assigned by IEEE per manufacturer
- last 3 bytes = individual address part, set by manufacturer, meant to be globally unique
- IP delivery on the local network happens via MAC, not IP — layer 2 needs the physical address to actually hand off a frame

**the last bits of the first octet aren't arbitrary — they encode meaning:**

|bit|meaning|
|---|---|
|last bit of 1st octet = 0|unicast (goes to one host)|
|last bit of 1st octet = 1|multicast (goes to all hosts, they individually decide to accept)|
|2nd-to-last bit = 0|globally unique (IEEE-assigned OUI)|
|2nd-to-last bit = 1|locally administered (manually overridden)|

broadcast MAC = `FF:FF:FF:FF:FF:FF`, all bits set, used by ARP/DHCP when the receiver's address isn't known yet.

**ARP (Address Resolution Protocol)**: resolves layer-3 IP → layer-2 MAC. device broadcasts "who has X IP", owner replies with its MAC. this is how devices go from "I know your IP" to "I can actually send you an ethernet frame."

**ARP spoofing/cache poisoning** (pentest relevant): send falsified ARP replies claiming your MAC owns someone else's IP (usually the gateway). victim's ARP cache gets poisoned → traffic meant for the gateway routes through the attacker → MITM. tools: Ettercap, Cain & Abel.

Q: why does ARP spoofing work at all — what's the actual flaw being exploited? A: ARP has no authentication, any device can claim to own any IP, so a forged reply is accepted at face value.

**MAC attack vectors:**

|attack|what it does|
|---|---|
|MAC spoofing|change your MAC to impersonate another device, bypass MAC filtering|
|MAC flooding|flood a switch with fake MACs until its MAC table overflows, switch fails open and starts broadcasting everything (defeats the whole point of a switch)|
|MAC filtering bypass|if network only allows specific MACs, spoof an allowed one|

---

**IPv6 — mental model: same job as IPv4 (address a host), but designed for a world with way more devices, no NAT needed.**

|feature|IPv4|IPv6|
|---|---|---|
|bit length|32-bit|128-bit|
|address space|~4.3 billion|~340 undecillion|
|representation|decimal|hex|
|notation|`10.10.10.0/24`|`fe80::dd80:b1a9:6687:2d3b/64`|
|dynamic addressing|DHCP|SLAAC / DHCPv6|
|IPsec|optional|mandatory|
|broadcast|exists|eliminated — replaced by multicast|

**why hex specifically:** decimal shows 10 states per char, binary shows 2, hex shows 16 — most compact human-readable way to represent long binary strings. IPv6 = 128 bits = 8 blocks × 16 bits (4 hex digits each), separated by `:` instead of `.`.

**shortening rules (RFC 5952)** — this part is arbitrary convention, memorize it:

- lowercase letters always
- leading zeros in a block always dropped
- one run of consecutive all-zero blocks → `::` (only once, leftmost run if there's a tie)

`fe80:0000:0000:0000:dd80:b1a9:6687:2d3b/64` → `fe80::dd80:b1a9:6687:2d3b/64`

**structure:** Network Prefix (network part) + Interface Identifier/Suffix (host part). default prefix is `/64`. interface identifier is derived from the 48-bit MAC, expanded to 64 bits.

Q: why can IPv6 skip NAT entirely? A: address space is so large every device can get a real public address, no need to conserve via private+NAT translation.

---

**Subnetting — mental model: carving one IP range into smaller labeled sub-ranges using the subnet mask as a template.**

worked example: `192.168.12.160/26`, mask `255.255.255.192`

- 1-bits in the mask = fixed (network part), 0-bits = variable (host part)
- **network address** = set all host bits to 0
- **broadcast address** = set all host bits to 1
- everything between = usable host range

`/26` → 64 addresses total → minus network + broadcast = **62 usable hosts**

**the fast mental-math trick** (no memorization needed, just powers of 2):

1. find which octet the CIDR falls in (`/8`,`/16`,`/24`,`/32` are octet boundaries)
2. `CIDR mod 8` = remainder → tells you the block size via `256 / 2^remainder`

|remainder|block size|
|---|---|
|0|256|
|1|128|
|2|64|
|3|32|
|4|16|
|5|8|
|6|4|
|7|2|

e.g. `/25` → 25 mod 8 = 1 → block size 128. `/27` → 27 mod 8 = 3 → block size 32.

**dividing into N subnets:** subnet count must be a power of 2. extend the mask by however many bits `2^n = required subnets`. e.g. need 4 subnets → extend mask by 2 bits (`2^2=4`).

Q: if you have a `/24` and need 8 subnets, how many bits do you extend the mask by? A: 3 bits (`2^3=8`), giving `/27`.

---

### 1.3 Data flow: how a webpage request actually travels

full journey when you type a URL, step by step (this ties every other topic together):

1. **connect to WLAN** → SSID selected, WPA2/3 password auth, DHCP takes over IP config
2. **DHCP** → laptop gets private IP (e.g. `192.168.1.10`) + subnet mask, gateway, DNS server
3. **DNS resolution** → query sent to DNS server (ISP/Google DNS), response returns the real IP of the site
4. **encapsulation** (OSI/TCP-IP model, top-down):
    - application layer: browser builds HTTP/HTTPS request
    - transport layer: wrapped in TCP segment (ports: 80/443)
    - internet layer: wrapped in IP packet (source = private IP, dest = remote server IP)
    - link layer: wrapped in Ethernet/WiFi frame (source/dest MAC), gateway's MAC found via ARP if not cached
5. **NAT at the router**: private IP swapped for the router's public IP in the packet header, forwarded to ISP
6. **server response**: firewall checks the port, web server (Apache/Nginx/IIS) processes and replies, reverse path back through the internet
7. **NAT unwinds** at the home router (maps public IP back to the private IP)
8. **decapsulation**: laptop strips frame → IP header → TCP header, browser renders the HTML/CSS/JS

Q: why does the laptop need ARP before it can even send the first packet out to the internet? A: it needs the gateway's MAC address to build the link-layer frame; you can know the destination IP but you still need the _local_ physical hop's MAC to actually transmit anything.

---

### 1.4 IP packet anatomy + sniffing/traceroute

**IP packet = envelope model.** header = sender/recipient/routing info, payload = the actual data (like the letter inside).

**key header fields:**

|field|purpose|
|---|---|
|Version|IPv4 vs IPv6|
|Total Length|packet size in bytes|
|Identification (IP ID)|16-bit, 0-65535, tags fragments of the same original packet|
|Flags / Fragment Offset|fragmentation control|
|TTL|hop budget before packet is dropped|
|Protocol|what's inside (TCP/UDP/etc)|
|Checksum|header error detection|
|Source/Destination|routing addresses|

**IP ID as a fingerprinting trick:** if one host has multiple IPs, its outbound packets from different source IPs will still show a very close/sequential IP ID counter, since it's the _same OS TCP/IP stack_ generating them. Continuous IDs across two "different" IPs → strong signal they're the same physical host. (pentest relevant: host discovery/correlation)

**Record-Route field + `ping -R`**: forces intermediate routers to log their IP into the packet, showing every hop on the way there and back.

**how `traceroute` actually works (TCP method):**

1. send TCP SYN with TTL=1
2. first router decrements TTL to 0, drops packet, replies with ICMP Time-Exceeded
3. note that router's IP, resend with TTL+1
4. repeat until you get SYN/ACK or RST from the actual destination

Q: why does incrementing TTL by 1 each time reveal the whole path? A: each router along the route becomes the "last hop" once, drops the packet at that TTL value and identifies itself via the Time-Exceeded reply — you're essentially forcing each hop to "raise its hand" one at a time.

**UDP traceroute** (common on Unix hosts): gets Destination/Port Unreachable instead of SYN/ACK when it finally reaches the target.

**Blind spoofing**: attacker forges source IP/port and a fake Initial Sequence Number (ISN) in a TCP packet without seeing the real responses — trying to trick the target into establishing a connection state with a spoofed identity, blind because the attacker never sees the actual reply traffic.

---

### 1.5 Wireless: WEP → WPA → WPA2 → WPA3 (the important one)

**persona: network security engineer**

this is the deepest section of the whole module — full attack-relevant breakdowns of all four protocol generations. mental model to hold onto throughout: **each generation fixes exactly one structural flaw in the previous one, nothing more.**

#### WEP (fully broken)

**phase 1 — joining:**

|step|what happens|
|---|---|
|scan|device sees beacon frames, gets SSID|
|auth request|open system or shared key mode|
|open system|AP says "authenticated", zero crypto check|
|shared key|AP sends plaintext challenge, client encrypts w/ WEP key+RC4, AP verifies — **this leaks free keystream to any sniffer**|
|association + DHCP|standard|

**phase 2 — data encryption, every packet:**

1. 24-bit IV chosen per packet
2. seed = IV + static WEP key
3. CRC32 (ICV) computed over plaintext, appended
4. RC4(seed) generates keystream
5. ciphertext = (plaintext + ICV) XOR keystream
6. IV sent in cleartext alongside ciphertext

**why it collapses:** 24-bit IV space = only 16.7M values, cycles in hours on a busy AP. IV reuse → identical keystream → `ciphertext_A XOR ciphertext_B = plaintext_A XOR plaintext_B`. combined with predictable WiFi traffic patterns (ARP, DHCP), tools like aircrack-ng statistically recover the static key itself (FMS attack, KoreK/PTW improvements).

mental model: reusing a static key with only a tiny IV to disguise it is structurally guaranteed to fail given enough traffic — not "a clever trick was found once."

**mnemonic**: SIRKX → Seed, ICV, RC4 keystream, Key reuse, Xor collision.

Q: why does IV reuse leak info even without knowing the WEP key? A: identical seed → identical keystream → XOR of two ciphertexts cancels the keystream, leaving XOR of the two plaintexts, no key knowledge needed for that step.

**WEP attack tooling:**

|tool|job|
|---|---|
|airodump-ng|passive capture of IV/ciphertext pairs|
|aireplay-ng|inject/replay traffic (ARP replay) to force more IVs faster|
|aircrack-ng|statistical key recovery (FMS/KoreK/PTW)|

---

#### WPA / TKIP (patch, not a fix)

phase 1 (joining/4-way handshake) is **identical** to WPA2 — see below. the entire difference is phase 2: how the per-packet key gets generated.

| step                  | what happens                                                | why                                                |
| --------------------- | ----------------------------------------------------------- | -------------------------------------------------- |
| TK extracted from PTK | temporal key, feeds into TKIP                               |                                                    |
| TSC increments        | 48-bit TKIP sequence counter, one per packet                | way bigger than WEP's 24-bit IV — kills fast reuse |
| phase 1 key mixing    | TK + TSC high bytes + client MAC → intermediate key         | ties key to this device                            |
| phase 2 key mixing    | intermediate key + TSC low bytes → final per-packet RC4 key | guarantees uniqueness per packet                   |
| RC4 keystream         | same weak cipher as WEP, but genuinely unique seed now      | keeps old hardware compatible                      |
| Michael MIC           | real cryptographic integrity check, replaces fake CRC32     | actually detects tampering                         |

**mental model**: WEP's flaw was architectural (a bad lock). WPA doesn't replace the lock, it rotates the pins slightly every use — better, but still trusting a fundamentally old design underneath. still built on RC4, and Michael MIC was deliberately weakened for speed → still crackable via chopchop-derived attacks, just not total key recovery like WEP.

**mnemonic**: TTM → TSC, Two-phase mixing, Michael MIC.
#### the full wpa pipeline

| stage                                       | step                    | who                 | what happens                                                       | why                                                                                        |
| ------------------------------------------- | ----------------------- | ------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| **0. pre handshake**                        | psk entered             | alice               | password typed into device                                         | the one thing the human actually provides                                                  |
|                                             | pmk derived             | both, independently | `pmk = pbkdf2(psk, ssid, 4096, 256)`                               | slow hash, resists brute force, both sides compute the same result without transmitting it |
| **1. association**                          | open auth + association | alice ↔ ap          | normal wifi connect, no encryption yet                             | identical to wep at this stage                                                             |
| **2. 4 way handshake**                      | message 1               | ap → alice          | sends anonce (random, plaintext)                                   | fresh randomness so this session's keys are unique                                         |
|                                             | ptk derived             | alice               | `ptk = prf(pmk, anonce, snonce, mac_ap, mac_alice)`                | alice is first, she now has all 5 ingredients                                              |
|                                             | message 2               | alice → ap          | sends snonce + mic (built from kck)                                | proves she derived correct ptk without sending it                                          |
|                                             | ptk derived             | ap                  | same prf, now has both nonces                                      | verifies alice's mic matches its own                                                       |
|                                             | message 3               | ap → alice          | sends gtk (wrapped with kek) + its own mic                         | ap proves itself back, mutual proof, delivers broadcast key                                |
|                                             | message 4               | alice → ap          | ack                                                                | handshake complete, both sides now hold identical ptk + gtk                                |
| **3. ptk slicing**                          | kck extracted           | both                | key confirmation key                                               | signs/verifies eapol messages (the mics above)                                             |
|                                             | kek extracted           | both                | key encryption key                                                 | wraps/unwraps the gtk in message 3                                                         |
|                                             | tk extracted            | both                | temporal key                                                       | the actual key that encrypts data frames                                                   |
| **4. per packet encryption (tkip)**         | tsc increments          | both                | 48 bit counter, +1 every packet                                    | way bigger than wep's 24 bit iv, kills fast reuse                                          |
|                                             | phase 1 mixing          | both                | tk + tsc high bits + client mac → intermediate key                 | expensive, run once per 65536 packets, ties key to device                                  |
|                                             | phase 2 mixing          | both                | intermediate key + tsc low bits → per packet rc4 key               | cheap, run every packet, guarantees per packet uniqueness                                  |
|                                             | rc4 keystream           | both                | same old cipher, but genuinely unique seed now                     | keeps old hardware compatible                                                              |
|                                             | michael mic             | both                | real cryptographic integrity check                                 | replaces wep's forgeable crc32                                                             |
| **4alt. per packet encryption (ccmp/wpa2)** | aes ccm                 | both                | tk directly used with aes in ccm mode                              | encryption + integrity in one pass, no separate mic keys needed                            |
| **5. group traffic**                        | gtk usage               | both                | broadcast/multicast frames encrypted with shared gtk instead of tk | everyone needs to decrypt the same broadcast frame                                         |
|                                             | gtk rotation            | ap                  | regenerates + redistributes gtk when a device disconnects          | prevents departed devices from still decrypting broadcast traffic                          |

#### the acronym master list

| acronym | stands for                                  |
| ------- | ------------------------------------------- |
| wpa     | wi-fi protected access                      |
| psk     | pre-shared key                              |
| pmk     | pairwise master key                         |
| ptk     | pairwise transient key                      |
| tk      | temporal key                                |
| kck     | key confirmation key                        |
| kek     | key encryption key                          |
| gtk     | group temporal key                          |
| anonce  | authenticator nonce                         |
| snonce  | supplicant nonce                            |
| mic     | message integrity code                      |
| tsc     | tkip sequence counter                       |
| tkip    | temporal key integrity protocol             |
| ccmp    | counter mode cbc mac protocol               |
| eapol   | extensible authentication protocol over lan |
| prf     | pseudo random function                      |
| pbkdf2  | password based key derivation function 2    |

### the throughline, one sentence each layer

- **pmk**: proves you know the password, without sending it
- **ptk**: unique key per session, derived not transmitted
- **kck/kek**: split ptk so handshake proof and key wrapping don't share a key
- **tsc mixing**: unique key per packet, so rc4's weakness can't be exploited
- **gtk**: separate shared key, because broadcast traffic can't use a private per device key
---

#### WPA2 / CCMP-AES (real fix, but only in phase 2)

**phase 1**: byte-for-byte identical 4-way handshake to WPA (below). **phase 2 is the actual innovation:**

|step|what happens|why|
|---|---|---|
|TK from PTK|feeds AES instead of RC4||
|PN assigned|48-bit packet number|guarantees uniqueness|
|nonce built|PN + source MAC + priority field|makes every AES op unique|
|AAD built|parts of MAC header that need integrity but not secrecy|tamper protection without hiding routing info|
|AES-CCM runs, one pass|produces ciphertext + MIC together|encryption+integrity as one unified primitive — this is the real architectural shift|

**why this actually closes the door**: AES is a block cipher with no practical break, and integrity is baked into the _same_ cipher operation (AEAD = authenticated encryption with associated data) instead of bolted on separately like WEP/WPA. remaining attack surface shifts entirely to weak passphrases, not the crypto.

**the full 4-way handshake (shared by WPA/WPA2):**

|acronym|meaning|role|
|---|---|---|
|PSK|pre-shared key|the wifi password|
|SSID|service set identifier|network name, mixed into derivation|
|PBKDF2|password-based key derivation function 2|slow hash turning PSK+SSID into PMK|
|PMK|pairwise master key|root key, `= PBKDF2(PSK, SSID)`, identical on both sides, computed locally with zero network traffic|
|ANonce/SNonce|AP/station nonce ("number used once")|randoms mixed in so each session's keys differ|
|PTK|pairwise transient key|actual session key, `= f(PMK, ANonce, SNonce, MACs)`|
|KCK|key confirmation key|sub-key from PTK, signs handshake messages|
|MIC|message integrity code|proves message authenticity, computed w/ KCK|
|GTK|group temporal key|separate key for broadcast/multicast traffic|

|msg|direction|contents|
|---|---|---|
|1|AP → client|ANonce|
|2|client → AP|SNonce + MIC|
|3|AP → client|GTK + MIC|
|4|client → AP|ack|

Q: why mix MAC address into PTK derivation instead of just PMK+nonces? A: ties the key to this specific device pair — even a nonce coincidence between two different devices would still produce different PTKs.

**WPA2 attack tooling** (crypto itself is never attacked, target is the password):

| tool                    | job                                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------------------ |
| airodump-ng             | monitors for a client (re)connecting, captures 4-way handshake                                   |
| aireplay-ng (deauth)    | forges deauth frames — 802.11 mgmt frames are unauthenticated — forces reconnect on demand       |
| hashcat / aircrack-ng   | offline crack: tries password candidates through PBKDF2→PMK→PTK→MIC, checks against captured MIC |
| PMKID attack (hcxtools) | grabs PMKID directly from AP, skips needing a full handshake or connected client at all          |

---

#### WPA2 Enterprise (802.1X + RADIUS)

**core structural difference:**

| criteria          |personal|enterprise|
|---|---|---|
| credential        |one shared PSK for everyone|unique per-user creds/cert|
| who checks it     |the AP itself|central RADIUS server, AP just relays|
| revoking one user |must change password for everyone|disable that one account|

**the actors:**

|term|role|
|---|---|
|supplicant|client device requesting access|
|authenticator|the AP, relays only, doesn't verify|
|authentication server|RADIUS, does actual checking|

flow: 802.1X blocks the port until EAP auth completes → EAP method negotiated (PEAP/MSCHAPv2 for passwords, or EAP-TLS for certs) → RADIUS sends "access accept" + MSK (becomes PMK) → **same 4-way handshake as personal from here on**. phase 2 encryption is identical to personal too — enterprise only changes _who gets to prove they belong_.

**mental model**: AP is a bouncer who doesn't know anyone, just radios a name to the manager (RADIUS) inside who checks a list and radios back yes/no.

**attack surface: trust, not crypto.** `hostapd-wpe` rogue APs impersonate the real SSID/RADIUS; if the client doesn't validate the RADIUS server's cert, credentials get handed straight to the attacker. never touches AES-CCMP.

---

#### WPA3 (fixes the handshake, not the encryption)

**core problem it fixes**: WPA2's PMK is a pure local computation (`PBKDF2(PSK,SSID)`) — attacker captures one handshake, brute forces offline forever, unlimited speed. WPA3 replaces this with **SAE** (Simultaneous Authentication of Equals, aka the Dragonfly handshake) — every single password guess requires a live round-trip with the AP, so offline cracking is dead.

| criteria         |WPA2|WPA3|
|---|---|---|
| PMK source       |local PBKDF2 computation|live SAE exchange only|
| offline cracking |trivial, GPU-bound|not possible, AP-interaction-bound|
| cipher mode      |AES-CCMP|AES-GCMP (modernization, not a security fix — AES-CCM was never broken)|
| forward secrecy  |no — old PSK compromise decrypts old captures|yes — fresh secrets each session|

**important nuance**: unlike WEP→WPA→WPA2, this isn't "encryption was broken and got fixed." AES-CCM was never cracked. GCMP is purely a performance/standardization upgrade. the actual security jump lives entirely in phase 1 (the handshake).

**WPA3 attack surface (much narrower)**: Dragonblood side-channel timing attacks (implementation bug, since patched), downgrade attacks against WPA3-transition-mode networks falling back to WPA2, DoS via SAE-flood (CPU exhaustion, not a confidentiality break).

**generational summary — what's actually vulnerable shifted each time:**

|generation|real weak point|
|---|---|
|WEP|the crypto itself (key, structurally broken)|
|WPA/WPA2 personal|human-chosen passwords, not the crypto|
|WPA2 enterprise|trust in the EAP negotiation (fake RADIUS), not crypto|
|WPA3|narrow implementation bugs or fallback misconfig, SAE itself holds up|

**mnemonic**: KHTN → Key (WEP), Human password (WPA/WPA2), Trust in EAP (enterprise), Narrow edge cases (WPA3).

---

#### the older/original WEP CRC / Challenge-Response material (supporting detail)

WEP-40/64 uses 24-bit IV + 40-bit secret key; WEP-104 uses 24-bit IV + 80-bit secret key. The CRC-based ICV has a structural flaw: it's calculated over _plaintext_, meaning you can determine plaintext content even from encrypted data via the CRC relationship — this is on top of the IV-reuse collapse above.

**LEAP vs PEAP**: LEAP uses a shared key for both encryption+auth (weak, RC4), PEAP tunnels through TLS first (strong). PEAP > LEAP because PEAP encrypts the MSCHAPv2 hash exchange, LEAP doesn't.

**Wireless hardening checklist**: disable SSID broadcast, use WPA2/WPA3, MAC filtering (weak alone, spoofable), deploy EAP-TLS for cert-based auth.

**Disassociation attack**: forged disassociation frames kick a client off, often used as a precursor to MITM by forcing a reconnect.

---

### 1.6 VLANs, trunking, VLAN hopping

**mental model**: a VLAN turns one physical switch into multiple logical switches — each VLAN is its own broadcast domain, so broadcast traffic in one VLAN never crosses into another without a router. this is _why_ it's a security boundary, not just an organizational one.

- VLAN IDs: 1-4094 usable (0 and 4095 reserved). 1 is default VLAN. 1-1005 = normal-range (saved in `vlan.dat`), 1006-4094 = extended-range (not saved).
- benefits: better organization, increased security (broadcast isolation), simplified admin (no physical-location dependency), increased performance (less broadcast noise)

**802.1Q tagging**: inserts a VLAN tag into the Ethernet frame so switches know which VLAN a frame belongs to. trunk ports carry multiple VLANs' traffic tagged; access ports carry one VLAN untagged.

**VLAN hopping attack**: exploits Cisco's DTP (Dynamic Trunking Protocol), which auto-negotiates trunk links between switches. attacker configures a host to mimic a switch, spoofs DTP+802.1Q signaling on a port where DTP is enabled by default → switch establishes a trunk link with the attacker → attacker now sees traffic from every VLAN, not just their assigned one. tool: Yersinia.

**Double-tagging VLAN hopping** (only works if attacker is on the _native VLAN_ of the trunk):

1. attacker sends a double-tagged frame: outer tag = native VLAN, inner tag = target VLAN
2. first switch strips the outer tag (matches native VLAN, doesn't re-tag since native VLAN frames go untagged on trunk) — never inspects the inner tag
3. second switch sees only the inner tag now, forwards to the target VLAN

mental model: the outer tag is a decoy that gets consumed by the "who am I" check; the inner tag survives because nobody thought to check twice. tools: Scapy, Yersinia.

Q: why does double-tagging only work from the native VLAN specifically? A: the native VLAN's tag is the one that gets silently stripped without re-inspection when crossing the trunk — that's the exact blind spot being exploited.

**VLAN-capable NICs** (Linux): load `8021q` kernel module → `ip link add link eth0 name eth0.20 type vlan id 20` (or deprecated `vconfig`) → assign IP → bring interface up.

**Wireshark VLAN analysis**: filter `vlan` for tagged traffic, `vlan.id == 10` for specific VLAN, `tshark -r file.pcapng -T fields -e vlan.id | sort -n -u` to enumerate all VLAN IDs seen in a capture.

---

### 1.7 VPNs, IPsec, key exchange, cryptography basics

**VPN core idea**: encrypted tunnel between a remote device and a private network, so the remote machine can act as if it's locally connected. requires: VPN client, VPN server, encryption, mutual authentication.

**IPsec** — two component protocols:

|protocol|does what|
|---|---|
|AH (Authentication Header)|integrity + authenticity, **no encryption**|
|ESP (Encapsulating Security Payload)|encryption + optional authentication|

two modes:

|mode|scope|
|---|---|
|Transport|encrypts payload only, IP header stays visible — host-to-host|
|Tunnel|encrypts the entire packet including header — network-to-network VPN tunnels|

**firewall ports for IPsec VPN traffic:**

|protocol|port|
|---|---|
|IKE|UDP/500|
|ESP|IP protocol 50 (or UDP/4500 when NAT traversal needed)|
|AH|IP protocol 51|

**PPTP**: older VPN protocol, deprecated for security — MSCHAPv2 auth relies on outdated DES, crackable with specialized hardware. superseded by L2TP/IPsec, IKEv2, OpenVPN.

---

**Cryptography fundamentals — symmetric vs asymmetric:**

||symmetric|asymmetric|
|---|---|---|
|keys|one shared key, encrypts+decrypts|public key (encrypt) + private key (decrypt)|
|speed|fast, used for bulk data|slower, math-heavy|
|key distribution problem|yes — how do you share the secret key safely?|no — public key can be shared openly|
|examples|AES, DES, 3DES|RSA, PGP, ECC|

**DES/3DES/AES lineage:**

- DES: 64-bit key, but 8 bits are checksum → effective 56-bit key. weak by modern standards.
- 3DES: 3 rounds (encrypt→decrypt→encrypt) with 3 keys, more secure than DES but still capped by 56-bit-per-round math.
- AES: DES's successor, 128/192/256-bit keys, faster (processes multiple blocks at once), current standard. found in WLAN 802.11i, IPsec, SSH, VoIP, PGP, OpenSSL.

**cipher modes** (how a block cipher handles data longer than one block):

|mode|typical use|
|---|---|
|ECB|avoid — doesn't hide data patterns, statistically attackable|
|CBC|disk encryption, email, default AES mode (TrueCrypt, TLS, SSL)|
|CFB|real-time streaming encryption (PKCS, BitLocker file encryption)|
|OFB|real-time streams, better keystream generation than CFB (SSH, PKCS)|
|CTR|real-time streams (IPsec, BitLocker)|
|GCM|when confidentiality AND integrity both matter together (wireless, VPNs) — this is the AEAD family WPA2/3 use|

---

**Key Exchange Mechanisms — mental model: how do two parties agree on a shared secret over a channel someone else might be listening to?**

|method|mechanism|notable trait|
|---|---|---|
|Diffie-Hellman (DH)|both sides generate a shared secret via math, no prior shared info needed|vulnerable to MITM if unauthenticated; slower than ECDH at equivalent security|
|RSA|based on the difficulty of factoring large primes|widely used, heavier computationally than ECC at same security level|
|ECDH|DH using elliptic curve math instead of raw modular exponentiation|more efficient + provides forward secrecy|
|ECDSA|elliptic curve digital signatures|used to authenticate parties in a key exchange|

Q: why is plain Diffie-Hellman vulnerable to MITM even though the math itself is sound? A: DH alone has no built-in authentication — an attacker can intercept and run separate DH exchanges with each side, impersonating the other party to both, since nothing verifies _who_ you're actually exchanging keys with.

**IKE (Internet Key Exchange)**: combines DH + other crypto to negotiate VPN security parameters. two modes:

|mode|phases|tradeoff|
|---|---|---|
|Main mode|3 phases|more secure, slower, default|
|Aggressive mode|2 phases (all params exchanged in phase 1)|faster, but no identity protection — less secure|

**PSK in IKE**: optional shared secret used to authenticate the parties before/during key exchange. if compromised via MITM, the whole session's security collapses.

---

### 1.8 Cisco/vendor stuff: CDP, STP, VXLAN

**Cisco IOS basics**: the OS running on Cisco routers/switches. password types worth knowing:

|password type|purpose|
|---|---|
|User|login to the device itself|
|Enable|enter "enable" mode (privileged/advanced functions)|
|Secret|restrict access to specific services/remote mgmt tools|
|Enable Secret|extra-secure, stored encrypted, protects enable mode|

**CDP (Cisco Discovery Protocol)**: layer-2 protocol, Cisco devices broadcast info about themselves to directly connected Cisco neighbors (device ID, IP, port, OS version, hardware platform). useful for topology discovery — but also an information leak, often disabled for security.

**STP (Spanning Tree Protocol)**: prevents Layer 2 loops in networks with redundant switch paths, by logically blocking redundant links until needed. STP traffic includes root switch info, bridge/port IDs, timing parameters (max-age, hello-time, forward-delay).

**VXLAN (Virtual eXtensible LAN)**: solves the problem that classic 802.1Q VLAN tags are only 12 bits (max 4094 VLANs) — not enough for large data centers/cloud providers. VXLAN is a "Layer 2 overlay on a Layer 3 network," uses a 24-bit VNI (VXLAN Network Identifier) → supports ~16 million segments. each VXLAN segment keeps VMs isolated to that segment only.

---

## 2. Structured Breakdown (Cheat Sheet)

| Concept                   | Explanation                                                      | Practical Usage                                             |
| ------------------------- | ---------------------------------------------------------------- | ----------------------------------------------------------- |
| TCP vs UDP                | connection-oriented+reliable vs connectionless+fast              | protocol choice signals the app's tolerance for loss        |
| Port scanning result      | open TCP 88+389                                                  | strongly suggests Windows AD environment                    |
| TTL fingerprinting        | 128=windows, 64=linux/mac, 255=solaris (minus hop count)         | rough OS guess from a single ping                           |
| ARP spoofing              | forged ARP replies poison the cache, redirect traffic            | classic MITM setup on a LAN                                 |
| IPv6 `::` shorthand       | collapse one run of all-zero blocks                              | reading/writing IPv6 addresses fast                         |
| subnet mod trick          | `CIDR mod 8` → block size via `256/2^remainder`                  | mental subnetting without a calculator                      |
| WEP collapse              | 24-bit IV reuse → keystream recovery via statistics              | explains why WEP is "broken," not "weak"                    |
| WPA2 4-way handshake      | PMK(local) → PTK(session) via ANonce/SNonce/MIC                  | capture handshake, crack offline (weak PW = real vuln)      |
| WPA3 SAE                  | live per-guess AP interaction, kills offline cracking            | current best-practice for personal WiFi                     |
| VLAN hopping (double-tag) | outer tag stripped blindly at native VLAN, inner tag survives    | requires attacker on native VLAN of the trunk               |
| IPsec AH vs ESP           | AH=integrity only, ESP=encryption(+optional integrity)           | ESP is what actually hides your VPN traffic                 |
| DH vs RSA vs ECDH         | DH=shared secret math, RSA=factoring primes, ECDH=curve-based DH | ECDH generally preferred now (efficiency + forward secrecy) |
| CDP                       | Cisco-only topology discovery broadcast                          | info leak if left enabled unnecessarily                     |

---

## 3. Feynman Checks

1. **You capture a WPA2 handshake off the air but the password is genuinely strong (20+ random chars). Can you crack it? Why does WPA3 change this specific scenario?** → No, practically not — hashcat/aircrack still have to brute force PBKDF2→PMK→PTK→MIC offline, and a strong random password makes that computationally infeasible regardless of speed. WPA3 doesn't make strong passwords "more crackable" — it makes _any_ password (including weak ones) resistant, because SAE requires a live AP round-trip per guess, so the offline math shortcut WPA2 allows simply doesn't exist in WPA3.
    
2. **Why does double-tagging VLAN hopping fail if the attacker isn't on the trunk's native VLAN?** → The exploit relies on the outer 802.1Q tag matching the native VLAN, so the first switch treats it as "no tag needed" traffic and strips it without re-tagging or re-inspecting the frame. If the attacker's outer tag isn't the native VLAN, the switch just forwards it normally with its actual tag intact — the "blind spot" that lets the inner tag survive to the second switch simply isn't there.
    
3. **You're pentesting a network and see UDP 500 and IP protocol 50 in a packet capture. What's happening, and what's the actual key exchange doing underneath?** → That's IKE negotiating an IPsec VPN tunnel (UDP/500) followed by ESP traffic (protocol 50) carrying the actual encrypted payload. Underneath, IKE is using Diffie-Hellman (or its elliptic-curve variant) to establish a shared secret over that insecure channel without ever transmitting the secret itself, then deriving session keys from it to feed AES/ESP encryption.
    

---
