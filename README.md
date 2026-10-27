# BIG-IP DNS Traffic Lab

An interactive simulation of GSLB on F5 BIG-IP DNS (GTM), in a single HTML file with no dependencies beyond Google Fonts. Open `index.html` in a browser, or enable GitHub Pages for this repo. The simulation starts paused: press **Start** to begin traffic.

## What it models

- **Two data centers:** each has one BIG-IP DNS device and one BIG-IP LTM.
- **Three wide IPs:** `www.example.com` (Round Robin), `api.example.com` (Global Availability) and `shop.example.com` (Round Trip Times). Each points to a GTM pool with one LTM virtual server in each data center.
- **bigip monitor:** sends no probe. The LTM's big3d reports the status of its own virtual servers to every gtmd over iQuery.
- **Custom monitors:** create http, https, tcp or gateway_icmp monitors with your own send and receive strings, then tick them on any pool. A prober big3d, picked by prober preference and fallback (server, data center or prober pool), probes the virtual server and reports the result to every gtmd. With several monitors on a pool, all must pass.
- **iQuery mesh:** gtmd on each of the two DNS devices keeps a TLS connection (TCP 4353) to big3d on every BIG-IP, itself included: 8 connections.
- **DNS resolution:** client → LDNS → DNS listener. The answer depends on that DNS device's own view of member status.

## Scenarios

- GSLB with the bigip monitor
- Add a custom HTTP monitor
- Application failure the bigip monitor misses (HTTP 503 behind a green LTM virtual server)
- Data center failover with Global Availability
- WAN link cut
- iQuery certificate mismatch
- Prober pool failover for a custom monitor
- Round Trip Times for shop.example.com

Click any device, virtual server, iQuery line, data center or the WAN link to break it or reconfigure it. The console tabs mimic `/var/log/gtm`, `tmsh show gtm iquery`, monitor instances, DNS answers, `tmsh show gtm path`, `bigip_gtm.conf` and `iqdump`.

## Caveats

- Time is compressed: 1 real second is 5 simulated seconds at 1×.
- Log lines, iQuery XML and config are modeled on real output but simplified. Some edge behavior is approximated. Verify against a real device.
