\# RPKIOriginGuard Pro: User Guide & NetDevOps CI/CD Integration

\#\# Overview  
RPKIOriginGuard Pro operates both as a 100% client-side web application and as a standalone Python CLI tool to audit BGP route announcements against RPKI ROAs, simulate failure modes, and synthesize production router configurations.

\---

\#\# 1\. Web Studio Quickstart  
1\. Open \`source-code/index.html\` in any modern web browser (no local web server or internet connection required).  
2\. Choose an architecture preset from the dropdown:  
   \- \*\*Apex FinBank\*\*: Tier-1 Zero-Trust ROA setup with dual RTR caches (Grade A+).  
   \- \*\*CloudPulse SaaS\*\*: Multi-homed transit edge with RFC 9319 strict MaxLength (Grade A).  
   \- \*\*Legacy Corp Network\*\*: Loose MaxLength anti-pattern and blackhole outage (Grade F).  
3\. Click \*\*Audit RPKI\*\* to view the live 10-point scorecard.  
4\. Review the \*\*Interactive BGP Hijack & MaxLength Blackhole Simulator\*\* to visualize sub-prefix hijacking prevention and de-aggregation outage risks.  
5\. Select a router platform (\*\*BIRD 2\*\*, \*\*FRRouting\*\*, \*\*Cisco IOS-XR\*\*, or \*\*Juniper Junos\*\*) and click \*\*Copy Configuration\*\*.

\---

\#\# 2\. Standalone Python CLI Tool

The CLI tool requires Python 3.8+ with zero external package dependencies.

\#\#\# Command Reference

\`\`\`bash  
\# Audit a BGP & RPKI configuration preset  
python3 build/rpkiguard-cli.py audit \-p presets/enterprise-bank-hardened-roa.json

\# Audit with strict CI/CD gating (exits with code 1 on failures or score \< 90\)  
python3 build/rpkiguard-cli.py audit \-p presets/enterprise-bank-hardened-roa.json \--strict

\# Inspect route validation details as raw JSON  
python3 build/rpkiguard-cli.py inspect \-p presets/enterprise-bank-hardened-roa.json  
\`\`\`

\---

\#\# 3\. NetDevOps CI/CD Pipeline Integration

Integrate RPKIOriginGuard Pro into Infrastructure-as-Code (Terraform, Ansible, NetBox) deployment pipelines to prevent deploying BGP routes that conflict with published ROAs:

\`\`\`yaml  
\# .github/workflows/rpki-guard.yml  
name: BGP & RPKI Pre-Deployment Verification  
on: \[push, pull\_request\]

jobs:  
  rpki-audit:  
    runs-on: ubuntu-latest  
    steps:  
      \- uses: actions/checkout@v4  
      \- name: Set up Python  
        uses: actions/setup-python@v5  
        with:  
          python-version: '3.11'  
      \- name: Run RPKIOriginGuard Strict Audit  
        run: |  
          python3 build/rpkiguard-cli.py audit \\  
            \-p presets/enterprise-bank-hardened-roa.json \\  
            \--strict  
\`\`\`  
