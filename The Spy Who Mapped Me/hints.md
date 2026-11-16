# Hints

## Rung 1
**Cheap nudge:** The blocklist checks *IP strings*, but the URL it receives
   first passes through the platform's standard URL parser. Compare the
   hostname text you can put in a URL with the exact string form the checker
   ends up seeing.

## Rung 2
**Narrower:** `isPrivateV6` special-cases addresses starting with `::ffff:`
   — the IPv4-mapped IPv6 range. It slices off the prefix and tries to parse
   the remainder as odtted-quad IPv4. What other textual forms can an
   IPv4-mapped address legally take after that prefix?

## Rung 3
**Narrow:** The WHATWG URL parser normalizes IPv4-mapped literals into
   *hex group* form: `::ffff:127.0.0.1` becomes `::ffff:7f00:1`. After slicing,
   `"7f00:1"` is not valid dotted-quad, so `net.isIPv4(...)` fails and the
   function falls through to `return false` — "not private".

## Rung 4
**Spicy:** In `src/security.ts`:
   ```ts
   if (lower.startsWith("::ffff:")) {
     const v4 = lower.slice(7);
     if (net.isIPv4(v4)) return isPrivateV4(v4);
   }
   return false;
   ```
   The mapped-address handling only understands one of the two legal encodings;
   the hex-group encoding produced by URL normalization is silently treated as
   public.

## Rung 5
**Giveaway:** Ask GateFetch to fetch `http://[::ffff:7f00:1]/` — that is
   loopback 127.0.0.1 in normalized hex form — or `http://[::ffff:a9fe:fe01]/`
   for 169.254.169.254 (cloud metadata). The host parses to a loopback/private
   socket at connect time, but `isPrivateV6("::ffff:7f00:1")` returns false, so
   the guard waves the request through to the server's own network.
