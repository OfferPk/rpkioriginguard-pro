/\*\*  
 \* RPKIOriginGuard Pro \- Core Engine  
 \* RFC 6480, RFC 6811 (ROV), RFC 8210 (RTR), and RFC 9319 (Loose MaxLength) Analyzer  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\[\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.RPKIEngine \= factory();  
  }  
})(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  /\*\*  
   \* Helper to parse CIDR prefix into IP and prefix length  
   \*/  
  function parseCidr(cidr) {  
    if (\!cidr || typeof cidr \!== 'string') return null;  
    const parts \= cidr.trim().split('/');  
    if (parts.length \!== 2\) return null;  
    const ip \= parts\[0\];  
    const len \= parseInt(parts\[1\], 10);  
    if (isNaN(len)) return null;  
    const isIpv6 \= ip.includes(':');  
    return { cidr, ip, length: len, isIpv6 };  
  }

  /\*\*  
   \* Simplified prefix containment check  
   \* Checks if candidate route prefix is covered by roa prefix  
   \*/  
  function isPrefixCovered(routePrefixStr, roaPrefixStr) {  
    const route \= parseCidr(routePrefixStr);  
    const roa \= parseCidr(roaPrefixStr);  
    if (\!route || \!roa) return false;  
    if (route.isIpv6 \!== roa.isIpv6) return false;

    // Route must be equal or more specific than ROA prefix  
    if (route.length \< roa.length) return false;

    if (\!route.isIpv6) {  
      // IPv4 octet containment  
      const routeOctets \= route.ip.split('.').map(Number);  
      const roaOctets \= roa.ip.split('.').map(Number);  
        
      const routeInt \= (routeOctets\[0\] \<\< 24\) | (routeOctets\[1\] \<\< 16\) | (routeOctets\[2\] \<\< 8\) | routeOctets\[3\];  
      const roaInt \= (roaOctets\[0\] \<\< 24\) | (roaOctets\[1\] \<\< 16\) | (roaOctets\[2\] \<\< 8\) | roaOctets\[3\];

      const mask \= (\~0 \<\< (32 \- roa.length)) \>\>\> 0;  
      return (routeInt & mask) \=== (roaInt & mask);  
    } else {  
      // IPv6 basic prefix matching check  
      const routeClean \= route.ip.split('::')\[0\];  
      const roaClean \= roa.ip.split('::')\[0\];  
      return routeClean.startsWith(roaClean) || route.ip.startsWith(roa.ip);  
    }  
  }

  /\*\*  
   \* Perform RFC 6811 Route Origin Validation (ROV)  
   \* @param {Object} route { prefix: "198.51.100.0/24", originAsn: 64512 }  
   \* @param {Array} roas Array of registered ROAs \[{ prefix, maxLength, asn }\]  
   \*/  
  function validateRouteOrigin(route, roas) {  
    const routeParsed \= parseCidr(route.prefix);  
    if (\!routeParsed) {  
      return { status: 'Invalid', reason: 'Malformed Route Prefix' };  
    }

    const coveringRoas \= (roas || \[\]).filter(function (roa) {  
      return isPrefixCovered(route.prefix, roa.prefix);  
    });

    if (coveringRoas.length \=== 0\) {  
      return {  
        status: 'NotFound',  
        reason: 'No covering ROA found. Route is accepted by default but vulnerable to BGP hijacking.',  
        coveringRoas: \[\]  
      };  
    }

    // Check if any covering ROA produces 'Valid'  
    for (let i \= 0; i \< coveringRoas.length; i++) {  
      const roa \= coveringRoas\[i\];  
      const asnMatches \= Number(route.originAsn) \=== Number(roa.asn);  
      const lengthValid \= routeParsed.length \<= roa.maxLength;

      if (asnMatches && lengthValid) {  
        return {  
          status: 'Valid',  
          reason: \`Matched ROA \${roa.prefix} (AS\${roa.asn}, MaxLength \${roa.maxLength}).\`,  
          matchedRoa: roa,  
          coveringRoas: coveringRoas  
        };  
      }  
    }

    // If covering ROAs exist but none matched both criteria, it is 'Invalid'  
    const lenExceeded \= coveringRoas.some(r \=\> routeParsed.length \> r.maxLength);  
    const asnMismatch \= coveringRoas.every(r \=\> Number(route.originAsn) \!== Number(r.asn));

    let invalidReason \= 'Route conflicts with covering ROA(s).';  
    if (lenExceeded && asnMismatch) {  
      invalidReason \= 'Invalid: Both Origin ASN mismatch AND prefix length exceeds MaxLength\!';  
    } else if (lenExceeded) {  
      invalidReason \= \`Invalid: Prefix length /\${routeParsed.length} exceeds ROA MaxLength /\${coveringRoas\[0\].maxLength}\! (Causes Tier-1 transit drop)\`;  
    } else if (asnMismatch) {  
      invalidReason \= \`Invalid: Origin AS\${route.originAsn} does not match authorized ROA AS\${coveringRoas\[0\].asn}\! (BGP Hijack suspected)\`;  
    }

    return {  
      status: 'Invalid',  
      reason: invalidReason,  
      coveringRoas: coveringRoas  
    };  
  }

  /\*\*  
   \* Complete inspection of enterprise BGP & RPKI state  
   \*/  
  function inspectEnterpriseRPKI(config) {  
    const org \= config.organization || 'Unknown Org';  
    const asn \= config.asn || 0;  
    const announcedPrefixes \= Array.isArray(config.announcedPrefixes) ? config.announcedPrefixes : \[\];  
    const roas \= Array.isArray(config.roas) ? config.roas : \[\];  
    const routerConfig \= config.routerConfig || {};  
    const monitoring \= config.monitoring || {};

    // 1\. Audit each announced prefix  
    let validCount \= 0;  
    let invalidCount \= 0;  
    let notFoundCount \= 0;

    const auditedRoutes \= announcedPrefixes.map(function (route) {  
      const rov \= validateRouteOrigin(route, roas);  
      if (rov.status \=== 'Valid') validCount++;  
      else if (rov.status \=== 'Invalid') invalidCount++;  
      else if (rov.status \=== 'NotFound') notFoundCount++;

      return {  
        prefix: route.prefix,  
        originAsn: route.originAsn,  
        addressFamily: route.addressFamily || 'IPv4',  
        rovStatus: rov.status,  
        reason: rov.reason,  
        matchedRoa: rov.matchedRoa || null  
      };  
    });

    // 2\. Audit RFC 9319 Loose MaxLength Anti-Patterns  
    let looseMaxLengthCount \= 0;  
    const auditedRoas \= roas.map(function (roa) {  
      const parsed \= parseCidr(roa.prefix);  
      const isLoose \= parsed && roa.maxLength \> parsed.length;  
      if (isLoose) looseMaxLengthCount++;

      return {  
        prefix: roa.prefix,  
        maxLength: roa.maxLength,  
        asn: roa.asn,  
        isLooseMaxLength: isLoose,  
        looseDifference: isLoose ? roa.maxLength \- parsed.length : 0  
      };  
    });

    // 3\. Router & RTR Validator Evaluation  
    const rtrValidators \= Array.isArray(routerConfig.rtrValidators) ? routerConfig.rtrValidators : \[\];  
    const dropInvalids \= Boolean(routerConfig.dropInvalids);  
    const bogonFiltering \= Boolean(routerConfig.bogonFiltering);  
    const applyToEBGP \= Boolean(routerConfig.applyToEBGP);

    return {  
      organization: org,  
      asn: asn,  
      rir: config.rir || 'ARIN',  
      routes: auditedRoutes,  
      roas: auditedRoas,  
      metrics: {  
        totalAnnounced: announcedPrefixes.length,  
        validCount: validCount,  
        invalidCount: invalidCount,  
        notFoundCount: notFoundCount,  
        looseMaxLengthCount: looseMaxLengthCount,  
        rtrCacheCount: rtrValidators.length  
      },  
      router: {  
        platform: routerConfig.platform || 'BIRD2',  
        dropInvalids: dropInvalids,  
        bogonFiltering: bogonFiltering,  
        applyToEBGP: applyToEBGP,  
        rtrValidators: rtrValidators  
      },  
      monitoring: {  
        realtimeBgpTelemetry: Boolean(monitoring.realtimeBgpTelemetry),  
        subprefixHijackAlerting: Boolean(monitoring.subprefixHijackAlerting),  
        roaExpiryAlerting: Boolean(monitoring.roaExpiryAlerting)  
      }  
    };  
  }

  return {  
    parseCidr,  
    isPrefixCovered,  
    validateRouteOrigin,  
    inspectEnterpriseRPKI  
  };  
});  
