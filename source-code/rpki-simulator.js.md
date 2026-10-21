/\*\*  
 \* RPKIOriginGuard Pro \- BGP Hijack & Sub-Prefix Outage Simulator  
 \* Simulates:  
 \* 1\. Sub-Prefix BGP Hijack vs Strict RPKI Route Origin Validation (ROV)  
 \* 2\. Self-Inflicted MaxLength De-aggregation Blackhole Outage  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\[\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.RPKISimulator \= factory();  
  }  
})(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  function simulateBgpCrossHijack(params) {  
    const hasRoa \= Boolean(params.hasRoa);  
    const dropInvalids \= Boolean(params.dropInvalids);  
    const isProtected \= hasRoa && dropInvalids;

    const steps \= \[  
      {  
        phase: '1. Legitimate BGP Announcement',  
        description: 'Authorized AS64512 announces parent prefix 198.51.100.0/22 to upstream Tier-1 transit providers.',  
        trafficRoute: 'Global traffic routes normally to legitimate enterprise data center.',  
        isHijacked: false  
      },  
      {  
        phase: '2. Adversary Sub-Prefix Injection',  
        description: 'Malicious Attacker AS666 announces more-specific 198.51.100.0/24 prefix to global BGP table.',  
        trafficRoute: 'BGP Longest Prefix Match (LPM) rule triggers: Routers prefer /24 over /22.',  
        isHijacked: false  
      },  
      {  
        phase: '3. Route Origin Validation (ROV) Evaluation',  
        description: hasRoa  
          ? 'Transit routers check RPKI cache: ROA specifies MaxLength 22 & AS64512. Rogue route evaluates to RPKI INVALID.'  
          : 'Transit routers check RPKI cache: No ROA found (NotFound). Standard BGP acceptance applies.',  
        trafficRoute: hasRoa  
          ? (dropInvalids ? 'Tier-1 transits immediately DROP the invalid /24 announcement\!' : 'Routers accept Invalid route due to permissive policy\!')  
          : 'Routers accept /24 and propagate it globally\!',  
        isHijacked: \!isProtected  
      },  
      {  
        phase: '4. Global Routing & Traffic Impact',  
        description: isProtected  
          ? 'Zero Traffic Diverted: Strict RPKI ROV dropped rogue route. 100% of user traffic remains with legitimate origin.'  
          : 'CRITICAL SECURITY BREACH: 100% of traffic destined for 198.51.100.0/24 hijacked and intercepted by Attacker AS666\!',  
        trafficRoute: isProtected ? '100% Availability (Hijack Dropped)' : 'BGP Hijacked (Traffic Intercepted)',  
        isHijacked: \!isProtected  
      }  
    \];

    return {  
      scenario: 'Sub-Prefix BGP Hijack vs RPKI Route Origin Validation (ROV)',  
      hasRoa: hasRoa,  
      dropInvalids: dropInvalids,  
      isProtected: isProtected,  
      verdict: isProtected  
        ? 'Protected: Strict RPKI ROA \+ Drop-Invalid policy prevents sub-prefix BGP hijacking.'  
        : 'CRITICAL VULNERABILITY: Lack of ROA or Drop-Invalid enforcement allows attackers to hijack IP space via more-specific routes\!',  
      steps: steps  
    };  
  }

  function simulateMaxLengthBlackhole(params) {  
    const roaMatchesAnnouncement \= Boolean(params.roaMatchesAnnouncement);

    const steps \= \[  
      {  
        phase: '1. Registered ROA State',  
        description: 'RIR database holds registered ROA for 198.51.100.0/23 with MaxLength /23 for AS65500.',  
        networkState: 'Normal transit operation.'  
      },  
      {  
        phase: '2. Engineering Route De-aggregation',  
        description: roaMatchesAnnouncement  
          ? 'Network engineers update ROA in RIR portal to allow /24 before de-aggregating prefix.'  
          : 'Network engineers split /23 into two /24 announcements for traffic engineering, forgetting to update ROA.',  
        networkState: roaMatchesAnnouncement ? 'ROA synchronized.' : 'Desynchronized BGP announcement.'  
      },  
      {  
        phase: '3. Tier-1 Transit Ingestion',  
        description: roaMatchesAnnouncement  
          ? 'Transit providers validate announced /24 routes against updated ROA: State is VALID.'  
          : 'Transit providers validate announced /24 routes against old ROA: Prefix /24 exceeds MaxLength /23\! State is INVALID.',  
        networkState: roaMatchesAnnouncement ? 'Accepted by peers.' : 'Rejected as RPKI INVALID\!'  
      },  
      {  
        phase: '4. Service Availability Outcome',  
        description: roaMatchesAnnouncement  
          ? 'Smooth Deployment: De-aggregated sub-prefixes propagate globally with 0 packet loss.'  
          : 'SELF-INFLICTED BLACKOUT: Major Tier-1 transits discard the invalid announcements. 100% of incoming customer traffic dropped\!',  
        isOutage: \!roaMatchesAnnouncement  
      }  
    \];

    return {  
      scenario: 'Self-Inflicted MaxLength De-aggregation Blackhole Outage',  
      isOutage: \!roaMatchesAnnouncement,  
      verdict: roaMatchesAnnouncement  
        ? 'Resilient: Pre-commit ROA synchronization prevents MaxLength validation blackholes.'  
        : 'OUTAGE RISK: Announcing prefixes more specific than ROA MaxLength causes Tier-1 transit to blackhole your network\!',  
      steps: steps  
    };  
  }

  return {  
    simulateBgpCrossHijack,  
    simulateMaxLengthBlackhole  
  };  
});  
