# Privacy

This software will turn your Raspberry Pi into a DShield sensor. The
software will transmit various logs to DShield.org / isc.sans.edu. You
may review the data you submitted by logging into these websites. See
the screen shots in the "docs" folder to get an idea what this looks
like.

Currently, the following data is transmitted:

Firewall Logs (Via IPTables)
The content of firewall logs. The logs can be reviewed in
/var/log/dshield. You may adjust your iptables rules to reduce the
amount of data logged. By default, your "admin network" is exempt from
logging. If your Pi is behind a NAT firewall, then only the NAT'ed IP
will be reported.

SSH/Telnet Logs (Via Cowrie)
Connections intercepted by the cowrie SSH honeypot, in particular source
IP address, username and password. This is also performed for Telnet
connections.

Web Request Logs (Via web.py)
Connections to the web.py service are also, logged.

All data submitted will be shared as part of the DShield website.

Please note that we are in our beta test phase. Data submitted may
change at any time and without notice. We try hard to consider the
privacy of our submitters, but bugs happen in particular during the beta
testing phase
