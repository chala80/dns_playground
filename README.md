# BIG-IP DNS Traffic Lab

An interactive simulation of BIG-IP DNS (GTM) traffic, in a single HTML file with no dependencies beyond Google Fonts. Open `index.html` in a browser, or enable GitHub Pages for this repo.

## What it models

- **iQuery mesh:** gtmd on each of two BIG-IP DNS devices keeps a TLS connection (TCP 4353) to big3d on every BIG-IP, including itself. That makes 8 connections in total.
- **big3d:** keepalives, `bigip` monitor virtual server status from the LTMs, monitor probes, and path probes. Each result goes to every gtmd connected to that big3d.
- **Prober pools:** Round Robin vs Global Availability, member order and enable state, and prober preference and fallback per data center or per server.
- **Path metrics:** DNS_DOT probes to each LDNS for Round Trip Times load balancing.
- **DNS resolution:** client → LDNS → DNS listener. The answer depends on that DNS device's own view of member status.

## Scenarios

- Healthy mesh
- Prober pool failover
- WAN link cut
- iQuery certificate mismatch
- No prober available
- Round Trip Times load balancing

Click any device, iQuery line, data center or the WAN link to break it or reconfigure it. The console tabs mimic `/var/log/gtm`, `tmsh show gtm iquery`, `tmsh show gtm path` and `iqdump`.

## Caveats

- Time is compressed: 1 real second is 5 simulated seconds at 1×.
- Log lines and iQuery XML are modeled on real output but simplified. Some edge behavior is approximated. Verify against a real device.
