/\*\*  
 \* RPKIOriginGuard Pro \- Web Application Orchestrator  
 \*/

(function () {  
  'use strict';

  // Embedded Presets for 100% Offline / Client-Side Execution  
  const PRESETS \= {  
    apex: {  
      name: "Apex FinBank Core BGP & Hardened RPKI ROV (Tier-1 Zero-Trust)",  
      organization: "Apex Financial Services Corp",  
      asn: 64512,  
      rir: "ARIN",  
      announcedPrefixes: \[  
        {  
          prefix: "198.51.100.0/22",  
          originAsn: 64512,  
          addressFamily: "IPv4"  
        },  
        {  
          prefix: "2001:db8:a000::/36",  
          originAsn: 64512,  
          addressFamily: "IPv6"  
        }  
      \],  
      roas: \[  
        {  
          prefix: "198.51.100.0/22",  
          maxLength: 22,  
          asn: 64512,  
          rir: "ARIN",  
          notBefore: "2026-01-01T00:00:00Z",  
          notAfter: "2027-01-01T00:00:00Z"  
        },  
        {  
          prefix: "2001:db8:a000::/36",  
          maxLength: 36,  
          asn: 64512,  
          rir: "ARIN",  
          notBefore: "2026-01-01T00:00:00Z",  
          notAfter: "2027-01-01T00:00:00Z"  
        }  
      \],  
      routerConfig: {  
        platform: "BIRD2",  
        rtrValidators: \[  
          "10.0.1.50:3323",  
          "10.0.2.50:3323"  
        \],  
        dropInvalids: true,  
        applyToEBGP: true,  
        applyToIBGP: true,  
        bogonFiltering: true  
      },  
      monitoring: {  
        realtimeBgpTelemetry: true,  
        subprefixHijackAlerting: true,  
        roaExpiryAlerting: true  
      }  
    },  
    cloudpulse: {  
      name: "CloudPulse SaaS Edge BGP (Multi-Homed Transit with RPKI)",  
      organization: "CloudPulse Global Infrastructure",  
      asn: 65001,  
      rir: "RIPE NCC",  
      announcedPrefixes: \[  
        {  
          prefix: "203.0.113.0/24",  
          originAsn: 65001,  
          addressFamily: "IPv4"  
        }  
      \],  
      roas: \[  
        {  
          prefix: "203.0.113.0/24",  
          maxLength: 24,  
          asn: 65001,  
          rir: "RIPE NCC",  
          notBefore: "2026-06-01T00:00:00Z",  
          notAfter: "2027-06-01T00:00:00Z"  
        }  
      \],  
      routerConfig: {  
        platform: "FRR",  
        rtrValidators: \[  
          "192.168.10.20:8282"  
        \],  
        dropInvalids: true,  
        applyToEBGP: true,  
        applyToIBGP: false,  
        bogonFiltering: true  
      },  
      monitoring: {  
        realtimeBgpTelemetry: true,  
        subprefixHijackAlerting: false,  
        roaExpiryAlerting: true  
      }  
    },  
    legacy: {  
      name: "Legacy Corp Network (Loose MaxLength Anti-Pattern & Blackhole Outage)",  
      organization: "Legacy Corp Communications",  
      asn: 65500,  
      rir: "ARIN",  
      announcedPrefixes: \[  
        {  
          prefix: "198.51.100.0/24",  
          originAsn: 65500,  
          addressFamily: "IPv4"  
        },  
        {  
          prefix: "192.0.2.0/20",  
          originAsn: 65500,  
          addressFamily: "IPv4"  
        }  
      \],  
      roas: \[  
        {  
          prefix: "198.51.100.0/23",  
          maxLength: 23,  
          asn: 65500,  
          rir: "ARIN",  
          notBefore: "2025-01-01T00:00:00Z",  
          notAfter: "2027-01-01T00:00:00Z"  
        },  
        {  
          prefix: "192.0.2.0/20",  
          maxLength: 24,  
          asn: 65500,  
          rir: "ARIN",  
          notBefore: "2025-01-01T00:00:00Z",  
          notAfter: "2027-01-01T00:00:00Z"  
        }  
      \],  
      routerConfig: {  
        platform: "Cisco\_IOS\_XR",  
        rtrValidators: \[\],  
        dropInvalids: false,  
        applyToEBGP: false,  
        applyToIBGP: false,  
        bogonFiltering: false  
      },  
      monitoring: {  
        realtimeBgpTelemetry: false,  
        subprefixHijackAlerting: false,  
        roaExpiryAlerting: false  
      }  
    }  
  };

  let currentParsedConfig \= null;  
  let activeSynthTarget \= 'bird';  
  let activeSimScenario \= 'hijack';

  // DOM Elements  
  const presetSelector \= document.getElementById('presetSelector');  
  const configInput \= document.getElementById('configInput');  
  const btnAudit \= document.getElementById('btnAudit');  
  const btnReset \= document.getElementById('btnReset');  
  const inspAsn \= document.getElementById('inspAsn');  
  const inspRov \= document.getElementById('inspRov');  
  const inspMaxLength \= document.getElementById('inspMaxLength');  
  const inspRouter \= document.getElementById('inspRouter');  
  const gradeBadge \= document.getElementById('gradeBadge');  
  const scoreText \= document.getElementById('scoreText');  
  const gradeSummary \= document.getElementById('gradeSummary');  
  const scoreSummaryTag \= document.getElementById('scoreSummaryTag');  
  const checksContainer \= document.getElementById('checksContainer');  
  const btnSimHijack \= document.getElementById('btnSimHijack');  
  const btnSimBlackhole \= document.getElementById('btnSimBlackhole');  
  const simUnhardenedSummary \= document.getElementById('simUnhardenedSummary');  
  const simHardenedSummary \= document.getElementById('simHardenedSummary');  
  const simUnhardenedTimeline \= document.getElementById('simUnhardenedTimeline');  
  const simHardenedTimeline \= document.getElementById('simHardenedTimeline');  
  const codeOutput \= document.getElementById('codeOutput');  
  const btnCopyCode \= document.getElementById('btnCopyCode');

  function loadPreset(key) {  
    const preset \= PRESETS\[key\] || PRESETS.apex;  
    configInput.value \= JSON.stringify(preset, null, 2);  
    runAnalysis();  
  }

  function runAnalysis() {  
    let parsed;  
    try {  
      parsed \= JSON.parse(configInput.value);  
      currentParsedConfig \= parsed;  
    } catch (e) {  
      alert('JSON Syntax Error: ' \+ e.message);  
      return;  
    }

    const result \= window.RPKIAuditor.auditRPKIConfig(parsed);  
    const inspected \= result.inspected;

    // Update Inspector Pills  
    inspAsn.textContent \= \`AS\${inspected.asn} (\${inspected.organization})\`;  
    inspRov.textContent \= \`\${inspected.metrics.validCount} / \${inspected.metrics.totalAnnounced} Valid (\${inspected.metrics.invalidCount} Invalid)\`;  
    inspRov.style.color \= inspected.metrics.invalidCount \=== 0 ? 'var(--accent-emerald)' : 'var(--accent-rose)';

    inspMaxLength.textContent \= inspected.metrics.looseMaxLengthCount \=== 0  
      ? 'Strict (RFC 9319 Aligned)'  
      : \`Loose (\${inspected.metrics.looseMaxLengthCount} Anti-Patterns)\`;  
    inspMaxLength.style.color \= inspected.metrics.looseMaxLengthCount \=== 0 ? 'var(--accent-emerald)' : 'var(--accent-amber)';

    inspRouter.textContent \= \`\${inspected.router.dropInvalids ? 'Drop Invalids' : 'Accepts Invalids'} (\${inspected.metrics.rtrCacheCount} RTR)\`;  
    inspRouter.style.color \= inspected.router.dropInvalids ? 'var(--accent-cyan)' : 'var(--accent-rose)';

    // Update Scorecard  
    scoreText.textContent \= \`\${result.overallScore} / 100 Points\`;  
    gradeBadge.textContent \= result.grade;  
    gradeBadge.className \= 'grade-circle';  
    if (result.grade \=== 'F' || result.grade \=== 'D') {  
      gradeBadge.classList.add('grade-f');  
    } else if (result.grade \=== 'A+' || result.grade \=== 'A') {  
      gradeBadge.classList.add('grade-a');  
    }

    scoreSummaryTag.textContent \= \`\${result.summary.passed} Passed • \${result.summary.warnings} Warnings • \${result.summary.failures} Failures\`;  
    gradeSummary.textContent \= \`\${result.grade \=== 'A+' ? 'Tier-1 Transit Protected' : 'Audit Complete'} • \${result.summary.failures} Outage Risks Identified\`;

    // Render Checks  
    checksContainer.innerHTML \= '';  
    result.checks.forEach(function (c) {  
      const item \= document.createElement('div');  
      item.className \= 'check-item';

      const badgeClass \=  
        c.status \=== 'PASS'  
          ? 'badge-pass'  
          : c.status \=== 'WARN'  
          ? 'badge-warn'  
          : c.status \=== 'FAIL'  
          ? 'badge-fail'  
          : 'badge-info';

      item.innerHTML \= \`  
        \<div class="check-header"\>  
          \<span class="check-name"\>\${c.title}\</span\>  
          \<span class="badge \${badgeClass}"\>\${c.status} (\${c.score}/\${c.weight})\</span\>  
        \</div\>  
        \<div class="check-desc"\>\${c.details}\</div\>  
        \${c.remediation ? \`\<div class="check-remediation"\>Remediation: \${c.remediation}\</div\>\` : ''}  
      \`;  
      checksContainer.appendChild(item);  
    });

    renderSimulation();  
    renderSynthesizer();  
  }

  function renderSimulation() {  
    if (activeSimScenario \=== 'hijack') {  
      const unhardened \= window.RPKISimulator.simulateBgpCrossHijack({ hasRoa: false, dropInvalids: false });  
      const hardened \= window.RPKISimulator.simulateBgpCrossHijack({ hasRoa: true, dropInvalids: true });

      simUnhardenedSummary.textContent \= unhardened.verdict;  
      simHardenedSummary.textContent \= hardened.verdict;

      simUnhardenedTimeline.innerHTML \= unhardened.steps.map(s \=\> \`  
        \<div class="timeline-step \${s.isHijacked ? 'step-outage' : ''}"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: \${s.isHijacked ? 'var(--accent-rose)' : 'var(--accent-emerald)'};"\>  
            \${s.trafficRoute}  
          \</div\>  
        \</div\>  
      \`).join('');

      simHardenedTimeline.innerHTML \= hardened.steps.map(s \=\> \`  
        \<div class="timeline-step"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: var(--accent-emerald);"\>  
            \${s.trafficRoute}  
          \</div\>  
        \</div\>  
      \`).join('');  
    } else {  
      const unhardened \= window.RPKISimulator.simulateMaxLengthBlackhole({ roaMatchesAnnouncement: false });  
      const hardened \= window.RPKISimulator.simulateMaxLengthBlackhole({ roaMatchesAnnouncement: true });

      simUnhardenedSummary.textContent \= unhardened.verdict;  
      simHardenedSummary.textContent \= hardened.verdict;

      simUnhardenedTimeline.innerHTML \= unhardened.steps.map(s \=\> \`  
        \<div class="timeline-step \${s.isOutage ? 'step-outage' : ''}"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: \${s.isOutage ? 'var(--accent-rose)' : 'var(--accent-emerald)'};"\>  
            \${s.networkState}  
          \</div\>  
        \</div\>  
      \`).join('');

      simHardenedTimeline.innerHTML \= hardened.steps.map(s \=\> \`  
        \<div class="timeline-step"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: var(--accent-emerald);"\>  
            \${s.networkState}  
          \</div\>  
        \</div\>  
      \`).join('');  
    }  
  }

  function renderSynthesizer() {  
    if (\!currentParsedConfig) return;  
    const synth \= window.RouterSynthesizer;  
    let code \= '';  
    if (activeSynthTarget \=== 'bird') {  
      code \= synth.generateBird2Config(currentParsedConfig);  
    } else if (activeSynthTarget \=== 'frr') {  
      code \= synth.generateFrrConfig(currentParsedConfig);  
    } else if (activeSynthTarget \=== 'cisco') {  
      code \= synth.generateCiscoConfig(currentParsedConfig);  
    } else if (activeSynthTarget \=== 'junos') {  
      code \= synth.generateJunosConfig(currentParsedConfig);  
    }  
    codeOutput.textContent \= code;  
  }

  // Event Listeners  
  presetSelector.addEventListener('change', function () {  
    loadPreset(this.value);  
  });

  btnAudit.addEventListener('click', runAnalysis);

  btnReset.addEventListener('click', function () {  
    loadPreset(presetSelector.value);  
  });

  btnSimHijack.addEventListener('click', function () {  
    activeSimScenario \= 'hijack';  
    btnSimHijack.classList.add('active');  
    btnSimBlackhole.classList.remove('active');  
    renderSimulation();  
  });

  btnSimBlackhole.addEventListener('click', function () {  
    activeSimScenario \= 'blackhole';  
    btnSimBlackhole.classList.add('active');  
    btnSimHijack.classList.remove('active');  
    renderSimulation();  
  });

  document.querySelectorAll('.synth-tabs button').forEach(function (btn) {  
    btn.addEventListener('click', function () {  
      document.querySelectorAll('.synth-tabs button').forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeSynthTarget \= btn.getAttribute('data-target');  
      renderSynthesizer();  
    });  
  });

  btnCopyCode.addEventListener('click', function () {  
    navigator.clipboard.writeText(codeOutput.textContent).then(function () {  
      const originalText \= btnCopyCode.textContent;  
      btnCopyCode.textContent \= 'Copied\!';  
      setTimeout(function () {  
        btnCopyCode.textContent \= originalText;  
      }, 1800);  
    });  
  });

  // Initial Load  
  loadPreset('apex');  
})();  
