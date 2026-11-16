# Solution

## Vulnerability

The challenge contains an SSRF protection bypass in [`src/security.ts`](./files/src/security.ts), in the IPv4-mapped IPv6 handling of `isPrivateV6()`. The guard recognizes mapped addresses when the IPv4 suffix is dotted-decimal, but Node.js's WHATWG URL parser normalizes an address such as `::ffff:127.0.0.1` to `::ffff:7f00:1`. The hex suffix is not recognized as IPv4, so the private-address check can incorrectly treat a loopback or other private IPv4 destination as safe.

An attacker able to supply a URL to the affected fetch or media-download tools could use this bypass to make requests to services reachable only on the host or private network.

**Affected versions:** `auth-fetch-mcp` <= 3.0.1  
**Patched version:** 3.0.2

## Fix

The fix reconstructs dotted-decimal IPv4 from the two hexadecimal groups in a normalized IPv4-mapped IPv6 address, then applies the existing private-IPv4 check. This retains the existing behavior for public mapped addresses while blocking mapped loopback and private addresses.

- [GitHub Security Advisory GHSA-pvrj-8cg3-j5f8](https://github.com/ymw0407/auth-fetch-mcp/security/advisories/GHSA-pvrj-8cg3-j5f8)
- [Fixed-code commit 177ec5f8ee9c2d5749035777e562f699971b0da9](https://github.com/ymw0407/auth-fetch-mcp/commit/177ec5f8ee9c2d5749035777e562f699971b0da9)
