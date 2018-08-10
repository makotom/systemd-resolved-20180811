# What happened?
This is a collection of logs for my attempt `dig @127.0.0.53 slave.www.nosec.makotom.net +dnssec` using systemd-resolved with `DNSSEC=yes` as a global option.
Although I expected `192.0.2.1` as a response, I actually got `SERVFAIL` as a response.

Note that I have the following zones and records:

## Zone `nosec.makotom.net`:
* `nosec.makotom.net` -> SOA `ns1.he.net`
* `www.nosec.makotom.net` -> CNAME `master.www.nosec.makotom.net`
* `master.www.nosec.makotom.net` -> NS `ns1.he.net` (+ other slave nameservers)
* `slave.www.nosec.makotom.net` -> CNAME `www.nosec.makotom.net`

## Zone `master.www.nosec.makotom.net`
* `master.www.nosec.makotom.net` -> SOA `ns1.he.net`
* `master.www.nosec.makotom.net` -> A `192.0.2.1`

From these logs, we can see that `master.www.nosec.makotom.net IN DS` is queried for validation of `www.nosec.makotom.net IN SOA`, which is obviously unneeded considering their hierarchy and which should not occur.
I also understand that 1) the resolver queries `www.nosec.makotom.net IN SOA` with `CD` flag asserted, 2) BIND (I guess, because I got the same result with my private BIND server) includes `master.www.nosec.makotom.net IN SOA` for some informative purposes (I guess), and that 3) the resolver gets confused as a result.
