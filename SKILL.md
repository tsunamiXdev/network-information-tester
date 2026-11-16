---
name: network-information-tester
description: Test a browser's public IP, WebRTC address exposure, and DNS resolvers, then save a compact network snapshot. Also inspect public DNS records when a domain is supplied. Use for network identity and VPN/proxy exposure checks, not speed tests or security scanning.
---

# Network Information Tester

Run three checks in the same browser/profile and produce a short, timestamped snapshot. No installation or account is required; use available browser automation and, for optional domain records, a local DNS lookup tool.

## Establish the test context

- Use the browser/profile the user identifies; otherwise use the active user browser and record its identity. If multiple plausible targets are present, ask which one while preparing the report.
- Record the time with timezone, browser/profile, execution location (user device or remote environment), and VPN/proxy expectation if known. Never infer that a VPN is enabled from IP ownership alone.
- A web-search fetch, cloud browser, or shell request can have different routing from the user's browser. Never present that environment's IP or DNS as the user's. If the requested browser cannot be controlled, mark browser checks unavailable and report any independently supported results with their separate context.
- Keep network settings and extensions unchanged. These diagnostics contact external test services and may use STUN and generated DNS queries. Do not grant camera, microphone or precise-location permission; it is unnecessary for this check.

## Run the three checks

### 1. Public IP

Open [BrowserLeaks IP](https://browserleaks.com/ip) in the target browser. Record the displayed public IPv4 and IPv6, ISP/organization, ASN and approximate country/region when available. Record an absent IPv6 result as “not observed,” not proof that IPv6 is disabled. IP geolocation is an estimate, not the device's physical location.

### 2. WebRTC

Open [BrowserLeaks WebRTC](https://browserleaks.com/webrtc) in that same browser. Wait for the test to settle; record WebRTC support, local/private addresses or mDNS names, public addresses and the displayed test outcome. Distinguish disabled/unsupported, completed with no exposed address, and failed/incomplete tests.

Compare the public WebRTC addresses with the HTTP IPv4/IPv6 from check 1. A different public address merits investigation; it does not by itself prove a VPN leak. A private address or mDNS name alone is not a public-IP leak. If the HTTP baseline on this page differs from check 1, note possible route rotation and repeat the IP check once before comparing. Do not toggle WebRTC to obtain a different result.

### 3. DNS

Open [BrowserLeaks DNS](https://browserleaks.com/dns) in the same browser. Start the test if necessary and wait for completion. Record the observed resolver IPs, provider/organization and approximate locations, deduplicating repeated resolvers. Resolver locations can differ from the exit-IP location because of forwarding, anycast, or encrypted DNS; do not label that difference alone a leak. Compare against the user's expected VPN/proxy/DNS provider only when that expectation is established. Without an expected route, report “exposure observed; leak status unverified.”

If the user supplies a domain or specifically requests DNS records, also query that domain's A, AAAA, CNAME, MX, NS, TXT and SOA records using `dig` or an equivalent available tool. A URL's hostname is the lookup target; omit its path, query and credentials. With `dig`, run each type separately, for example:

```sh
dig +time=3 +tries=1 example.com A
```

Keep the response status, TTL, answer and resolver used; distinguish NXDOMAIN, NOERROR with no answer, SERVFAIL and timeout. Use bounded lookups, not zone transfers or subdomain enumeration. Label this as a domain-record lookup from the shell's resolver, separate from the browser DNS exposure test. If only a record lookup is requested and the domain is missing, ask for the domain; continue the other checks. If a DNS tool is absent, report that limitation without installing software.

## Save the snapshot

Use the requested output destination, otherwise `tmp/network-information-tester/<YYYYMMDD-HHMMSS-timezone>-snapshot.md` relative to the working project. Use a unique filename, protect the saved file with owner-only permissions where supported, and avoid overwriting earlier snapshots. Keep full IP addresses and private network details in the local snapshot; omit them from shared work logs unless requested. No credentials or account identifiers belong in this skill or its outputs.

Use this compact report structure, replacing each field with observations or an explicit unavailable/not-tested reason:

```text
Network snapshot — <timestamp and timezone>
Context: <browser/profile, user device or remote, expected routing or unknown>
IP: <IPv4; IPv6; ISP/ASN; approximate region; source and observation time>
WebRTC: <support; local/mDNS and public addresses; comparison; source and time>
DNS: <completion; resolvers/providers/regions; expectation comparison; source and time>
Domain records: <domain, resolver, status/TTL/answers by type, or not requested>
Assessment: <observations, concerns, and remaining uncertainty>
Limitations: <blocked, unavailable, or incomplete checks>
```

Allow about 60 seconds per browser test; if it remains incomplete, retry once, then record the failure and continue. A missing result is never a pass. Report “no additional public address observed in this test” when supported; never certify anonymity, complete leak protection or security from this snapshot.

Return the snapshot link and a brief assessment. If comparing runs, keep their browser, routing and timestamps explicit. This is a point-in-time observation; do not change network configuration or launch recurring monitoring unless separately requested.
