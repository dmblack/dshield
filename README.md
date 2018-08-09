# D-Shield Raspberry Pi Sensor

This project is set of scripts configures a Raspberry Pi as a D-Shield
Sensor.

The D-Shield sensor utilizes [Cowrie](https://github.com/micheloosterhof/cowrie || https://github.com/cowrie/cowrie),
[web.py](https://github.com/webpy/webpy), and iptables, to monitor and
record appropriate network activity.

At set intervals (twice a hour); logs are then parsed, feeding data to
ISC.SANS.EDU.

## Project Goals

* Ease of Configuration and Installation
* Minimal Complexity
* Use of a dedicated device

## Current Limitations

* No IPv6 Support (Currently Disabled)

## Getting Started

The following information will assist you in initializing your own
D-Shield Raspberry Pi Sensor. It is recommended you follow them, in
order.

### Forewarning

[liability here] D-Shield, it's community, and it's contributors; can
not take any responsibility for any undesired behavior or outcomes
otherwise resulting in use of in use of this tool and project.

By design; the purpose of this tool is to collect activity by exposing a
controlled endpoint to the internet.

It is, therefore, highly recommended that you;
* Read all installation steps before proceeding
* Understand the tool, including it's dependencies.

### Dependencies

* Appropriate [Network Infrastructure](Network-Infrastructure)
* [D-Shield Account and API Key](D-Shield-Account)
* Internet connectivity
* Internet quota of at least [~10MB per/day](Bandwdith)
* GIT
```
# sudo apt install -y git
```
* Raspberry Pi any model as [per](https://isc.sans.edu/forums/diary/Using+a+Raspberry+Pi+honeypot+to+contribute+data+to+DShieldISC/22680/)
* Raspian as [per](https://www.raspberrypi.org/downloads/raspbian/)
* SSH Enabled within Raspian as [per](https://www.raspberrypi.org/documentation/remote-access/ssh/)
* Timezone configured within Raspian
```
# sudo raspi-config
``` 
or
```
# sudo dpkg-reconfigure tzdata
```
* Updates installed for Raspian
```
# sudo apt update
# sudo apt -y upgrade
# sudo apt -y dist-upgrade
```

#### D-Shield Account

A D Shield account is required to use this service. You can create one
by [registering here](https://dshield.org/register.html). Soon after
registration, an API key is allocated.

You can confirm your [API Key Here](https://dshield.org/myaccount.html).

#### Bandwidth

Some users have reported from, and beyond, 10MB per day of traffic. Most
of this traffic was associated with SSH and Telnet serices. Your
experience will likely differ.

Below is a casual report of traffic, experienced by a user in Australia
over a 24 hour window.

(Gathered via pfSense)
```
IPv4 TCP 22 (SSH) NAT SSH Honeypot 	
2.16 MiB

IPv4 TCP 80 (HTTP) NAT HTTP Honeypot 	
3 KiB

IPv4 TCP 23 (Telnet) NAT Telnet Honeypot 	
5.65 MiB

IPv4 ANY:ANY (All) NAT (1:1) SANS Honeypot
.5 MiB
```

#### Network Infrastructure

To expose an asset to the internet, your Network Infrastucture should
support one of the following features;
* DMZ / Exposed Host
* NAT (1:1, etc)

[Optional Section] At very least; your Network Infrstructure should
support Port Forwarding, which will allow monitoring of core services (
HTTP/s, Telnet, and SSH).

### How It Works

This script will;
* Ensure most of the prerequisites above are met, and prompt if not.
* Disable IPv6
* Install further dependencies (Python, PIP, etc)
* Configure iptables to;
  * NAT illigimate connections for telnet (and telnet alt) to port 2221
  (Cowrie) (21,2121 ->  2221)
  * NAT illigimate connections for ssh to port 2222 (Cowrie) (22 ->
  2222)
  * NAT legitimate ssh connections to port 22 from 12222 
  (12222 -> 22)
  * NAT illigimate connections for http/s to a web.py service
  (80,5555,7547,8080,9000 -> 8000)
* Configure RSyslog as necessary
* Configure SSH to listen on port 12222
* Configure CRON for 2xper hour reports to dshield
* Install, and configure, Cowrie
* Install, and configure, Web.py
* Install and configure postfix (not sure why?)
* Change your MOTD to reflect it's now a D-Shield SEnsor
* Reboot

### Installation

Before installation, it's important you [change the default username and
or password](https://www.raspberrypi.org/documentation/linux/usage/users.md)
for raspian. Whilst no services are intentionally exposed,
it is, at very least; best practice.

#### Software Installation
```
git clone https://github.com/DShield-ISC/dshield.git
```
- run the installation script
```
cd dshield/bin
sudo ./install.sh
```
- if curious watch the debug log file in parallel to the installation:
connect with an additional ssh session to the system and run (name of
the log file will be printed out by the installation script):
```
sudo tail -f LOGFILE
```
- answer the questions of the installation routine
- if everything goes fine and the script finishes OK: reboot the device 
```
sudo reboot
```

#### Network Configuration / Physical Placement

This dshield sensor and honeypot is meant to only analyze Internet
related traffic, i.e. traffic which is issued from public IP addresses:
- this is due to how the dshield project works (collection of
information about the current state of the Internet)
- only in this way information which is interesting for the Internet
security community can be gathered
- only in this way it can be ensured that no internal, non-public
information is leaked from your Pi to Dshield

So you must place the Pi on a network where it can be exposed to the
Internet (and won't be connected to from the inner networks, except for
administrative tasks). For a maximum sensor benefit it is desirable that
the Pi is exposed to the whole traffic the Internet routes to a public
IP (and not only selected ports).

For SoHo users there is normally an option in the DSL or cable router to
direct all traffic from the public IP the router is using (i.e. has been
assigned by the ISP) to an internal IP. This has to be the Pi. This
feature is named e.g. "exposed host", "DMZ" (here you may have to enable
further configuration to ensure ___all___ traffic is being routed to the
Pi's internal IP address and not only e.g. port 80).

For enterprises a protected DMZ would be a suitable place (protected: if
the sensor / honeypot is hacked this incident is contained and doesn't
affect other hosts in the DMZ). Please be aware that - if using static
IPs - you're exposing attacks / scans to your IP to the dhshield project
and the community which can be tracked via whois to your company.

To test your set up you may use a public port scanner and point it to
the router's public IP (which is then internally forwarded to the Pi).
This port scan should be directly visible in `/var/log/dshield.log` and
later in your online report accessible via your dshield account. Use
only for quick and limited testing purposes, please, so that dhshield
data isn't falsified.

- expose the Pi to inbound traffic. For example, in many firewalls and
home routers you will be able to configure it as a "DMZ Hosts", "exposed
devices", ... see [hints below](#how-to-place-the-dshield-sensor--honeypot)
for - well - hints ...

### Usage

After installation; the D-Shield Sensor is mostly self sufficient. You
should not need to login (via port SSH: 12222) for anything other than
general maintenance (System Updates, if not enabled).

Within the hour, you should start seeing data fed via your [D-Shield
Reports](https://dshield.org/myreports.html).

## Deployment

Add additional notes about how to deploy this on a live system

## Built With

* [Cowrie](https://github.com/micheloosterhof/cowrie) - The ssh/telnet
Honeypot
* [Web.py](https://github.com/webpy/webpy) - The HTTP/s/alt Honeypot

## Contributing

Please read [CONTRIBUTING.md](https://github.com/DShield-ISC/dshield/blob/master/CONTRIBUTING.md)
for details on our code of conduct, and contributing.

## Authors

See the list of [contributors](https://github.com/DShield-ISC/dshield/graphs/contributors)
who participated in this project.

## License

This project is licensed under the GPL V2 License - see the [LICENSE](LICENSE)
file for details
