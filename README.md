\# RPKIOriginGuard Pro (v1.0.0)

\> \*\*RPKI Route Origin Validation (RFC 6811 / RFC 8210\) & BGP Hijack Defense Studio, RFC 9319 MaxLength Auditor & Multi-Router Zero-Trust Synthesizer.\*\*

\[\!\[Standards\](https\://img.shields.io/badge/Standards-RFC%206480%20RPKI%20%7C%20RFC%206811%20ROV%20%7C%20RFC%208210%20RTR-indigo)\](\#)  
\[\!\[Security Standard\](https\://img.shields.io/badge/Policy-RFC%209319%20Loose%20MaxLength%20Defense%20%7C%20MANRS-amber)\](\#)  
\[\!\[Tests\](https\://img.shields.io/badge/Tests-7%2F7%20Passed-green)\](\#)  
\[\!\[License\](https\://img.shields.io/badge/License-Commercial%20%2F%20Developer-emerald)\](\#)

\---

\#\# Overview

The Border Gateway Protocol (BGP \- RFC 4271\) forms the routing fabric of the global internet, exchanging routes between tens of thousands of Autonomous Systems (ASNs). Because BGP historically lacked cryptographic origin authentication, networks are constantly vulnerable to BGP route hijacks, sub-prefix interception, and route leaks.

\*\*Resource Public Key Infrastructure (RPKI)\*\* and \*\*Route Origin Validation (ROV)\*\* solve this by cryptographically signing Route Origin Authorizations (ROAs) in Regional Internet Registry (RIR) databases, binding an IP prefix to an authorized Origin AS and a \`MaxLength\`.

However, ROA misconfigurations frequently cause catastrophic self-inflicted outages and security vulnerabilities:

1\. \*\*The Sub-Prefix BGP Hijack\*\*: Under BGP's Longest Prefix Match (LPM) rule, routers prefer more-specific routes. If an enterprise announces \`198.51.100.0/22\` without strict ROAs, an attacker announcing \`198.51.100.0/24\` intercepts 100% of global traffic.  
2\. \*\*The Loose MaxLength Vulnerability (RFC 9319)\*\*: Creating a ROA with \`MaxLength\` broader than the announced prefix (e.g. \`/20\` with \`MaxLength 24\`) allows an attacker to hijack \`/24\` sub-prefixes using a forged AS path, as routers evaluate the rogue announcement as RPKI Valid.  
3\. \*\*The Self-Inflicted MaxLength Blackhole\*\*: Network engineers de-aggregate prefixes for local traffic engineering without updating the RIR ROA. Major Tier-1 transit providers (Lumen, Telia, NTT, Cloudflare, AT\&T) evaluate the announcements as \*\*RPKI Invalid: MaxLength Exceeded\*\* and drop them, causing a total service blackout.

\*\*RPKIOriginGuard Pro\*\* delivers a 100% client-side web application and standalone Python CLI tool to inspect BGP routes against ROAs, audit configurations against a 10-point Zero-Trust scorecard (0–100%, Grades A+ to F), simulate sub-prefix hijacks and de-aggregation blackholes, and synthesize production router configurations (\*\*BIRD 2\*\*, \*\*FRRouting\*\*, \*\*Cisco IOS-XR\*\*, and \*\*Juniper Junos\*\*).

\---

\#\# Key Features

1\. \*\*BGP Route & RPKI ROA Inspector\*\*:  
   \- Analyzes IPv4 and IPv6 announced prefixes against covering ROAs.  
   \- Evaluates exact RFC 6811 ROV states (\`Valid\`, \`Invalid: ASN Mismatch\`, \`Invalid: MaxLength Exceeded\`, \`NotFound\`).  
   \- Flags RFC 9319 Loose MaxLength Anti-Patterns.  
2\. \*\*10-Point Zero-Trust RPKI Scorecard (0–100%, Grades A+ to F)\*\*:  
   \- Evaluates: ROV validity rate, zero active invalids, strict MaxLength hygiene, edge router drop-invalids policy, RTR validator cache redundancy, complete prefix coverage, bogon/martian filtering, real-time BGP telemetry, dual-stack parity, and ROA expiration horizons.  
3\. \*\*Interactive BGP Hijack & Outage Simulator\*\*:  
   \- \*\*Scenario A (Sub-Prefix BGP Hijack)\*\*: Visualizes how an attacker injecting a \`/24\` against an unprotected \`/22\` diverts 100% of traffic vs how strict ROA \+ Drop-Invalid causes Tier-1 transits to discard the attack.  
   \- \*\*Scenario B (MaxLength Blackhole Outage)\*\*: Simulates the disastrous consequences of de-aggregating routes without updating RIR ROAs.  
4\. \*\*Multi-Router Zero-Trust Synthesizer\*\*:  
   \- Generates hardened edge router policies for \*\*BIRD 2\*\*, \*\*FRRouting (FRR)\*\*, \*\*Cisco IOS-XR\*\*, and \*\*Juniper Junos\*\*.  
5\. \*\*Standalone Python CLI Tool (\`rpkiguard-cli.py\`)\*\*:  
   \- Zero external dependencies. Audits configs, checks prefix/ASN pairs, and provides a \`--strict\` flag for NetDevOps CI/CD pre-commit pipelines.

\---

\#\# Directory Structure

\`\`\`  
rpkiguard-pro/  
├── source-code/  
│   ├── index.html                  \# Cyber dark UI web studio  
│   ├── rpkiguard.css               \# Responsive dark stylesheet  
│   ├── rpki-engine.js              \# Core RPKI & ROV analysis engine  
│   ├── rpki-auditor.js             \# 10-point Zero-Trust auditor  
│   ├── rpki-simulator.js           \# BGP hijack & blackhole simulator  
│   ├── router-synthesizer.js       \# BIRD 2, FRR, Cisco, Junos generator  
│   └── app.js                      \# UI state manager & event orchestrator  
├── build/  
│   ├── rpkiguard-pro-v1.0.0.zip    \# Distributable package  
│   └── rpkiguard-cli.py            \# Standalone Python CLI & CI/CD tool  
├── presets/  
│   ├── enterprise-bank-hardened-roa.json  
│   ├── cloud-saas-multihomed-bgp.json  
│   └── vulnerable-hijack-maxlength-leak.json  
├── documentation/  
│   ├── ARCHITECTURE.md             \# Threat modeling & technical specification  
│   ├── USER\_GUIDE.md               \# User manual & CI/CD integration guide  
│   └── MONETIZATION\_AND\_MARKETING\_PLAYBOOK.md  
├── screenshots/  
│   ├── rpkiguard-studio-preview.svg  
│   └── rpki-rov-architecture.svg  
└── tests/  
    └── test\_rpkiguard\_suite.py     \# Automated test suite (7/7 passed)  
\`\`\`

\---

\#\# Quickstart

\#\#\# Running the Web Studio  
Open \`source-code/index.html\` directly in any web browser.

\#\#\# Running the Automated Tests  
\`\`\`bash  
python3 tests/test\_rpkiguard\_suite.py  
\`\`\`

\#\#\# Running the Standalone CLI  
\`\`\`bash  
\# Audit the enterprise banking preset  
python3 build/rpkiguard-cli.py audit \-p presets/enterprise-bank-hardened-roa.json

\# Audit with strict NetDevOps CI/CD gating (exits 1 on failures or score \< 90\)  
python3 build/rpkiguard-cli.py audit \-p presets/enterprise-bank-hardened-roa.json \--strict  
\`\`\`  
