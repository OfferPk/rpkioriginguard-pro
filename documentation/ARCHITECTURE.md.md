\# RPKIOriginGuard Pro: Architectural Specification & Threat Modeling

\#\# Executive Summary  
The Border Gateway Protocol (BGP \- RFC 4271\) routes global internet traffic across thousands of Autonomous Systems (ASNs). Historically built without cryptographic authentication, BGP is susceptible to route hijacking, sub-prefix interception, and accidental route leaks. Resource Public Key Infrastructure (RPKI \- RFC 6480\) and Route Origin Validation (ROV \- RFC 6811 & RFC 8210\) address this by cryptographically binding IP prefixes to authorized origin ASNs via Route Origin Authorizations (ROAs).

However, improper ROA registration and router misconfigurations frequently cause severe self-inflicted blackhole outages and leave networks exposed to sub-prefix hijacking. \*\*RPKIOriginGuard Pro\*\* provides a unified auditing engine, BGP hijack simulator, and multi-router synthesizer to enforce Zero-Trust route origin validation and RFC 9319 compliance.

\---

\#\# Technical Fundamentals: RPKI & Route Origin Validation

\#\#\# 1\. Cryptographic Structure: Route Origin Authorizations (ROAs)  
Issued by Regional Internet Registries (ARIN, RIPE NCC, APNIC, LACNIC, AFRINIC), a ROA is a digitally signed CMS object asserting:  
\- \`Prefix\`: The IP address block (e.g., \`198.51.100.0/22\`).  
\- \`MaxLength\`: The maximum prefix length that the authorized ASN is allowed to announce (e.g., \`22\` or \`24\`).  
\- \`ASN\`: The authorized Autonomous System Number (e.g., \`AS64512\`).

\#\#\# 2\. Route Origin Validation (ROV) Outcomes (RFC 6811\)  
When an edge router receives a BGP route announcement \`(Route\_Prefix, Route\_AS)\`:  
1\. \*\*Valid\*\*: The route is covered by at least one ROA where \`Route\_AS \== ROA\_ASN\` AND \`Route\_Prefix\_Length \<= ROA\_MaxLength\`.  
2\. \*\*Invalid\*\*: The route is covered by at least one ROA, but conflicts with all covering ROAs (either because the announcing ASN does not match, or the prefix is more specific than \`MaxLength\`). Major Tier-1 transit providers (Lumen, Telia, NTT, Cloudflare, AT\&T) discard RPKI Invalids.  
3\. \*\*NotFound / Unknown\*\*: No ROA covers the prefix. The route is accepted by default, leaving it vulnerable to BGP hijacking.

\#\#\# 3\. The RFC 9319 Loose MaxLength Anti-Pattern  
Setting \`MaxLength\` longer than the announced prefix (e.g. creating a ROA for \`/20\` with \`MaxLength 24\`) is an anti-pattern. If an organization only announces \`/20\`, an attacker can forge a BGP announcement for any \`/24\` sub-prefix using a spoofed AS path; because the route matches the loose MaxLength, routers evaluate it as Valid. RFC 9319 recommends issuing minimal ROAs where \`MaxLength \== PrefixLength\`.

\---

\#\# Key Threat Models & Operational Failures

\#\#\# 1\. The Sub-Prefix BGP Hijack (Longest Prefix Match Exploit)  
\- \*\*Vulnerability\*\*: An enterprise announces \`198.51.100.0/22\`. A malicious actor announces \`198.51.100.0/24\`.  
\- \*\*Failure\*\*: Standard BGP routers prefer more-specific routes over less-specific ones. Without a strict ROA, the attacker's \`/24\` overrides the legitimate \`/22\`, redirecting 100% of sub-prefix traffic to the attacker.  
\- \*\*Remediation\*\*: Register strict ROAs and ensure transit providers drop RPKI Invalids.

\#\#\# 2\. The Self-Inflicted MaxLength Blackhole Outage  
\- \*\*Vulnerability\*\*: An engineering team de-aggregates an IP block for traffic engineering (e.g. announcing \`/24\`s instead of \`/22\`) without updating registered ROAs.  
\- \*\*Failure\*\*: Upstream transit routers evaluate the announcements as RPKI Invalid (prefix length exceeds MaxLength) and drop all incoming traffic, causing an immediate total service blackout.  
\- \*\*Remediation\*\*: Run pre-commit CI/CD validation to synchronize BGP configurations with RIR ROA databases.

\---

\#\# 10-Point Zero-Trust RPKI Scorecard Matrix

| ID | Check Title | Weight | Criteria |  
|---|---|---|---|  
| 1 | \*\*ROV Validity Rate\*\* | 15 pts | 100% of announced routes match valid ROAs. |  
| 2 | \*\*Zero RPKI Invalids Active\*\* | 15 pts | Zero routes blackholed by transit providers. |  
| 3 | \*\*Strict MaxLength Hygiene\*\* | 10 pts | RFC 9319 compliant (\`maxLength \== prefixLength\`). |  
| 4 | \*\*Edge Router Drop Invalids\*\* | 10 pts | Routers drop RPKI Invalid routes on eBGP transit. |  
| 5 | \*\*RTR Validator Redundancy\*\* | 10 pts | At least 2 redundant RTR validator cache servers. |  
| 6 | \*\*Complete Prefix Coverage\*\* | 10 pts | Zero \`NotFound\` unauthenticated IP prefixes. |  
| 7 | \*\*Bogon Prefix Filtering\*\* | 10 pts | Ingress filtering of private/unallocated IP space. |  
| 8 | \*\*Real-Time BGP Telemetry\*\* | 10 pts | Automated stream monitoring for rogue announcements. |  
| 9 | \*\*Dual-Stack Parity\*\* | 5 pts | Both IPv4 and IPv6 space protected with ROAs. |  
| 10| \*\*ROA Expiry Window Safety\*\* | 5 pts | ROAs have \> 60 days remaining before expiration. |

\---

\#\# Compliance Tiers  
\- \*\*Grade A+ (95–100%)\*\*: Zero-Trust Compliant, RFC 9319 Aligned, Tier-1 Transit Protected.  
\- \*\*Grade A (85–94%)\*\*: Enterprise Hardened BGP Standard.  
\- \*\*Grade B (75–84%)\*\*: Compliant with Minor Recommendations.  
\- \*\*Grade C (65–74%)\*\*: Elevated Hijacking or Outage Risk.  
\- \*\*Grade F (\< 60%)\*\*: Critical Outage Risk (Active Invalids, Loose MaxLength, or No Drop Policy).  
