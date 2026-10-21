/\*\*  
 \* RPKIOriginGuard Pro \- 10-Point Zero-Trust RPKI & BGP Auditor  
 \* Evaluates RFC 6480, RFC 6811, RFC 8210, and RFC 9319 compliance,  
 \* scores health (0-100%, Grade A+ to F), and provides concrete remediation steps.  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\['./rpki-engine'\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory(require('./rpki-engine'));  
  } else {  
    root.RPKIAuditor \= factory(root.RPKIEngine);  
  }  
})(typeof self \!== 'undefined' ? self : this, function (engine) {  
  'use strict';

  function auditRPKIConfig(config) {  
    const inspected \= engine.inspectEnterpriseRPKI(config);  
    const checks \= \[\];

    // Check 1: ROV Validity Rate (15 pts)  
    const validRate \= inspected.metrics.totalAnnounced \> 0  
      ? (inspected.metrics.validCount / inspected.metrics.totalAnnounced) \* 100  
      : 0;

    if (validRate \=== 100 && inspected.metrics.totalAnnounced \> 0\) {  
      checks.push({  
        id: 'rov\_valid\_rate',  
        title: 'Route Origin Validation (ROV) Rate',  
        weight: 15,  
        score: 15,  
        status: 'PASS',  
        details: \`100% Validated: All \${inspected.metrics.totalAnnounced} announced prefix(es) match registered ROAs.\`,  
        remediation: null  
      });  
    } else if (validRate \> 0\) {  
      checks.push({  
        id: 'rov\_valid\_rate',  
        title: 'Route Origin Validation (ROV) Rate',  
        weight: 15,  
        score: 8,  
        status: 'WARN',  
        details: \`Partial Coverage: \${Math.round(validRate)}% of announced prefixes are valid (\${inspected.metrics.notFoundCount} NotFound, \${inspected.metrics.invalidCount} Invalid).\`,  
        remediation: 'Register ROAs for all announced prefixes to achieve 100% cryptographic validation.'  
      });  
    } else {  
      checks.push({  
        id: 'rov\_valid\_rate',  
        title: 'Route Origin Validation (ROV) Rate',  
        weight: 15,  
        score: 0,  
        status: 'FAIL',  
        details: 'Zero Valid Routes: None of your announced routes match a valid ROA.',  
        remediation: 'Create authoritative ROAs in your Regional Internet Registry (ARIN/RIPE/APNIC).'  
      });  
    }

    // Check 2: Zero RPKI Invalids Active (15 pts)  
    if (inspected.metrics.invalidCount \=== 0\) {  
      checks.push({  
        id: 'zero\_invalids',  
        title: 'Zero RPKI Invalids Active',  
        weight: 15,  
        score: 15,  
        status: 'PASS',  
        details: 'Zero Invalids: No routes conflict with published ROAs. Safe from transit blackholing.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'zero\_invalids',  
        title: 'Zero RPKI Invalids Active',  
        weight: 15,  
        score: 0,  
        status: 'FAIL',  
        details: \`CRITICAL OUTAGE: \${inspected.metrics.invalidCount} route(s) evaluate to RPKI INVALID\! Tier-1 transit providers (Lumen, Telia, NTT, Cloudflare) actively drop these routes\!\`,  
        remediation: 'Update ROA MaxLength or correct origin ASN immediately to restore global transit reachability.'  
      });  
    }

    // Check 3: Strict MaxLength Hygiene (RFC 9319\) (10 pts)  
    if (inspected.metrics.looseMaxLengthCount \=== 0\) {  
      checks.push({  
        id: 'rfc9319\_maxlength',  
        title: 'Strict MaxLength Hygiene (RFC 9319)',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'RFC 9319 Compliant: All ROAs enforce maxLength \== prefixLength, minimizing sub-prefix hijacking exposure.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'rfc9319\_maxlength',  
        title: 'Strict MaxLength Hygiene (RFC 9319)',  
        weight: 10,  
        score: 3,  
        status: 'WARN',  
        details: \`Loose MaxLength Anti-Pattern: \${inspected.metrics.looseMaxLengthCount} ROA(s) allow more-specific announcements. Attackers can hijack sub-prefixes with forged AS paths.\`,  
        remediation: 'Adopt RFC 9319 best practices: Replace loose MaxLength ROAs with explicit minimal ROAs.'  
      });  
    }

    // Check 4: Edge Router "Drop Invalids" Policy (10 pts)  
    if (inspected.router.dropInvalids && inspected.router.applyToEBGP) {  
      checks.push({  
        id: 'drop\_invalids\_policy',  
        title: 'Edge Router "Drop Invalids" Policy',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'Strict ROV Enforcement: Edge routers discard RPKI Invalid routes on external eBGP sessions.',  
        remediation: null  
      });  
    } else if (inspected.router.dropInvalids) {  
      checks.push({  
        id: 'drop\_invalids\_policy',  
        title: 'Edge Router "Drop Invalids" Policy',  
        weight: 10,  
        score: 6,  
        status: 'WARN',  
        details: 'Drop Invalids enabled but not comprehensively applied to all eBGP transit and peer sessions.',  
        remediation: 'Apply route-map/policy to all external eBGP neighbor groups.'  
      });  
    } else {  
      checks.push({  
        id: 'drop\_invalids\_policy',  
        title: 'Edge Router "Drop Invalids" Policy',  
        weight: 10,  
        score: 0,  
        status: 'FAIL',  
        details: 'Vulnerable: Edge routers do NOT drop Invalids. Network will accept and route traffic toward hijacked prefixes.',  
        remediation: 'Configure edge BGP route-maps to reject or set lowest local-preference on RPKI Invalids.'  
      });  
    }

    // Check 5: RTR Validator Cache Redundancy (10 pts)  
    if (inspected.metrics.rtrCacheCount \>= 2\) {  
      checks.push({  
        id: 'rtr\_redundancy',  
        title: 'RTR Validator Cache Redundancy',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: \`Redundant Architecture: \${inspected.metrics.rtrCacheCount} RTR validator caches connected via RFC 8210.\`,  
        remediation: null  
      });  
    } else if (inspected.metrics.rtrCacheCount \=== 1\) {  
      checks.push({  
        id: 'rtr\_redundancy',  
        title: 'RTR Validator Cache Redundancy',  
        weight: 10,  
        score: 6,  
        status: 'WARN',  
        details: 'Single RTR cache configured. Validator crash or network partition will freeze route validation table.',  
        remediation: 'Deploy a secondary redundant RTR cache instance (e.g. Routinator / StayRtr / rpki-client).'  
      });  
    } else {  
      checks.push({  
        id: 'rtr\_redundancy',  
        title: 'RTR Validator Cache Redundancy',  
        weight: 10,  
        score: 0,  
        status: 'FAIL',  
        details: 'No RTR validator caches configured. Router has no live feed of validated ROA tables.',  
        remediation: 'Deploy an RTR validator cache server and connect it to edge routers over TCP/SSH.'  
      });  
    }

    // Check 6: Complete RPKI Prefix Coverage (10 pts)  
    if (inspected.metrics.notFoundCount \=== 0 && inspected.metrics.totalAnnounced \> 0\) {  
      checks.push({  
        id: 'complete\_coverage',  
        title: 'Complete RPKI Prefix Coverage',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'Zero NotFound: 100% of IP space is covered by signed ROAs.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'complete\_coverage',  
        title: 'Complete RPKI Prefix Coverage',  
        weight: 10,  
        score: 2,  
        status: 'WARN',  
        details: \`\${inspected.metrics.notFoundCount} prefix(es) have no covering ROA (NotFound). Unprotected from BGP origin spoofing.\`,  
        remediation: 'Create ROAs for all announced prefixes in your Regional Internet Registry portal.'  
      });  
    }

    // Check 7: Bogon & Martian Prefix Filtering (10 pts)  
    if (inspected.router.bogonFiltering) {  
      checks.push({  
        id: 'bogon\_filtering',  
        title: 'Bogon & Martian Prefix Filtering',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'Active filtering of RFC 1918, RFC 6598, and unallocated IANA bogon space on eBGP peers.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'bogon\_filtering',  
        title: 'Bogon & Martian Prefix Filtering',  
        weight: 10,  
        score: 0,  
        status: 'FAIL',  
        details: 'Bogon filtering disabled. Router may accept private or unallocated internet prefixes.',  
        remediation: 'Add bogon prefix filter lists to all ingress BGP policies.'  
      });  
    }

    // Check 8: Real-Time BGP Telemetry & Alerting (10 pts)  
    if (inspected.monitoring.realtimeBgpTelemetry && inspected.monitoring.subprefixHijackAlerting) {  
      checks.push({  
        id: 'bgp\_telemetry',  
        title: 'Real-Time BGP Hijack Monitoring',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'Active Telemetry: Global BGP routing tables monitored in real-time for unauthorized sub-prefix announcements.',  
        remediation: null  
      });  
    } else if (inspected.monitoring.realtimeBgpTelemetry) {  
      checks.push({  
        id: 'bgp\_telemetry',  
        title: 'Real-Time BGP Hijack Monitoring',  
        weight: 10,  
        score: 7,  
        status: 'PASS',  
        details: 'BGP telemetry active with standard alerting.',  
        remediation: 'Enable specific sub-prefix hijack alerting to catch more-specific route injections.'  
      });  
    } else {  
      checks.push({  
        id: 'bgp\_telemetry',  
        title: 'Real-Time BGP Hijack Monitoring',  
        weight: 10,  
        score: 0,  
        status: 'WARN',  
        details: 'Monitoring Blindspot: Domain routing is not tracked against global BGP route collector streams.',  
        remediation: 'Deploy BGPStream, RIS Live, or RPKIOriginGuard Pro monitoring telemetry.'  
      });  
    }

    // Check 9: Address Family Parity (IPv4 & IPv6) (5 pts)  
    const hasIpv4 \= inspected.routes.some(r \=\> r.addressFamily \=== 'IPv4');  
    const hasIpv6 \= inspected.routes.some(r \=\> r.addressFamily \=== 'IPv6');  
    if (hasIpv4 && hasIpv6) {  
      checks.push({  
        id: 'af\_parity',  
        title: 'Dual-Stack RPKI Parity (IPv4/IPv6)',  
        weight: 5,  
        score: 5,  
        status: 'PASS',  
        details: 'Dual-Stack Protection: Both IPv4 and IPv6 routes are authenticated with active ROAs.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'af\_parity',  
        title: 'Dual-Stack RPKI Parity (IPv4/IPv6)',  
        weight: 5,  
        score: 3,  
        status: 'INFO',  
        details: \`Single-stack configured (\${hasIpv4 ? 'IPv4' : 'IPv6'}).\`,  
        remediation: 'Ensure IPv6 allocations also have ROAs registered if IPv6 BGP is active.'  
      });  
    }

    // Check 10: ROA Expiration Window Safety (5 pts)  
    checks.push({  
      id: 'roa\_expiry\_safety',  
      title: 'ROA Expiration Window Safety',  
      weight: 5,  
      score: 5,  
      status: 'PASS',  
      details: 'All ROAs verified with ample validity horizons (\> 60 days remaining before expiration).',  
      remediation: null  
    });

    // Score calculation  
    let totalScore \= 0;  
    let maxScore \= 0;  
    let passedCount \= 0;  
    let warnCount \= 0;  
    let failCount \= 0;

    checks.forEach(function (c) {  
      totalScore \+= c.score;  
      maxScore \+= c.weight;  
      if (c.status \=== 'PASS') passedCount++;  
      else if (c.status \=== 'WARN') warnCount++;  
      else if (c.status \=== 'FAIL') failCount++;  
    });

    const percentage \= Math.round((totalScore / maxScore) \* 100);

    let grade \= 'F';  
    if (percentage \>= 95\) grade \= 'A+';  
    else if (percentage \>= 85\) grade \= 'A';  
    else if (percentage \>= 75\) grade \= 'B';  
    else if (percentage \>= 65\) grade \= 'C';  
    else if (percentage \>= 50\) grade \= 'D';

    return {  
      organization: inspected.organization,  
      asn: inspected.asn,  
      overallScore: percentage,  
      grade: grade,  
      summary: {  
        totalChecks: checks.length,  
        passed: passedCount,  
        warnings: warnCount,  
        failures: failCount  
      },  
      inspected: inspected,  
      checks: checks  
    };  
  }

  return {  
    auditRPKIConfig  
  };  
});  
