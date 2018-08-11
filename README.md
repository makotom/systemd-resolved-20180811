# What happened?
This is a collection of logs for my attempt `dig @127.0.0.53 slave.www.nosec.makotom.net +dnssec` using systemd-resolved (in systemd 239) with `DNSSEC=yes` as a global option.
Although I expect `192.0.2.2` as a response, I actually get `SERVFAIL` as a response.

Note that I have the following zones and records _with DNSSEC disabled_:

## Zone `nosec.makotom.net`:
* `nosec.makotom.net` -> SOA `ns1.he.net`
* `www.nosec.makotom.net` -> CNAME `master.www.nosec.makotom.net`
* `master.www.nosec.makotom.net` -> NS `ns1.he.net` (+ other slave name servers)
* `slave.www.nosec.makotom.net` -> NS `ns1.he.net` (+ other slave name servers)

## Zone `master.www.nosec.makotom.net`
* `master.www.nosec.makotom.net` -> SOA `ns1.he.net`
* `master.www.nosec.makotom.net` -> A `192.0.2.1`

## Zone `slave.www.nosec.makotom.net`
* `slave.www.nosec.makotom.net` -> SOA `ns1.he.net`
* `slave.www.nosec.makotom.net` -> A `192.0.2.2`

From these logs, we can see that validation of `master.www.nosec.makotom.net` is required for validation of `www.nosec.makotom.net IN SOA`. This is obviously redundant and broken, because `master.www.nosec.makotom.net` is apparently a child (i.e. a sub-zone or a simple record name) of the zone for `www.nosec.makotom.net`, and it should not affect processes for its parents - i.e. the validation of `www.nosec.makotom.net IN SOA`.

I also understand that 1) the recursive name server adds `master.www.nosec.makotom.net IN SOA` to a response for `www.nosec.makotom.net IN SOA` (which happens with my private BIND recursive server as well), and that 2) the resolver seems to get confused as a result. I also confirmed that this does not happen with systemd 232, but 239.

# Impact of this issue
Due to this issue, you may fail to open per-region AWS control panel (`*.console.aws.amazon.com`), because `console.aws.amazon.com IN SOA` fails for the same logic.
