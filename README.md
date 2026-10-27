# BIG-IP DNS Traffic Lab

An interactive simulation of GSLB on F5 BIG-IP DNS (GTM), in a single HTML file with no dependencies beyond Google Fonts. Open `index.html` in a browser, or enable GitHub Pages for this repo. The lab always loads paused: press **Start** to begin traffic.

## What it models

- **BIND and delegation:** `ns1.chala.info` (BIND) is authoritative for `chala.info`. Each app name is a CNAME into a delegated subdomain (`app1.chala.info → app1.gtm.chala.info`), and `gtm.chala.info` is delegated with NS records to the two BIG-IP DNS listeners.
- **Split DNS:** office clients use an internal resolver (10.0.0.53) and internet clients use an ISP resolver (198.51.100.53).
  - BIND views: resolvers in `10.0.0.0/8` get the `internal` view and everyone else the `external` view. Both CNAME the apps into `gtm.chala.info`, but delegate it to the listeners' internal (10.x) or public NATed (203.0.113.x) addresses.
  - BIG-IP DNS topology: each wide IP has an `_int` and an `_ext` pool and uses Topology. An LDNS in region `internal_nets` gets internal answers; any other LDNS gets public ones.
  - External virtual server objects carry a translation address, so gtmd answers the public address while big3d monitors the private one.
- **Resolver behavior:** the LDNS follows the CNAME and caches each record for its TTL: BIND's CNAME and NS for 3600 s, gtmd's A record for the pool TTL of 30 s.
- **Two data centers:** each has one BIG-IP DNS device and one BIG-IP LTM.
- **Wide IPs:**
  - `app1.gtm.chala.info` uses Round Robin.
  - `app2.gtm.chala.info` uses Global Availability.
  - `app3.gtm.chala.info` uses Round Trip Times.

  Each points to a GTM pool with one LTM virtual server per data center.
- **Generic hosts:** `app4.gtm.chala.info` (Global Availability) points to `gen-nyc-01` and `gen-sea-01`, servers of type Generic Host. They run no big3d, so they have no iQuery connection and can't use the bigip monitor. Each is monitored by a custom monitor run through its data center's prober pool (`pp_east`, `pp_west`).
- **bigip monitor:** sends no probe. The LTM's big3d reports the status of its own virtual servers to every gtmd over iQuery.
- **Custom monitors:** create http, https, tcp or gateway_icmp monitors with your own send and receive strings, then tick them on any pool. A prober big3d probes the member and reports the result to every gtmd. It's picked by prober preference and fallback: server, data center or prober pool. With several monitors on a pool, all must pass.
- **iQuery mesh:** gtmd on each of the two DNS devices keeps a TLS connection (TCP 4353) to big3d on every BIG-IP, itself included: 8 connections.

## Scenarios

- DNS lookup walkthrough, internal and then external, with a hint at each hop (default)
- GSLB with the bigip monitor
- Add a custom HTTP monitor
- Application failure the bigip monitor misses (HTTP 503 behind a green LTM virtual server)
- Data center failover with Global Availability
- Generic hosts monitored through prober pools
- WAN link cut
- iQuery certificate mismatch
- Round Trip Times for app3.chala.info

Click any device, virtual server, generic host, BIND, an iQuery line, a data center or the WAN link to inspect or change it. From a client's panel you can walk through a lookup of any app name with hints. The console tabs mimic `/var/log/gtm`, DNS answers (with the hops each lookup took), monitor instances, `tmsh show gtm iquery`, `tmsh show gtm path`, `bigip_gtm.conf`, BIND `named.conf` views and zone files, and `iqdump`.

## Caveats

- In a real config the internal and external virtual server objects are monitored separately. The lab shares one status per real server to keep the diagram readable.
- 1× is real time with BIG-IP default timers: iQuery heartbeat 10 s, bigip monitor 30 s / 90 s, custom monitors 30 s / 120 s, path TTL 2400 s, pool TTL 30 s. Use 5× to 20× to skip ahead. Packets in flight are slowed down so they are visible.
- The LDNS is assumed to already know where `chala.info` lives, so root and `.info` lookups are skipped.
- Log lines, iQuery XML and config are modeled on real output but simplified. Some edge behavior is approximated. Verify against a real device.
