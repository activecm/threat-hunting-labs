# Threat Hunting

## Install:

* rita
  * setup internal subnets
* wireshark
* tshark
* zeek 3.0
  * or bro w/ alias for zeek and zeek-cut
* set up starting point
  * bookmark instructions in bookmark bar
  * set homepage to instructions
  * shortcut to instructions on desktop

Example:

```bash
#!/bin/bash

echo alias zeek=bro >> ~/.bashrc
echo alias zeek-cut=bro-cut >> ~/.bashrc

pushd "$(dirname "$BASH_SOURCE[0]")" > /dev/null

cp -R instructions Introduction.desktop ~/Desktop
cp pcaps/* ~/ &

firefox ~/Desktop/instructions/Introduction.html &

export DEBIAN_FRONTEND=noninteractive

sudo -E apt update
sudo -E apt install -y wireshark tshark

echo "n" | sudo installers/rita-install.sh

echo "Please setup InternalSubnets in /etc/rita/config.yaml"
echo "Please bookmark, and set the homepage"

```


## Future ideas

### Tools
* datamash
	* https://www.gnu.org/software/datamash/alternatives/
	* https://www.activecountermeasures.com/identifying-long-connections-with-bro-zeek/
* AI-Hunter
* Suricata
* SiLK
* Security Onion

### New labs
* Threat Intel
* Use other bro logs
* Lab on filtering
	- BPF filter - tcpdump, Zeek, Wireshark
	- Zeek - config file
	- RITA - config file
* Putting it all together. Develop a rating system. Bring all the labs together into a process.
	* Tips
		- start with most frequent (sort by largest # connections)
		- start with those that move the most data (sort by data size)
		- Reduce false positives with whitelists. Can the connection be explained? Is there a valid business need?
	- Start with persistent connections, major modifier
		- long connections
		- beacons
	- Analyze protocol & endpoint attributes, minor modifier
		- wrong protocol on port
		- blacklist/threat intel
		- large amount of dns traffic
		- invalid tls/ssl
		- uncommon user agent
		- uncommon ja3
		- bytes transferred
	- Trusted
		- whitelist, major
			- strongest is expected port, protocol, between two IPs
		- common server TLS, minor
		- common dns, alexa 1m, minor
	- Experimental modifiers
		- system purpose, should it be talking so much? e.g. HVAC, cameras, IoT
		- system visibility, do I have other visibility into this system? e.g. agent installs or anti-virus, or impossible due to embedded system
		- system criticality, how important is this system? e.g. database server, PCI
		- system exposure, e.g. employee email or web browsing


### Lab improvements
* Add references and further reading.
* Smaller pcap
	* either generate new or pare down current
* More pcaps
* Add callouts/warnings on long running activities (if we don't use a smaller pcap)
	* process pcap with Bro
	* load pcap in Wireshark
	* open Wireshark conversations window
* Add callouts when tools will spit out warnings/errors that can be ignored
* Pipe to `less -S` when screen overflows
* Add more guidepost statements like "The purpose of this exercise is to ..."
* Explain in the DNS lab how DNS malware is set up and why it works
* Make a rita section in the User Agent lab
* Come up with challenges that use the skills taught in the labs but which don't have walkthrouhgs
	* Could be used for CTF flags
	* Examples:
		- IP with longest connection
		- domain with most dns lookups, make it separate from the NS record so there's no direct traffic to it
		- least common user agent string
		- least common ja3 hash
		- IP with the highest score, given defined rating system

* Host the pcap and provide a download link
